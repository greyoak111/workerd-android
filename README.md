# workerd-android

在安卓手机上本地运行 [workerd](https://github.com/cloudflare/workerd)（Cloudflare Workers 运行时，`wrangler dev` 靠它），供 [cccodroid](https://github.com/greyoak111/cccodroid) 的 Cloudflare 组件使用。

## 为什么要单独编译

Cloudflare 官方只发 Linux、macOS、Windows 版本。linux-arm64 版本可以借 glibc 加载器在安卓上启动，但它内置的内存分配器 tcmalloc 假定有 48 位虚拟地址，而安卓内核一般只给用户程序 39 位，运行一会儿就会崩。

workerd 本身就有关掉 tcmalloc 的编译开关（原本给 ASan 构建用）。这个仓库只做一件事：用官方 `release_linux` 配置，加上

```
--@workerd//src/workerd/server:use_tcmalloc=False
```

在 GitHub Actions 的 arm64 机器上编出 linux-arm64 版，改用 glibc 自带的 malloc。源码不做任何改动。

## 用法

在 Actions 里手动运行 `build`，填入要编的 workerd 版本标签（要和 wrangler 自带的 workerd 版本一致）。产物发布在同名 Release：

| 文件 | 是什么 |
|---|---|
| `workerd` | 编出来的原样二进制（解释器是 `/lib/ld-linux-aarch64.so.1`，在安卓上要显式经 glibc 加载器启动） |
| `workerd.gz` | 同上，压缩版 |
| `workerd-cc.codroid` | 同一份二进制，只用 patchelf 把解释器改成 cccodroid 的 glibc 加载器（`/data/data/cc.codroid/files/usr/glibc/lib/ld-linux-aarch64.so.1`），给 cccodroid 组件用 |
| `SHA256SUMS` | 以上三个的 sha256 |

早于构建流程加上 cc.codroid 版的 Release，可以手动运行 `cc.codroid 构件`（`cc-codroid.yml`）补上。

## 许可

workerd 以 Apache-2.0 许可发布，版权归 Cloudflare 所有；本仓库只包含构建脚本。
