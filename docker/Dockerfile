FROM registry.lazycat.cloud/anpho/sassv/telegram:4b6dbb1c77be81a6

USER root
RUN apt-get update \
    && apt-get install -y --no-install-recommends chromium fonts-noto-cjk xdg-utils tint2 \
    && rm -rf /var/lib/apt/lists/*
COPY root/ /
RUN chmod 755 /usr/local/bin/whatsapp /usr/local/bin/discord \
    && mv /defaults/startwm.sh /defaults/startwm-upstream.sh \
    && printf '#!/bin/sh\n/usr/local/bin/dualchat-panel &\nexec /bin/bash /defaults/startwm-upstream.sh\n' > /defaults/startwm.sh \
    && chmod 755 /defaults/startwm.sh /usr/local/bin/dualchat-panel
