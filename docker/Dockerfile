FROM registry.lazycat.cloud/anpho/sassv/telegram:4b6dbb1c77be81a6

USER root
RUN apt-get update \
    && apt-get install -y --no-install-recommends chromium fonts-noto-cjk xdg-utils tint2 wmctrl \
    && rm -rf /var/lib/apt/lists/*
COPY root/ /

# 关闭镜像内 Telegram 自带的更新检查：tdesktop 只要在可执行目录的
# externalupdater.d/ 下发现记录了自己路径的文件，就不会创建 UpdateChecker，
# 于是既不联网查版本也不再弹更新提示。版本改由 dualchat-telegram-update 自动更新。
RUN mkdir -p /opt/Telegram/externalupdater.d \
    && printf '/opt/Telegram/Telegram\n' >/opt/Telegram/externalupdater.d/dualchat \
    && chmod 755 /usr/local/bin/whatsapp /usr/local/bin/discord \
        /usr/local/bin/dualchat-browser /usr/local/bin/dualchat-panel /usr/local/bin/dualchat-launch \
        /usr/local/bin/dualchat-telegram /usr/local/bin/dualchat-telegram-update

# 桌面会话入口：先起底部任务栏和后台更新检查，再进原来的 openbox 会话。
RUN mv /defaults/startwm.sh /defaults/startwm-upstream.sh \
    && printf '%s\n' \
        '#!/bin/sh' \
        '/usr/local/bin/dualchat-panel &' \
        '/usr/local/bin/dualchat-telegram-update &' \
        'exec /bin/bash /defaults/startwm-upstream.sh' >/defaults/startwm.sh \
    && chmod 755 /defaults/startwm.sh
