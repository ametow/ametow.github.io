---
title: "xpos"
date: "2025-04-04"
description: "a tunneling service that takes your localhost to the public network"
summary: ""
tags: ["golang", "tunneling", "proxy"]
categories: ["golang"]
ShowToc: true
TocOpen: true
---

## xpos — a tunneling service.

I've been working on a lightweight alternative to tools like Ngrok, with a focus on simplicity, speed, and real-time developer experience.

Meet xpos – a tunneling service that exposes your local server to the internet in seconds:

```sh
xpos http 8080
```

- Instantly get a public HTTPS URL
- TLS termination handled automatically
- Works with HTTP, WebSocket, gRPC, and any tcp protocol.
- No config, no account needed
- 100% free

Ideal for:

- Sharing your local REST API during frontend development
- Demoing a local web app to clients
- Testing webhooks from Stripe, GitHub, etc.
- Remote database/port access via TCP tunnel
- ssh into your home computer over the internet (by exposing `xpos tcp 22`)

Built in Go with a focus on low latency and potential K8s controller integration.

Would love feedback from fellow devs and OSS contributors.

Give it a spin: <https://xpos-it.com>

Repo: <https://github.com/ametow/xpos>
