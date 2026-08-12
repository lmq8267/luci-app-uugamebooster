<div align="center">

# luci-app-uugamebooster

> OpenWrt LuCI - 网易UU游戏加速器插件

![GitHub Release](https://img.shields.io/github/v/release/lmq8267/luci-app-uugamebooster?style=flat-square&color=e94560)
![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/lmq8267/luci-app-uugamebooster/build.yml?branch=master&style=flat-square&label=build)
![GitHub Stars](https://img.shields.io/github/stars/lmq8267/luci-app-uugamebooster?style=flat-square&color=yellow)
![OpenWrt](https://img.shields.io/badge/OpenWrt-22.03%2B-brightgreen?style=flat-square)
![Arch](https://img.shields.io/badge/Arch-arm%20%7C%20aarch64%20%7C%20mipsel-blueviolet?style=flat-square)
![IPK](https://img.shields.io/badge/Package-IPK-blue?style=flat-square)
![APK](https://img.shields.io/badge/Package-APK-ff69b4?style=flat-square)
![LuCI](https://img.shields.io/badge/LuCI-compat-orange?style=flat-square)

为 OpenWrt 路由器提供 [网易UU加速器](https://uu.163.com) 的 LuCI Web 管理界面，支持在路由器上直接开启主机/PC/手机游戏加速。

</div>

---

## 功能特性

- 通过 LuCI Web 界面一键启用 / 关闭 UU 加速服务
- 支持自定义设备名称，便于在 APP 中区分多台路由器
- 实时显示服务运行状态（3 秒自动刷新）
- 内置 iOS / Android APP 下载二维码（方法可能不是最新的，具体见APP）
- 支持架构：`aarch64`、`arm`、`mipsel` 其他架构待发现

## 工作原理

本插件**不包含加速器二进制文件**。启用服务后，init 脚本会自动从网易服务器下载对应架构的 `uuplugin` 最新版本二进制并运行。

> **与 OpenWrt 通用版UU的区别：** 通用版 UU 路由器插件仅支持主机（游戏机）加速，而本插件在主机加速的基础上，还额外支持 **PC 和手机** 的加速。使用时只需在手机 APP 中同时绑定路由器和需要加速的设备即可。目前经实测，网易官方仅对 `arm`、`aarch64`、`mipsel` 三种架构提供了插件二进制下载，其他架构暂未发现可用的插件资源。

## 安装预编译包

从 [GitHub Releases](../../releases) 页面下载对应格式的安装包：

| 格式 | 文件名 | 适用版本 |
|------|--------|----------|
| ipk | `luci-app-uugamebooster_*.ipk` | OpenWrt 22.03+ |
| apk | `luci-app-uugamebooster-*.apk` | OpenWrt Snapshots |

### 安装 IPK 包

```bash
# 上传到路由器后安装
opkg update
opkg install luci-app-uugamebooster_*.ipk
```

### 安装 APK 包（含跳过证书校验）

APK 包管理器默认启用 HTTPS 证书校验，直接安装 GitHub Releases 上的包可能会因证书问题失败。以下是跳过证书校验的方法：

#### 方法一：临时禁用证书校验（推荐）

```bash
# 安装时指定 --force-signature 跳过签名验证
apk add --allow-untrusted --force-signature luci-app-uugamebooster-*.apk
```

#### 方法二：全局禁用证书校验

```bash
# 编辑 apk 仓库配置，为特定仓库禁用证书验证
sed -i 's|https://|http://|g' /etc/apk/repositories.d/*

# 或者设置环境变量
export APK_FORCE_CHECK_SIGNATURES=0
apk add luci-app-uugamebooster-*.apk
```

#### 方法三：手动导入证书后正常安装

```bash
# 如果有正确的 CA 证书，可以导入后正常安装
# 将证书放入信任目录
cp ca-cert.crt /usr/local/share/ca-certificates/
update-ca-certificates

# 然后正常安装
apk add luci-app-uugamebooster-*.apk
```

## 编译固件时集成

将本项目作为第三方包集成到 OpenWrt SDK / LEDE 编译环境中：

### 步骤 1：克隆到 package 目录

```bash
cd /path/to/openwrt
git clone https://github.com/lmq8267/luci-app-uugamebooster.git package/luci-app-uugamebooster
```

### 步骤 2：安装 feeds

```bash
./scripts/feeds update -a
./scripts/feeds install -a
```

### 步骤 3：选中插件

```bash
make menuconfig
```

进入 **LuCI → Applications** 菜单，找到 `luci-app-uugamebooster` 并选中（按 `M` 编译为模块）。

### 步骤 4：编译固件

```bash
# 仅编译该插件的 ipk 包
make package/luci-app-uugamebooster/compile V=s

# 编译完整固件（包含该插件）
make -j$(nproc) V=s
```

编译产物位于 `bin/targets/` 或 `bin/packages/` 目录下。

## 单独编译插件包

如果只想编译 `.ipk` 安装包而不编译完整固件：

```bash
cd /path/to/openwrt

# 确保已选中插件
echo "CONFIG_PACKAGE_luci-app-uugamebooster=m" >> .config
make defconfig

# 编译
make package/luci-app-uugamebooster/compile V=s
```

产物路径：`bin/packages/<arch>/base/luci-app-uugamebooster_*.ipk`

## 使用方法

1. 安装后进入 **服务 (Services)** → **网易UU游戏加速器**
2. 打开 **启用** 开关，可选填设备名称
3. 点击 **保存&应用**
4. 手机下载 UU 主机加速 APP（页面内有二维码），绑定路由器后即可加速

## 支持的架构与对应路由器

本插件通过 `uname -ms` 检测 CPU 架构，根据架构从网易服务器拉取对应的加速器二进制。

| 架构 | `uname -ms` 匹配规则 | API 接口参数 | 代表设备 |
|------|----------------------|-------------|----------|
| `arm` (32位) | `linux_armv*` | `type=h3c-bx54` | H3C BX54、中兴 AX3000 Pro 等 |
| `aarch64` (64位) | `linux_aarch64*` 或 `linux_armv8*` | `type=h3c-nx30pro` | H3C NX30 Pro、H3C Magic 等 |
| `mipsel` | `linux_mips*` (小端序) | `type=jd-hr06` | 京东云一代 等 |

> 不在以上列表中的架构（如 `x86_64`、`mips` 大端序等）将无法使用，服务启动时会提示架构不支持。

### 如何添加对新架构的支持

如果网易 UU 加速器已支持你的路由器架构，但插件尚未适配，可按以下步骤修改：

#### 1. 确认你的路由器架构

```bash
# 登录路由器执行
uname -ms
# 例如输出: Linux aarch64  → 对应 aarch64
# 例如输出: Linux armv7l   → 对应 arm
# 例如输出: Linux mips     → 需要判断大小端序
```

#### 2. 获取该设备的 API 下载地址

访问网易 UU 路由器插件 API，将 `type` 参数替换为你路由器对应的型号标识：

```bash
# 格式：https://router.uu.163.com/api/plugin?type=<设备型号>
# 例如：
curl -s "https://router.uu.163.com/api/plugin?type=h3c-nx30pro"
```

返回的 JSON 中包含 `uu.tar.gz` 的下载地址和 MD5。

#### 3. 修改 init 脚本

编辑 `root/etc/init.d/uuplugin`，需要修改以下几处：

**① 添加新的架构变量**（在文件头部变量区域）：

```bash
# 在 aarch64uurl 后面添加，例如支持 x86_64：
x86_64uurl="https://router.uu.163.com/api/plugin?type=<你的设备型号>"
```

**② 添加架构检测逻辑**（在 `start_service()` 函数中）：

```bash
# 在已有的架构检测行后面添加，例如：
[ -n "$(echo $cputype | grep -E "linux.*x86_64.*")" ] && cpucore="x86_64"
```

**③ 添加设备信息文件处理**（在 `if [ "$cpucore" = "mipsel" ]` 的分支旁）：

大多数新架构与 `arm`/`aarch64` 一样使用 `h3c_info` 格式，无需额外修改。如果设备使用其他格式参考 `haiapi` 的写法。

**④ 在 case 语句中添加下载分支**：

```bash
# 在 case "$cpucore" 语句的 arm|aarch64|mipsel 分支后添加：
  x86_64)
    uuurl="$x86_64uurl"
    ;;
```

以及在 fallback URL 部分：

```bash
  x86_64)
    UU_url="http://uu.gdl.netease.com/uuplugin/<设备型号>/<版本号>/uu.tar.gz,<md5>"
    ;;
```

**⑤ 更新错误提示**（在不支持架构的 echo 语句中加上新架构名）：

```bash
echo "当前架构不符合，无法安装！此luci-app-uugamebooster安装包仅支持arm aarch64 mipsel x86_64架构！"
```

#### 4. 完整修改示例

以添加 `x86_64` 架构为例，diff 大致如下：

```diff
 # 变量区域
 armuurl="https://router.uu.163.com/api/plugin?type=h3c-bx54"
 aarch64uurl="https://router.uu.163.com/api/plugin?type=h3c-nx30pro"
 mipseluurl="https://router.uu.163.com/api/plugin?type=jd-hr06"
+x86_64uurl="https://router.uu.163.com/api/plugin?type=<你的设备型号>"

 # 架构检测
 [ -n "$(echo $cputype | grep -E "linux.*armv.*")" ] && cpucore="arm"
 [ -n "$(echo $cputype | grep -E "linux.*aarch64.*|linux.*armv8.*")" ] && cpucore="aarch64"
+[ -n "$(echo $cputype | grep -E "linux.*x86_64.*")" ] && cpucore="x86_64"

 # case 下载分支
   arm)
     uuurl="$armuurl"
     ;;
   aarch64)
     uuurl="$aarch64uurl"
     ;;
   mipsel)
     uuurl="$mipseluurl"
     ;;
+  x86_64)
+    uuurl="$x86_64uurl"
+    ;;
```

> **提示：** 新架构的 `type` 参数和 fallback 下载地址/MD5 需要从网易 UU 官方 API 获取，不同型号的值不同。如果你不确定正确的型号标识，可以在 UU 主机加速 APP 中查看支持的路由器列表，或参考 [UU 加速器官网](https://uu.163.com) 的路由器适配信息。

## GitHub Actions 自动构建

推送到 `master` 分支或手动触发 workflow 时，CI 会自动：

1. 使用 OpenWrt SDK 分别编译 `.ipk`（22.03 稳定版 SDK）和 `.apk`（snapshot SDK）格式
2. 将编译产物上传为 Workflow Artifact
3. 创建 GitHub Release 并附带安装包

## 依赖

- `luci-compat`（LuCI 兼容层，运行时依赖）
- `kmod-tun`（UU加速时会创建tun网卡接口）

## 许可证

[Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0)

## 相关链接

- [网易UU加速器官网](https://uu.163.com)
- [OpenWrt 官网](https://openwrt.org)
- [OpenWrt Packages](https://openwrt.org/packages/)
