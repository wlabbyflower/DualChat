DualChat 1.5.0：四合一远程桌面环境。

- 同一桌面使用 Telegram 桌面客户端、WhatsApp Web、Discord Web 和 Chromium 浏览器。
- 底部任务栏新增第四个 Chromium 固定图标，点击打开带地址栏的普通浏览器窗口；桌面右键菜单也提供入口。
- 浏览器使用独立的持久化资料目录，保存书签、历史记录和登录状态，与 WhatsApp、Discord 隔离。
- 沿用统一启动器的提示、日志及窗口检测机制。
- 升级保留原包 ID 和数据绑定，无需重新创建聊天应用资料。

浏览器为 Chromium（Chrome 的开源基础），不是 Google Chrome 官方发行版。
需要微服系统 TUN 覆盖容器流量及 DNS。当前安装包面向 linux/amd64。

验证：LPK 构建、Shell 语法及差异检查通过；本地容器实测 Chromium 在约 1 秒内创建窗口并生成独立资料目录。微服实际安装、聊天收发与音视频功能需在目标设备验证。
