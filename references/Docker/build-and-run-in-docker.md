---
title: Build and run in Docker
slug: build-and-run-in-docker
hide_title: true
tags:
- references
- help
- docker
- build
- image
- container
---

import Author from '@site/src/components/Author';
import GitCloneOptions from '../../guides/partials/_git-clone-options.mdx';

## Clone the source code locally

<GitCloneOptions />

## Change directory to Lightning source code

```sh
cd ~/fleek-network/lightning
```

## Install Docker

:::tip
We're using Ubuntu Linux. You'll have to make the required tweaks for your preferred Linux Distro. Find the list of support operating systems [here](/docs/node/requirements#server).
:::

```sh
sudo apt update
```

Next, install the required packages to let apt use packages over HTTPS:

```sh
sudo apt install \
  apt-transport-https \
  ca-certificates \
  software-properties-common
```

Add the GPG key for the official Docker repository to your system:

```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add the Docker repository to apt sources and update the package database with the Docker packages from the new added repository:

```sh
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu jammy stable"
```

Set to install from the Docker repo instead of the default Ubuntu repo:

```sh
apt-cache policy docker-ce
```

Finally, install Docker:

```sh
sudo apt install docker-ce
```

Once complete you should be able to run it via the CLI, as such:

```sh
docker -v
```

Here's the output (versions might differ a bit from the time of writing):

```sh
Docker version 24.0.6, build ed223bc
```

## Docker setup verification

The following command's output will indicate if Docker's working correctly:

```sh
sudo docker run hello-world
```

You should get a "Hello from Docker!".

## Create the Dockerfile

```sh
touch Dockerfile
```
Copy and paste to the Dockerfile the following content:

```sh
FROM rust:latest as builder
ARG PROFILE=release
WORKDIR /lightning

RUN apt-get update
RUN apt-get install -y \
    build-essential \
    cmake \
    clang \
    pkg-config \
    libssl-dev \
    gcc \
    protobuf-compiler

RUN --mount=type=cache,target=/usr/local/cargo/registry \
    cargo install cargo-strip

COPY . .
ENV RUST_BACKTRACE=1

RUN --mount=type=cache,target=/usr/local/cargo/registry \
    --mount=type=cache,target=/lightning/target \
    cargo build --profile $PROFILE --bin lightning-cli \
    && cargo strip \
    && mv /lightning/target/release/lightning-cli /lightning-cli

FROM ubuntu:latest

RUN DEBIAN_FRONTEND=noninteractive apt-get update -yq && \
  DEBIAN_FRONTEND=noninteractive apt-get install -yq \
    libssl-dev \
    ca-certificates

# Get compiled binaries from builder's cargo install directory
COPY --from=builder /lightning/target/release/lightning-cli /usr/local/bin/lgtn

ENTRYPOINT ["lgtn", "run"]
```

## Build the Docker image

Build the image named as `lightning` from our Dockerfile:

```sh
sudo docker build -t lightning -f ./Dockerfile .
```

## Generate keys

```sh
sudo docker exec -it lightning-cli lgtn keys generate
```

## Docker Container

```sh
sudo docker run \
  -p 4069:4069 \
  -p 4200:4200 \
  -p 6969:6969 \
  -p 18000:18000 \
  -p 18101:18101 \
  -p 18102:18102 \
  -v $HOME/.lightning/:/root/.lightning/:rw \
  --name lightning-cli \
  -it lightning
```

## View logs

To view the logs of a Docker container in real time, use the following command:

```sh
sudo docker logs -f lightning-cli
```

<Author
    name="Helder Oliveira"
    image="https://github.com/heldrida.png"
    title="Software Developer + DX"
    url="https://github.com/heldrida"
/>