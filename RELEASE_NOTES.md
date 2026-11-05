1.4.0 修复任务栏图标点击无反应，Telegram 改为容器内自动更新。

- 底部三个固定图标（Telegram / WhatsApp / Discord）改成统一走
  `/usr/local/bin/dualchat-launch`：先压文件句柄软上限、用 setsid 脱离任务栏进程、
  输出写日志，再启动；20 秒内没有新窗口就激活已有窗口，并在超时时弹提示，
  不会再出现“点了什么都没有”。启动日志在 `文稿/telegram/dualchat/logs/launch.log`。
- 任务栏改为 dock 窗口，Openbox 不再把它当普通窗口托管，避免被应用窗口盖住
  或第一次点击被吃掉；补全了 tint2 的背景定义，图标悬停/按下有反馈。
- Telegram 反复提示更新：镜像里的 Telegram 是固定旧版本，自带的更新器永远认为
  有新版本。现在用官方 `externalupdater.d` 机制彻底关闭自带更新检查，
  改由容器启动时后台检查 telegram.org 官方 Linux 包，下载到
  `文稿/telegram/.dualchat/tdesktop/<版本>/`，下次启动生效；新版本启动失败会自动
  回退到镜像内置版本。
- 自动更新状态日志：`文稿/telegram/dualchat/logs/telegram-update.log`。
