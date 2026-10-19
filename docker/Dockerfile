FROM registry.lazycat.cloud/anpho/sassv/telegram:4b6dbb1c77be81a6

USER root
RUN apt-get update \
    && apt-get install -y --no-install-recommends chromium fonts-noto-cjk xdg-utils \
    && rm -rf /var/lib/apt/lists/*
COPY root/ /
RUN chmod 755 /usr/local/bin/whatsapp
