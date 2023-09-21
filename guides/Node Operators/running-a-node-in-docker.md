---
title: Running a node in Docker
hide_title: true
slug: running-a-node-in-docker
image: ./assets/running-a-node-in-docker.png?202311181211
date: 2023-09-18T17:00:00Z
description: A guide on how to run Fleek Network's node in a Docker container
category: Tutorial
tags:
- guide
- docker
- container
---

![Running a node in Docker](./assets/running-a-node-in-docker.png?202311181211)

<!--
  The following import is intentional (see partial <CheckoutCommitWarning />)
-->
import Author from '@site/src/components/Author';
import GitCloneOptions from '../partials/_git-clone-options.mdx';
import CreateAUser from '../../guides/partials/_create-a-user.mdx';

## Introduction

Our [Docker](https://www.docker.com/) [image](https://docs.docker.com/engine/reference/commandline/images/) provides all the requirements to have Fleek Network running quickly and the following guide will provide you a quick reference to get you started with Docker. 

Alternatively, if you need a deep dive into Docker, check the official getting started [here](https://docs.docker.com/get-started/).

## Pre-requisites

To follow the guide, you will need the following:

- Familiarity with the command-line interface
- Git

## Setup

### Requirements

To follow the guide successfully, a good amount of memory and disk space is necessary to run Docker. The main reason for our use-case is that your host machine requires a generous amount of memory and disk space, for the containers.

For this guide, we used a server with the 4vCPU, 32GB ram memory and 20 GB disk space specifications. Learn more about the recommended specifications [here](/docs/node/requirements).

### Create a user

<CreateAUser />

### Lightning CLI source code

Start by cloning the repository located at [https://github.com/fleek-network/lightning](https://github.com/fleek-network/lightning).

<GitCloneOptions />

### Change directory to Lightning source code

If you have cloned the project correctly, you should `change directory` to the project source code directory which by default is `~/fleek-network/lightning`.

```sh
cd ~/fleek-network/lightning
```

At time of writing, this is how the project root looks like (e.g. use the [ls](https://en.wikipedia.org/wiki/Ls) to see the list):

```sh
.
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── Cargo.lock
├── Cargo.toml
├── Dockerfile
├── LICENSE
├── README.md
├── codecov.yml
├── core
├── docs
├── etc
├── lib
├── rust-toolchain
├── rustfmt.toml
├── services
└── target
```

### Install Docker

:::tip
To keep our guide short, we're using Ubuntu Linux. You'll have to make the required tweaks for your preferred Linux Distro. Find the list of support operating systems [here](/docs/node/requirements#server).
:::

First, update the existing list of packages:

```sh
sudo apt update
```

Next, install the required packages to let apt use packages over HTTPS:

```sh
sudo apt install apt-transport-https ca-certificates software-properties-common
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

```sh
docker-ce:
  Installed: (none)
  Candidate: 5:24.0.6-1~ubuntu.22.04~jammy
  Version table:
     5:24.0.6-1~ubuntu.22.04~jammy 500
        500 https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
     5:24.0.6-1~ubuntu.22.04~jammy 500
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

The following command's output will indicate if Docker's working correctly:

```sh
sudo docker run hello-world
```

Here's an example of the output you'll find us "Hello from Docker!":

```sh
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

Run all the commands above in your terminal, to confirm everything's working before proceeding to the next steps.

### Create the Docker image

A Docker image is a read-only template with instructions for creating a Docker container, like a template. Docker images also act as a the starting point when using Docker. 

The starting point for our use-case is a Dockerfile, where all those "template instructions" are declared.

Create a new file `Dockerfile` in the [source code directory](#change-directory-to-lightning-source-code):

```sh
touch Dockerfile
```

Copy and paste to the Dockerfile the following content:

```sh
FROM rust:latest as build
WORKDIR /build

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

RUN mkdir -p /build/target/release
RUN --mount=type=cache,target=/usr/local/cargo/registry \
    --mount=type=cache,target=/build/target \
    cargo build --profile release --bin lightning-cli && \
    cargo strip && \
    cp /build/target/release/lightning-cli /build

FROM ubuntu:latest

RUN apt-get update && \
    apt-get install -y \
    libssl-dev \
    ca-certificates

COPY --from=build /build/lightning-cli /usr/local/bin/lgtn

ENTRYPOINT ["lgtn", "run"]
```

### Build the Docker image

Build the image named as `lightning` from our Dockerfile:

```sh
sudo docker build -t lightning -f ./Dockerfile .
```

<!-- 
TODO: Create the Docker buildkit reference

:::note
BuildKit is required by Docker Build if you are running the docker daemon (desktop users have it enabled by default), read our reference [BuildKit required by Docker build](../../reference/Docker/buildkit-required-by-docker-build) to learn more about how to enable it.
:::
-->

The build process takes awhile and you have to wait for completion. 

The output should be similar to:

```sh
[+] Building 1.2s (16/16) FINISHED                                                                                                     docker:default
 => [internal] load build definition from Dockerfile                                                                                             0.0s
 => => transferring dockerfile: 990B                                                                                                             0.0s
 => [internal] load .dockerignore                                                                                                                0.0s
 => => transferring context: 2B                                                                                                                  0.0s
 => [internal] load metadata for docker.io/library/debian:bullseye-slim                                                                          0.6s
 => [internal] load metadata for docker.io/library/rust:latest                                                                                   0.9s
 => [stage-1 1/3] FROM docker.io/library/debian:bullseye-slim@sha256:3bc5e94a0e8329c102203c3f5f26fd67835f0c81633dd6949de0557867a87fac            0.0s
 => [builder 1/7] FROM docker.io/library/rust:latest@sha256:8a4ca3ca75afbc97bcf5362e9a694fe049d15734fbbaf82b8b7e224616c1254b                     0.0s
 => [internal] load build context                                                                                                                0.3s
 => => transferring context: 948.93kB                                                                                                            0.3s
 => CACHED [stage-1 2/3] RUN DEBIAN_FRONTEND=noninteractive apt-get update -yq &&   DEBIAN_FRONTEND=noninteractive apt-get install -yq     libs  0.0s
 => CACHED [builder 2/7] WORKDIR /lightning                                                                                                      0.0s
 => CACHED [builder 3/7] RUN apt-get update                                                                                                      0.0s
 => CACHED [builder 4/7] RUN apt-get install -y     build-essential     cmake     clang     pkg-config     libssl-dev     gcc     protobuf-comp  0.0s
 => CACHED [builder 5/7] RUN --mount=type=cache,target=/usr/local/cargo/registry     cargo install cargo-strip                                   0.0s
 => CACHED [builder 6/7] COPY . .                                                                                                                0.0s
 => CACHED [builder 7/7] RUN --mount=type=cache,target=/usr/local/cargo/registry     --mount=type=cache,target=/lightning/target     cargo buil  0.0s
 => CACHED [stage-1 3/3] COPY --from=builder /lightning/target/release/lightning-node /usr/local/bin/lgtn                                        0.0s
 => exporting to image                                                                                                                           0.0s
 => => exporting layers                                                                                                                          0.0s
 => => writing image sha256:e8e5ed19f59c3cc6a9add5bdb578c464904e9789d5f386cc4af81044c062d998                                                     0.0s
 => => naming to docker.io/library/lightning
 ```

:::tip
The Docker image is only required to be built once and/or, when changes are pulled from the remote repository, or specific versions you might be interested in. Otherwise, you're not required to build it everytime to run the node. If you'd like to learn how to update the Lightning CLI, find our references [here](/references/Lightning%20CLI/update-cli-from-source-code).
:::

:::caution
If you don't update your source code and binary build often, you won't have the latest changes, which should happen frequently to take advandate of all the ongoing development. This is quite important to understand, as it causes confusion to some users. The Lightning application at time of writing does not update automatically.
:::

## Docker Container

A container is what's originated from the image we discussed in the section [build the docker image](#build-the-docker-image), it is a runnable instance of an image. We can create, start, stop, move, or delete a container using the Docker API or CLI.

Following up, we'll learn how to run the Docker container that includes our Lightning CLI program, built from our Dockerfile.

Once the [Docker image](#build-the-docker-image) is ready, run the container based on the image `lightning`. Effectively running the Fleek Network Lighthing node process:

```sh
sudo docker run \
  -p 4069:4069 \
  -p 4200:4200 \
  -p 6969:6969 \
  -p 18000:18000 \
  -p 18101:18101 \
  -p 18102:18102 \
  --mount type=bind,source=$HOME/.lightning,target=/root/.lightning \
  --name lightning-cli \
  -it lightning
```

:::tip
Notice that the command arguments we pass are for the flag's `-p` port numbers, `-v` to bind mount a location in your host to a container path (useful to persist your ursa configuration files, e.g. keystore), `--name` to make it easier to identify, `-it` to make it interactive (e.g. presents output to the terminal), and the image name we [built earlier](#create-the-docker-image).
:::

The output would look as the following, showing the error message "Node key does not exist. Use CLI to generate keys":

```sh
thread 'main' panicked at 'Node key does not exist. Use CLI to generate keys.', core/node/src/testnet_sync.rs:126:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
thread 'main' panicked at 'Node key does not exist. Use CLI to generate keys.', core/node/src/testnet_sync.rs:126:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

You'll have to generate the keys before launching the service.

## Generate keys

Execute the `keys generate` command on the container `lightning-cli`:

```sh
sudo docker exec -it lightning-cli lgtn keys generate
```

We've bound the host path `~/.lightning` into the container `/root/.lightning`.

You can list the contents of the `~/.lightning`, where you should find the `config.toml` and `keystore`:

```sh
.
..
config.toml
keystore
```

You only have to run the `keys generate` once from your host.

Finaly, you can start the Fleek Network node by running the command:

```sh
sudo docker start lightning-cli
```

## Viewing logs

To view the logs of a Docker container in real time, use the following command:

```sh
sudo docker logs -f lightning-cli
```

## Conclusion

Containers are a way to have a self-contained environment that includes all necessary dependencies, libraries, software, amongst others required to run an application.

Fleek Network's Lightning is developed with [Rust](https://www.rust-lang.org/), a general-purpose programming language, that requires several dependencies and libraries to compile the project. Some of these libraries are not installed by default and require some troubleshooting for the end user. [Docker](https://www.docker.com/) provides us with containers, self-containing all the required libraries for the purpose of running Lightning, our application.

We guided you through the initial installation steps, and how to build a [Docker](https://www.docker.com/) image, which then's used to Docker run a container. Plus, provided lower-level commands, to help you understand other present or advanced use-cases, and also at higher level, offerring simple utility methods.

While we do our best to provide the clearest instructions, there's always space for improvement, therefore feel free to make any contributions by messaging us on our [Discord](https://discord.gg/fleekxyz) or by opening a [PR](https://github.com/fleek-network) in any of our repositories.

Discover more about the project by [watching/contributing on Github](https://github.com/fleek-network/lightning), following us on [Twitter](https://twitter.com/fleek_net), and joining [our community Discord](https://discord.gg/fleekxyz) for all the best updates!

<Author
    name="Helder Oliveira"
    image="https://github.com/heldrida.png"
    title="Software Developer + DX"
    url="https://github.com/heldrida"
/>