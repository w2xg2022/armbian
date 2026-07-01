# w2xg2022 / armbian

> 🔧 个人云编译仓库 ｜ 姊妹仓：[Armbian（本仓）](https://github.com/w2xg2022/armbian) · [FnOS / FnNAS](https://github.com/w2xg2022/fnnas) · 内核源码 [armbian-kernel](https://github.com/w2xg2022/armbian-kernel)

本仓库 fork 自 [ophub/amlogic-s9xxx-armbian](https://github.com/ophub/amlogic-s9xxx-armbian)，用于**为新的电视盒子 / 开发板适配并云编译 Armbian 固件**。

适配成果（设备树 dts、驱动、机型定义 model_database）也会通过 Pull Request **回馈共享到上游 ophub**，让更多人受益。

- 📦 固件下载：[Releases](https://github.com/w2xg2022/armbian/releases)
- 🐧 内核源码：[w2xg2022/armbian-kernel](https://github.com/w2xg2022/armbian-kernel)（当前 6.18.y）
- 🗄️ FnOS 固件：[w2xg2022/fnnas](https://github.com/w2xg2022/fnnas)
- 默认账号 `root` ／ 默认密码 `1234`

## 已适配型号

<table>
<thead>
<tr>
<th nowrap>品牌</th><th nowrap>型号</th><th nowrap>芯片</th><th nowrap>架构</th><th nowrap>RAM+ROM</th><th nowrap>机型代号</th><th>固件</th>
</tr>
</thead>
<tbody>
<tr>
<td nowrap>浪潮</td><td nowrap>MD1000</td><td nowrap>RK3566</td><td nowrap>arm64</td><td nowrap>2+32</td><td nowrap><code>md1000</code></td><td><a href="https://github.com/w2xg2022/armbian/releases">Releases</a>（trixie）</td>
</tr>
</tbody>
</table>

> 注：本仓库只服务 64 位（arm64）设备。32 位（armhf）芯片（如 RK3288 / RK3128）不走本仓库。

## 如何获取固件

### 方法一：直接下载（普通用户推荐）

到 [Releases](https://github.com/w2xg2022/armbian/releases) 下载文件名含**对应机型代号**（见上表「机型代号」列）的 `.img.gz`，解压后用 balenaEtcher / RKDevTool 烧录到 TF 卡或 eMMC 即可。

### 方法二：云编译（GitHub Actions）

> ⚠️ 此方式面向**维护者 / 想自行编译的人**。GitHub 不允许在他人仓库触发 Actions，所以你需要先把本仓库 **Fork 到自己的账号**，在**你自己的 Fork** 里运行（只想要现成固件，请用方法一）。

在自己 Fork 的 **Actions** 页面手动触发，或用 `gh` 命令行（把 `<你的账号>` 换成你的 GitHub 账号）：

```bash
# ① 先编译内核（编译前会自动 sync 上游最新版，确保内核不落后）
gh workflow run compile-kernel.yml --repo <你的账号>/armbian

# ② 内核好了之后，再打包固件
gh workflow run build-armbian-arm64-server-image.yml --repo <你的账号>/armbian
```

**两步需要手动依次触发**（当前未开启自动接力，也未开启每月定时任务；待适配型号增多、仓库稳定后再考虑恢复自动化）：

1. **Compile the kernel** —— 编译前先把 [armbian-kernel](https://github.com/w2xg2022/armbian-kernel) 同步到 ophub/linux-6.18.y 最新版，再编译 6.18.y 内核，发布到本仓 `kernel_stable`。
2. **Build Armbian arm64 server image** —— 打包固件（trixie 用户空间在打包当下从官方源现装，天生最新）并上传到 Releases。

要打包**哪个机型**，触发 `build-armbian-arm64-server-image.yml` 时用 `armbian_board` 参数指定（见上表「机型代号」列），默认值为 `md1000`。

### 方法三：git 到本地编译（在 Ubuntu 22.04 / 24.04，x86 或 arm64）

```bash
git clone https://github.com/w2xg2022/armbian.git
cd armbian

# 安装编译依赖
sudo apt-get update
sudo apt-get install -y $(curl -fsSL https://ophub.org/ubuntu2404-build-armbian-depends)

# ① 编译内核（来源 = 我们自己的内核仓）
sudo ./recompile -k 6.18.y -r w2xg2022/armbian-kernel@main

# ② 打包固件，<机型代号> 见上表（需先准备 Armbian rootfs 基础镜像，详见上游文档）
sudo ./rebuild -b <机型代号> -k 6.18.y -r w2xg2022/armbian   # 例：-b md1000
```

> 本地编译的完整参数、写入 eMMC（`armbian-install`）、内核更新（`armbian-update`）等用法，参见上游文档：[ophub/amlogic-s9xxx-armbian](https://github.com/ophub/amlogic-s9xxx-armbian)。

## 如何适配一块新板子

1. 在 [armbian-kernel](https://github.com/w2xg2022/armbian-kernel) 的 `main` 加入该板设备树 `.dts`（必要时加驱动），编出 `.dtb`；
2. 在本仓 `build-armbian/armbian-files/common-files/etc/model_database.conf` 增加一行机型定义（`FDTFILE` 指向上一步的 dtb、`BUILD=yes`）；
3. 把机型代号加入 `MONTHLY_BOARDS` 变量；
4. 适配成功后，向 ophub 提交 Pull Request 共享成果。

---

本仓库基于 [ophub/amlogic-s9xxx-armbian](https://github.com/ophub/amlogic-s9xxx-armbian)（GPL-2.0），感谢 ophub、unifreq 等上游贡献者。
