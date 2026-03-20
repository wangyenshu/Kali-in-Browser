FROM i386/debian:bookworm-slim

WORKDIR /root
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    linux-image-686 \
    systemd-sysv \
    locales \
    wget gnupg dirmngr ca-certificates curl && \
    curl -s https://archive.kali.org/archive-key.asc -o /etc/apt/trusted.gpg.d/kali-archive-key.asc && \
    echo "deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free contrib" > /etc/apt/sources.list && \
    rm -f /etc/apt/sources.list.d/* && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    isc-dhcp-client sudo ifupdown iproute2 iputils-ping net-tools dnsutils procps htop kmod nano vim \
    tmux less file tree jq lsof strace tcpdump unzip p7zip-full xz-utils \
    git build-essential python3-pip python3-venv socat proxychains4 openssh-client \
    rlwrap smbclient cifs-utils openssl whois && \
    apt-get dist-upgrade -y && \
    apt-get install -y kali-linux-headless && \
    rm -rf /var/lib/apt/lists/*

RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && \
    locale-gen && \
    echo 'LANG="en_US.UTF-8"' > /etc/default/locale && \
    chsh -s /bin/bash root  

COPY getty-noclear.conf getty-override.conf /etc/systemd/system/getty@tty1.service.d/
COPY getty-autologin-serial.conf /etc/systemd/system/serial-getty@ttyS0.service.d/

RUN systemctl enable serial-getty@ttyS0.service

RUN systemctl disable systemd-timesyncd.service || true; \
    systemctl disable apt-daily.timer || true; \
    systemctl disable apt-daily-upgrade.timer || true

RUN printf '%s\n' 9p 9pnet 9pnet_virtio virtio virtio_ring virtio_pci | tee -a /etc/initramfs-tools/modules

RUN echo '#!/bin/sh' > /etc/initramfs-tools/scripts/boot-9p && \
    echo 'case $1 in prereqs) exit 0;; esac' >> /etc/initramfs-tools/scripts/boot-9p && \
    echo '. /scripts/functions' >> /etc/initramfs-tools/scripts/boot-9p && \
    echo 'mountroot() {' >> /etc/initramfs-tools/scripts/boot-9p && \
    echo '    mkdir -p ${rootmnt}' >> /etc/initramfs-tools/scripts/boot-9p && \
    echo '    mount -n -t 9p -o trans=virtio,version=9p2000.L,cache=loose,rw host9p ${rootmnt}' >> /etc/initramfs-tools/scripts/boot-9p && \
    echo '}' >> /etc/initramfs-tools/scripts/boot-9p && \
    chmod +x /etc/initramfs-tools/scripts/boot-9p

RUN echo 'BOOT=boot-9p' | tee -a /etc/initramfs-tools/initramfs.conf
RUN update-initramfs -u

RUN { \
    echo 'echo "127.0.0.1 localhost" > /etc/hosts'; \
    echo 'echo "localhost" > /etc/hostname'; \
    echo 'hostname localhost'; \
    echo 'echo -e "\n\nCLI_READY\n" > /dev/ttyS0'; \
} >> /root/.bashrc

WORKDIR /root
