# DualChat

[下载安装包（LPK）](https://github.com/wlabbyflower/DualChat/releases/latest)

在同一个远程桌面中使用 Telegram 桌面客户端、官方 [WhatsApp Web](https://web.whatsapp.com/) 和 [Discord Web](https://discord.com/app)。WhatsApp 与 Discord 使用 Chromium 打开独立窗口，不包含消息机器人或 API 转发功能。

## 使用

1. 在懒猫微服系统启用 TUN，并确认规则覆盖应用容器的出站流量及 DNS。仅在电脑或手机上开启代理不会让微服容器自动使用代理。
2. 安装 LPK，打开应用即可使用 Telegram。
3. 底部左侧常驻 Telegram、WhatsApp、Discord 三个品牌图标，悬停可查看名称，关闭窗口后入口仍保留。点击图标即启动对应应用（Telegram 已在运行时会把窗口提到前面），并在冷启动时弹出“正在启动”提示；点击任务栏中的窗口按钮可在三个应用之间切换，也可恢复已最小化的窗口。右击桌面的应用菜单仍可使用。
4. 使用手机 WhatsApp 的“关联设备”扫描二维码登录；Discord 在官方登录页登录。

三个图标、右键菜单和自启动都指向统一入口 `/usr/local/bin/dualchat-launch`：启动前压低文件句柄软上限、用 `setsid` 脱离任务栏进程、输出写日志，并在窗口迟迟不出现时激活已有窗口或明确报错，不会再出现“点了没反应”。启动记录在 `文稿/telegram/dualchat/logs/launch.log`，应用自身输出在 `dualchat/logs/<应用>-output.log`。

## Telegram 版本与自动更新

镜像内固定版本的 Telegram 自带更新器会一直提示“发现新版本”，而容器里点更新又会失败。1.4.0 起改为：

- 用 tdesktop 官方支持的 `externalupdater.d` 标记关闭自带更新检查（`/opt/Telegram/externalupdater.d/dualchat` 与 `文稿/telegram/.local/share/TelegramDesktop/externalupdater.d/dualchat`），不再弹窗；
- 容器启动时后台检查 `https://telegram.org/dl/desktop/linux`，有新版本就下载解包到 `文稿/telegram/.dualchat/tdesktop/<版本>/`，下次打开 Telegram 生效，旧版本自动清理；
- 新版本若启动失败会回退到镜像内置版本并提示；
- 状态见 `dualchat/logs/telegram-update.log`，启动细节见 `dualchat/logs/telegram.log`。

不希望自动更新时，删除或改名 `文稿/telegram/.dualchat/tdesktop/current` 即可回到镜像内置版本（自带更新提示已关闭，不会重弹）。

应用不再内置 v2rayA，也不再开放 2017、20170、20171、20172 端口。不需要配置 HTTP_PROXY 或 SOCKS 端口。系统 TUN 必须允许懒猫应用域名和内部网络正常直连，以免影响远程桌面连接。

## 升级与数据

- 包 ID、服务名和 `/config` 的旧绑定路径保持不变：用户文稿的 `telegram` 目录保存 Telegram 会话；其中的 `whatsapp`、`discord` 子目录分别保存两个网页客户端的 Chromium 登录资料。升级前请备份整个目录。
- 如果 Telegram 中配置过手动 SOCKS/HTTP 代理，请在 Telegram 设置中关闭，改为使用系统网络。
- 旧 `/lzcapp/var/v2raya` 数据不会主动删除，但不再使用。
- 当前仍沿用旧文稿目录绑定；lzcos 1.7.0 起需要管理员授权。未来若迁移至应用私有存储，应显式迁移数据，避免让用户重新登录。
- Chromium 及中文字体预装在镜像中，无需安装后手动执行 apt。浏览器版本随重新构建镜像更新；持久化的仅是登录资料。

## 构建

需要 lzc-cli 2.0.9+、lzcos 1.5.0+ 和本机 Docker/buildx。默认使用本机构建器，目标架构为 linux/amd64；其他架构需另行确认基础镜像支持。基础镜像沿用原项目固定的懒猫仓库版本，通过 `images` 构建并内嵌 Chromium 扩展层。

```bash
lzc-cli project build -o dualchat-1.4.0.lpk
```

本地容器已验证远程桌面 HTTP 返回 200、Telegram 与 WhatsApp 浏览器进程启动，以及菜单重复初始化不产生重复入口。1.2.1 增加底部 48 像素任务栏，已通过模拟点击验证两个应用窗口交替获得焦点，并确认桌面工作区为任务栏留出空间。任务栏随桌面会话启动，兼容已有 Openbox 自启动配置。1.3.0 加入 Discord 独立浏览器资料、任务栏启动器和右键菜单，已在本地验证点击启动、与 Telegram 切换窗口，以及菜单重复初始化不产生重复入口。构建结果不等于微服上的运行验证。安装后需检查：Telegram 收发消息、WhatsApp 扫码及收发消息、Discord 登录及收发消息、重启后三者仍保持登录、多用户会话隔离，以及开启系统 TUN 后远程桌面仍可访问。音视频通话、摄像头及麦克风转发尚未验证。

1.3.2 修复微服上固定启动图标点击后长时间无响应：任务栏启动前限制文件句柄软上限，避免 tint2 在微服的十亿级默认上限下逐项清理句柄。已在微服实测点击 Discord 图标后两秒内创建窗口。

1.4.0 把三个固定图标改为统一启动器（`dualchat-launch`），补全 tint2 背景定义与任务栏单实例锁，并在容器内实现 Telegram 自动更新。本地容器实测（`--ulimit nofile=1048576`，模拟微服的大句柄上限）：三个图标各在 1 秒内创建窗口，首次启动前弹出提示；Telegram 已在运行状态下点击立即激活窗口；桌面会话重启不会产生第二层任务栏；自动更新从 telegram.org 下载 7.3.5 并接管启动，`externalupdater.d` 标记指向当前二进制。

## 常见问题

- Telegram、WhatsApp 或 Discord 无法连接：先确认系统 TUN 接管容器流量及 DNS，再检查 Telegram 是否残留手动代理。只有设置系统 HTTP 代理环境变量不等于开启 TUN。
- 点击底部固定图标没反应：先看 `文稿/telegram/dualchat/logs/launch.log`（会记录每次点击、耗时、是否激活旧窗口），再看 `<应用>-output.log` 里的实际错误。
- Telegram 版本不对：看 `dualchat/logs/telegram-update.log`；删掉 `.dualchat/tdesktop/current` 或写入其内容为 `broken` 版本号即回到镜像内置版本。
- 没有 WhatsApp 或 Discord 菜单：在 xterm 执行 `whatsapp` 或 `discord`；检查日志中自定义初始化脚本是否执行。
- 启用中文输入：在左侧 KasmVNC 设置中启用“本地输入法”。
- 手机上滚动：双指滚动。
- 查看服务日志：`lzc-cli project log -s telegramv -f`。

Chromium 在当前容器环境中无法启用内置沙箱，因此启动器使用 `--no-sandbox`。浏览器和 Telegram 共用容器及数据权限；请只使用可信页面。

## 后续优化

当前 Telegram/KasmVNC 基础镜像仍是原项目版本（KasmVNC 1.3.3，2025-05 构建）。远程桌面流畅度有两个可做的方向：

1. 换成新版 KasmVNC 1.5.2 并启用视频流模式（`-videoCodec h264_vaapi`，需要微服 iGPU 与 `intel-media-va-driver-non-free`）。本地已验证官方 `kasmvncserver_debian_bookworm_1.5.2_amd64.deb` 能直接装在当前基础镜像上，用与现有一致的 `Xvnc` 参数启动正常、HTTP 返回 200；注意 1.5.x 把网页界面也放在 websocketPort 上，nginx 的 `proxy_pass` 需从 6900 改为 6901。
2. 迁移到 Selkies（LinuxServer 已用它整体替换 KasmVNC 基础镜像）。默认走 WebSocket 单端口，不需要 TURN/UDP，适合懒猫的 subdomain 反代；增量分条编码 + H.264/H.265/AV1 + Opus 音频（KasmVNC 完全没有音频）。工作量比上面一项大：需要换基础镜像、重写 `startwm.sh` 与 nginx 配置。

另外，现有微软雅黑字体文件的再分发授权也需确认，可后续改为只使用已预装的 Noto CJK。应用图标采用用户提供的 DualChat 图标，目前仍超过应用商店建议的 200 KiB，且缺少商店多语言元数据；商店发布前应处理。

## 项目来源

基于 [anpho/lzc-telegram-v](https://github.com/anpho/lzc-telegram-v) 改造，加入 WhatsApp Web、Discord Web 和底部任务栏，并改为使用系统网络。
