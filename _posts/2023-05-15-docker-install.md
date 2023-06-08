---
title: Docker install - Online
categories: [container, docker]
tag: [container, docker, install, 도커, 설치]
toc: true
toc_sticky: true
search: true
---

> [Docker Install](https://docs.docker.com/engine/install/)

## 1. Uninstall old versions

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```



## 2. Set up the repository

```bash
sudo yum install -y yum-utils
```

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```



## 3. Install Docker Engine

```bash
sudo yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash
sudo systemctl start docker
```



## 4. Grant docker privileges

```bash
sudo chmod 666 /var/run/docker.sock
```

```bash
sudo usermod -aG docker ${USER}
```

