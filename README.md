# DualChat

[下载安装包（LPK）](https://github.com/wlabbyflower/DualChat/releases/latest)

在同一个远程桌面中使用 Telegram 桌面客户端和官方 [WhatsApp Web](https://web.whatsapp.com/)。WhatsApp 使用 Chromium 打开独立窗口，不包含消息机器人或 API 转发功能。

## 使用

1. 在懒猫微服系统启用 TUN，并确认规则覆盖应用容器的出站流量及 DNS。仅在电脑或手机上开启代理不会让微服容器自动使用代理。
2. 安装 LPK，打开应用即可使用 Telegram。
3. 将 Telegram 窗口取消最大化，右击桌面，选择 **WhatsApp**；也可以打开 xterm 输入 `whatsapp`。
4. 使用手机 WhatsApp 的“关联设备”扫描二维码登录。

应用不再内置 v2rayA，也不再开放 2017、20170、20171、20172 端口。不需要配置 HTTP_PROXY 或 SOCKS 端口。系统 TUN 必须允许懒猫应用域名和内部网络正常直连，以免影响远程桌面连接。

## 升级与数据

- 包 ID、服务名和 `/config` 的旧绑定路径保持不变：用户文稿的 `telegram` 目录保存 Telegram 会话；其中的 `whatsapp` 子目录保存 Chromium 登录资料。升级前请备份整个目录。
- 如果 Telegram 中配置过手动 SOCKS/HTTP 代理，请在 Telegram 设置中关闭，改为使用系统网络。
- 旧 `/lzcapp/var/v2raya` 数据不会主动删除，但不再使用。
- 当前仍沿用旧文稿目录绑定；lzcos 1.7.0 起需要管理员授权。未来若迁移至应用私有存储，应显式迁移数据，避免让用户重新登录。
- Chromium 及中文字体预装在镜像中，无需安装后手动执行 apt。浏览器版本随重新构建镜像更新；持久化的仅是登录资料。

## 构建

需要 lzc-cli 2.0.9+、lzcos 1.5.0+ 和本机 Docker/buildx。默认使用本机构建器，目标架构为 linux/amd64；其他架构需另行确认基础镜像支持。基础镜像沿用原项目固定的懒猫仓库版本，通过 `images` 构建并内嵌 Chromium 扩展层。

```bash
lzc-cli project build -o telegram-whatsapp.lpk
```

本地容器已验证远程桌面 HTTP 返回 200、Telegram 与 WhatsApp 浏览器进程启动，以及菜单重复初始化不产生重复入口。构建结果不等于微服上的运行验证。安装后需检查：Telegram 收发消息、WhatsApp 扫码及收发消息、重启后两者仍保持登录、多用户会话隔离，以及开启系统 TUN 后远程桌面仍可访问。音视频通话、摄像头及麦克风转发尚未验证。

## 常见问题

- Telegram 或 WhatsApp 无法连接：先确认系统 TUN 接管容器流量及 DNS，再检查 Telegram 是否残留手动代理。只有设置系统 HTTP 代理环境变量不等于开启 TUN。
- 没有 WhatsApp 菜单：在 xterm 执行 `whatsapp`；检查日志中自定义初始化脚本是否执行。
- 启用中文输入：在左侧 KasmVNC 设置中启用“本地输入法”。
- 手机上滚动：双指滚动。
- 查看服务日志：`lzc-cli project log -s telegramv -f`。

Chromium 在当前容器环境中无法启用内置沙箱，因此启动器使用 `--no-sandbox`。浏览器和 Telegram 共用容器及数据权限；请只使用可信页面。

## 后续优化

当前 Telegram/KasmVNC 基础镜像仍是原项目版本，需另行评估升级并验证配置兼容性。现有微软雅黑字体文件的再分发授权也需确认，可后续改为只使用已预装的 Noto CJK。应用图标目前超过应用商店建议的 200 KiB，且缺少商店多语言元数据；发布前应处理。

## 项目来源

基于 [anpho/lzc-telegram-v](https://github.com/anpho/lzc-telegram-v) 改造，加入 WhatsApp Web，并改为使用系统网络。
