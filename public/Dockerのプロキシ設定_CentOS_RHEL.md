---
title: 'Dockerのプロキシ設定（CentOS, RHEL）'
tags:
  - CentOS
  - Docker
  - RHEL
  - Podman
private: false
updated_at: '2022-09-12T17:36:36+09:00'
id: 1e737578c66e60730392
organization_url_name: null
slide: false
ignorePublish: false
---
毎度忘れてしまうので、メモ。
社内プロキシ下などで`docker pull`するために必要なプロキシの設定です。

## CentOS, RHEL7系の場合

- ディレクトリ作成
```
$ sudo mkdir -p /usr/lib/systemd/system/docker.service.d
```

- ファイル作成＆環境変数設定
```
$ sudo vi /usr/lib/systemd/system/docker.service.d/http-proxy.conf
```

```ini
[Service]
Environment="HTTP_PROXY=http://user:pass@proxy.domain:port"
Environment="HTTPS_PROXY=http://user:pass@proxy.domain:port"
Environment="NO_PROXY=localhost,127.0.0.1"
```

- Docker再起動
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

## RHEL8以降の場合

Dockerの代わりに[Podman](https://podman.io/)を使うことが推奨されており、パスが若干違います。
※なお、podmanは`docker`コマンドとの互換性があるので、基本的にはDockerと同じように使えます。

- ディレクトリ作成

```bash
$ ls /usr/lib/systemd/system/podman
podman-auto-update.service  podman-auto-update.timer    podman-restart.service      podman.service              podman.socket               
```

```bash
$ sudo mkdir -p /usr/lib/systemd/system/podman.service.d
```

- ファイル作成＆環境変数設定
```bash
$ sudo vi /usr/lib/systemd/system/podman.service.d/http-proxy.conf
```

```ini
[Service]
Environment="HTTP_PROXY=http://user:pass@proxy.domain:port"
Environment="HTTPS_PROXY=http://user:pass@proxy.domain:port"
Environment="NO_PROXY=localhost,127.0.0.1"
```

- Podman再起動
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart podman
```
