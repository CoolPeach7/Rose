# 🌹 Rose — 英雄联盟国服适配

<div align="center">
  <img src="./assets/icon.png" alt="Rose 图标" width="128" height="128">

  **基于 Rose 的中文分支，修复国服 / WeGame 客户端的皮肤加载与 Pengu Loader 接管问题。**

  [原项目](https://github.com/Alban1911/Rose) · [MIT 许可证](LICENSE) · [开发与项目结构](CONTRIBUTING.md)
</div>

## 项目简介

Rose 是《英雄联盟》的开源本地皮肤显示工具，通过 Pengu Loader 插件读取选人界面的皮肤选择，由 Python 后端在游戏启动时加载对应资源。

**Rose 重点适配国服与 WeGame 的客户端行为。** 本分支解决“选择皮肤后进入游戏没有显示”以及“关闭独立 Pengu Loader 后，Rose 自带加载器无法接管”的问题。它基于上游 Rose 1.2.14 开发，保留原项目的皮肤、炫彩、自定义外观及客户端插件功能；不改变账号的皮肤所有权。

## 国服适配内容

| 问题 | 本分支的处理 |
| --- | --- |
| 国服客户端的 `lockfile` 为空，Rose 无法连接 LCU | 从本机 `LeagueClient` / `LeagueClientUx` 的启动参数获取连接信息，并在客户端重启后刷新；认证信息只在内存中使用 |
| WeGame 的 `Game` 与 `LeagueClient` 是并列目录 | 自动识别并保存这种安装结构，也保留国际服目录识别 |
| 中文目录或配置出现 GBK 解码错误 | 统一按 UTF-8 读写配置，并兼容 UTF-8 BOM |
| 游戏进程名称存在差异 | 同时识别 `League of Legends.exe` 和 `League of Legends (TM) Client.exe` |
| 进入对局后客户端界面退出，Rose 误判游戏结束 | 根据实际游戏进程维持皮肤加载，游戏退出后再清理 |
| 独立 Pengu 停用后，内置加载器无法接管 | 运行期间检查激活状态，在合适的客户端阶段自动启用 Rose 自带加载器并刷新界面 |

本次实测机器上的游戏文件名是 `League of Legends.exe`，任务管理器可能显示其描述 `League of Legends (TM) Client`。因此，本次修复同时处理了连接、目录和加载生命周期，而不只是改进程名称。

### Pengu Loader 的协作方式

- 如果独立 Pengu 已启用，且安装了 `ROSE-UI` 与 `ROSE-SkinMonitor` 插件，Rose 会继续使用它。
- 独立 Pengu 停用后，Rose 会尝试启用自带加载器，无需重启 Rose。
- **独立 Pengu 窗口也需要退出。** 两种加载器共用官方 GUI 互斥锁；窗口仍在运行时，Rose 会等待它关闭。
- 检测到游戏正在运行或处于选英雄阶段时，接管操作会推迟；刷新客户端前还会再次检查游戏状态。
- 仅关闭 Pengu 窗口并不一定代表停用其激活状态。是否接管取决于当前加载器的实际注册状态。

## 已验证的范围

- 国服 / WeGame 实际对局中，用户已确认所选皮肤正常显示。
- 停用独立 Pengu 并退出其窗口后，已验证 Rose 自动激活内置加载器、刷新客户端，皮肤监视插件重新连接。
- 已检查运行中的客户端确实加载了 Rose 自带运行目录下的 `core.dll`。
- 62 项自动测试通过，包括原有 Pengu 生命周期测试，以及国服连接、目录、中文配置、加载持续时间和自动接管回归测试。
- 国际服目录与连接行为有自动回归测试覆盖，本次未进行国际服实际对局验证。

上述验证对应本次测试环境，不代表已覆盖所有大区、所有游戏补丁或所有 Windows 配置。

## 环境要求

- Windows 10 / 11。
- 已安装《英雄联盟》国服客户端 / WeGame。
- 源码运行或构建需要 Python 3.11 及以上。
- 皮肤加载所需 DLL 按 Rose 原项目要求自行准备；本仓库不提供该 DLL，也不新增游戏文件或加载器 DLL 的补丁。

## 获取与构建

**本分支目前提供修复源码，尚未发布包含这些国服修复的安装包。** 上游原版安装包不等同于本分支修复版。

克隆本仓库并安装依赖：

```powershell
git clone https://github.com/CoolPeach7/Rose.git
cd Rose
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

构建前还需要 Visual Studio Build Tools，其中包含 .NET 桌面构建工具、WPF 支持和 .NET Framework 4.7.2 目标包。若要生成安装程序，还需 Inno Setup 6。

```powershell
# 构建 Rose，同时编译仓库自带的 Pengu Loader 源码
.\.venv\Scripts\python.exe scripts/build_pyinstaller.py

# 或者：构建 Rose 并生成 Windows 安装程序
.\.venv\Scripts\python.exe scripts/build_all.py
```

程序输出在 `dist/Rose/`，安装程序输出在 `installer/Rose_Setup.exe`。请使用上述构建脚本，以便先编译 `vendor/PenguLoader-1.1.6/`；不要跳过这一步直接调用 `pyinstaller Rose.spec`。

仅构建加载器时可运行：

```powershell
.\.venv\Scripts\python.exe scripts/build_pengu_loader.py
```

## 国服使用方法

1. 启动国服客户端并进入大厅，然后启动修复版 Rose。
2. 如自动识别失败，在设置中指定包含游戏可执行文件的 `Game` 目录，例如 `D:\WeGameApps\LOL\Game`。
3. 使用 Rose 自带加载器时，先停用独立 Pengu 并退出其窗口，等待客户端界面刷新、插件重新连接。
4. 建议先在训练模式选择皮肤并进入游戏，确认所选皮肤显示正常。
5. Rose 会在系统托盘中运行；游戏结束后自动停止本次加载并清理临时资源。

**更新说明：** 本分支仍保留上游自动更新逻辑，更新地址尚未改为独立的国服发行渠道。安装上游版本可能覆盖这里的修复；更新前请核对该版本是否已包含本分支修复。

## 排查问题

数据与日志默认位于 `%LOCALAPPDATA%\Rose\`：

- `logs\`：Rose 运行日志，可检查 LCU 连接、Pengu 激活和插件连接情况。
- `injection\runoverlay.log`：游戏资源加载进程的输出。
- `Pengu Loader\`：打包运行时使用的内置加载器目录。

如果皮肤仍未生效，请记录客户端版本、启动方式、问题发生阶段、是否使用独立 Pengu，以及相关错误片段。分享日志前请移除账号信息、认证令牌等个人数据。

## 运行测试

在仓库根目录执行：

```powershell
.\.venv\Scripts\python.exe -m unittest test.test_regional_compat test.test_pengu_loader
```

测试使用模拟进程、模拟 LCU 响应与临时目录，不需要启动实际对局。

## 使用统计

本分支保留上游的统计功能：启用时，会向 `https://analytics.rosekeys.site/` 发送随机生成的安装标识和应用版本，并包含启动、在线心跳和退出通知。该标识不是 Windows Machine GUID。可在 `config.py` 中将 `ANALYTICS_ENABLED` 设为 `False` 后运行或重新构建，以关闭统计。

统计服务由上游维护，本分支没有部署独立统计服务。

## 来源与许可

- 原项目：[Alban1911/Rose](https://github.com/Alban1911/Rose)。核心功能、素材与原有插件的贡献归原作者及相应贡献者所有。
- 加载器：[PenguLoader/PenguLoader](https://github.com/PenguLoader/PenguLoader)，相关源码位于 `vendor/PenguLoader-1.1.6/`，请遵守其[许可证](https://github.com/PenguLoader/PenguLoader/blob/main/LICENSE)。
- 本项目沿用仓库的 [MIT 许可证](LICENSE)，保留原有版权声明。
- 本项目与 Riot Games、腾讯及 WeGame 没有隶属或官方合作关系。
