# DualChat

一个包含 **Telegram、WhatsApp、Discord 和 Chromium 浏览器**的远程桌面环境，适用于懒猫微服。底部任务栏提供四个常驻图标，点击启动应用，也可切换窗口、恢复最小化窗口。

[下载安装包（最新版本）](https://github.com/wlabbyflower/DualChat/releases/latest) · [1.5.0 安装包](https://github.com/wlabbyflower/DualChat/releases/download/v1.5.0/DualChat-1.5.0.lpk)

## 四个入口，一个桌面

| 入口 | 使用方式 | 数据目录（用户文稿下） |
| --- | --- | --- |
| Telegram | 桌面客户端，启动桌面时自动打开 | `telegram/.local/share/TelegramDesktop` |
| WhatsApp | 官方 WhatsApp Web，手机扫码关联设备 | `telegram/whatsapp` |
| Discord | 官方 Discord Web，网页登录 | `telegram/discord` |
| Chromium | 带地址栏、标签页和书签的通用浏览器 | `telegram/chromium` |

Chromium 图标位于底部 Discord 图标右侧。浏览器与两个网页聊天应用分别使用独立资料目录，登录状态互不影响；关闭窗口后固定图标仍保留。桌面右键菜单也可启动应用。

这里的浏览器是 **Chromium**（Chrome 的开源基础），不是 Google Chrome 官方发行版。WhatsApp 和 Discord 使用 Chromium 独立应用窗口；项目不包含消息机器人或 API 转发服务。

## 安装与使用

1. 在懒猫微服系统启用 TUN，确认其覆盖应用容器出站流量和 DNS。只在电脑或手机开启代理不会让微服容器自动使用代理。
2. 从 Releases 下载 LPK 并安装，打开 DualChat 进入远程桌面。当前安装包面向 **linux/amd64**，需要 lzcos 1.5.0+。
3. 点击底部图标启动相应应用。Telegram 按官方流程登录；WhatsApp 用手机“关联设备”扫码；Discord 在官方页面登录。
4. 点击第四个 Chromium 图标，即可输入网址浏览网页。通过任务栏窗口按钮切换、最小化或恢复窗口。

中文输入可在左侧 KasmVNC 设置启用“本地输入法”；手机端使用双指滚动。音视频通话、摄像头及麦克风转发尚未验证。

应用不内置代理。系统 TUN 需允许懒猫应用域名和内部网络直连，以免影响远程桌面访问。如 Telegram 曾配置手动 SOCKS/HTTP 代理，请在其设置中关闭。

## 升级与数据保存

升级沿用原包 ID、服务名与 `/config` 绑定。请先备份用户文稿中的整个 `telegram` 目录，其中包括聊天会话、网页登录资料和通用浏览器数据。Chromium 及中文字体已预装，无需手动安装。

当前沿用用户文稿目录绑定；lzcos 1.7.0 起需要管理员授权。旧版 `v2raya` 数据不会主动删除，但已不再使用。Chromium 版本随镜像重新构建更新。

Telegram 在容器启动时后台检查官方 Linux 桌面版更新，下载到 `telegram/.dualchat/tdesktop/<版本>/`，下次打开生效；更新版本启动失败时回退到镜像内置版本。通过官方 `externalupdater.d` 标记关闭内置更新提示。排查或手动回退时可移除 `.dualchat/tdesktop/current` 指针；后续启动容器仍会执行自动更新检查。

## 常见问题

| 问题 | 检查方式 |
| --- | --- |
| 聊天应用或网页无法连接 | 检查系统 TUN 是否覆盖容器流量及 DNS，检查 Telegram 手动代理 |
| 点击固定图标没有窗口 | 查看 `telegram/dualchat/logs/launch.log` 与对应的 `<应用>-output.log`；浏览器输出为 `browser-output.log` |
| Telegram 更新或版本异常 | 查看 `telegram/dualchat/logs/telegram-update.log` 和 `telegram.log` |
| 桌面菜单缺失 | 在终端运行 `dualchat-launch browser`、`dualchat-launch whatsapp` 或 `dualchat-launch discord`，检查初始化日志 |
| 查看容器服务日志 | 在项目目录运行 `lzc-cli project log -s telegramv -f` |

所有任务栏入口使用统一启动器，限制文件句柄软上限、脱离任务栏进程并记录输出；启动时提示，出现窗口后记录耗时，超时提供日志信息。

Chromium 在当前容器中使用 `--no-sandbox`，与 Telegram 共用容器及数据权限，请使用可信网页。

## 本地构建

需要 lzc-cli 2.0.9+、本机 Docker/buildx。基础镜像固定为懒猫仓库版本，通过 `images` / `embed:messenger` 构建并内嵌扩展层。

```bash
git clone https://github.com/wlabbyflower/DualChat.git
cd DualChat
lzc-cli project build -o dualchat-1.5.0.lpk
```

主要文件：`docker/root/etc/dualchat/tint2rc` 配置底部图标；`docker/root/usr/local/bin/dualchat-launch` 处理统一启动；`dualchat-browser` 使用 `/config/chromium` 保存通用浏览器资料。

本次已通过 Shell 语法、Git 差异检查与 LPK 构建；本地容器实测 Chromium 约 1 秒创建普通浏览器窗口，并生成独立资料目录。构建与本地验证不代表微服完整运行验证，升级后请确认聊天收发、重启后登录状态及多用户隔离。

## 项目来源

基于 [anpho/lzc-telegram-v](https://github.com/anpho/lzc-telegram-v) 改造，加入 WhatsApp Web、Discord Web、Chromium 通用浏览器和底部任务栏，使用系统网络。

现有远程桌面基于 KasmVNC；仓库内微软雅黑字体的再分发授权仍需确认。应用图标与商店多语言元数据尚需按商店要求完善。
