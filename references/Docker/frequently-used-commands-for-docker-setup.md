---
title: Frequently used commands for Docker setup
slug: frequently-used-commands-for-docker-setup
hide_title: true
tags:
- commands
- cli
- lightning
- docker
---

import Author from '@site/src/components/Author';

## TL;DR

Most assisted processes are available through the `get.fleek.network` command, where you can select to install, do a health check amongst others.

To access the menu options run the command in the shell prompt:

```sh
curl https://get.fleek.network | bash
```

:::tip
For Native setup users read the corresponding version in the section [Frequently Used Commands for Native Setup](/references/Lightning%20CLI/frequently-used-commands-for-native-setup)
:::

## Systemctl Service Management

### Enable

```sh
sudo systemctl enable docker-lightning
```

### Disable

```sh
sudo systemctl enable docker-lightning
```

### Start

```sh
sudo systemctl start docker-lightning
```

### Stop

```sh
sudo systemctl stop docker-lightning
```

### Restart

```sh
sudo systemctl restart docker-lightning
```

### Status

```sh
sudo systemctl status docker-lightning
```

## Lightning CLI via Docker

### Show keys for user config

Show the keys by running the sub-commands `keys show` and declaring the configuration file location (in-docker pathname):

```sh
sudo docker exec -it lightning-node lgtn -c /home/lgtn/.lightning/config.toml keys show
```

## Diagnostic tools

### Extended verification health check

The command show be executed from host and not in-Docker container.

```sh
curl -sS https://get.fleek.network/healthcheck | bash
```

### Health status

The command show be executed from host and not in-Docker container.

```sh
curl -w "\n" localhost:4230/health
```

### Node details

The command show be executed from host and not in-Docker container.

```sh
curl -sS https://get.fleek.network/node_details | bash
```

## Analyzing Logs

### Standard output

```sh
tail -f /var/log/lightning/output.log
```

### Standard error

```sh
tail -f /var/log/lightning/diagnostic.log
```

### Docker Container Logs

```sh
sudo docker logs -f lightning-node
```

## Interactive Container

### Execute Bash

```sh
sudo docker exec -it lightning-node bash
```

## Docker

### List Containers

```sh
sudo docker ps -a
```

### Start Container

```sh
sudo docker start <CONTAINER ID or CONTAINER NAME>
```

### Stop Container

```sh
sudo docker stop <CONTAINER ID or CONTAINER NAME>
```

### Remove Container

```sh
sudo docker rm <CONTAINER ID or CONTAINER NAME>
```

<Author
    name="Helder Oliveira"
    image="https://github.com/heldrida.png"
    title="Software Developer + DX"
    url="https://github.com/heldrida"
/>
