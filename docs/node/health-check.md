---
title: Health check
slug: health-check
hide_title: true
tags:
  - healthcheck
  - verification
  - node status
---

<!--
  The following import is intentional (see partial <CheckoutCommitWarning />)
-->
import Author from '@site/src/components/Author';

## Overview

In network applications, a health check is a process of verifying the overall health and availability of a service. It usually involves sending a request to a specific endpoint of a service and receiving a response indicating the status. The response includes information about the status of the service, node or any errors or warnings if not healthy. This helps developers, node operators, and system administrators identify and resolve issues that may affect the network, node performance and availability.

## Quick health check

Run a quick health check by sending a GET request to `/health` endpoint of RPC on [port](/docs/learn/ports) 4069.

```sh
curl -w "\p" localhost:4069/health
```

If successful, you should get the response `OK`, as follows:

```sh
OK
```

If you get an error, then it means that your node is not healthy.

## JSON-RPC Health check

We'll send a request to the JSON RPC `flk_ping` method. Execute the following command:

```sh
$ curl -X POST -H "Content-Type: application/json" -d '{
      "jsonrpc": "2.0",
      "method": "flk_ping",
      "params": [],
      "id": 1
    }' http://127.0.0.1:4069/rpc/v0
```

:::tip
We're using cURL, make sure that you have it installed otherwise install it in your operating system.
:::

If the request is successful, you should get the result `pong` as follows:

```sh
{
  "jsonrpc": "2.0",
  "result": "pong",
  "id": 1
}
```

Any other response, such as errors determine that the node is not healthy.

<Author
    name="Helder Oliveira"
    image="https://github.com/heldrida.png"
    title="Software Developer + DX"
    url="https://github.com/heldrida"
/>