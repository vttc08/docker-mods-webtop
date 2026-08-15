ARG WEBTOP_VERSION=37509c6b
FROM lscr.io/linuxserver/webtop:ubuntu-mate-version-${WEBTOP_VERSION}

ARG SPACEFM_VERSION=1.0.6+gtk2.20260815
ENV DEBIAN_FRONTEND=noninteractive

COPY spacefm-dist/spacefm-gtk2-custom_${SPACEFM_VERSION}_amd64.deb /tmp/

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      /tmp/spacefm-gtk2-custom_${SPACEFM_VERSION}_amd64.deb \
      wget curl rsync vim unrar unzip xarchiver p7zip libmpv2 vlc \
      ffmpeg mediainfo mediainfo-gui fzf firefox less && \
    rm -f /tmp/spacefm-gtk2-custom_*.deb && \
    apt-get clean && rm -rf /var/lib/apt/lists/*
