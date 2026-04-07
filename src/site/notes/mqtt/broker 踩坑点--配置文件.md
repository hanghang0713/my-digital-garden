---
{"dg-publish":true,"permalink":"/mqtt/broker/","dg-note-properties":{}}
---


## broker 监听两个 listener 的问题

### 场景描述

A 设备需要通过 Broker 发布接口给到 B 设备，B 设备本身上的应用也需要通过 Broker 发布接口给。

B 设备上的 Broker 原来只配置了一个 listener 127.0.0.1:xxxx。 

为了能够接收 A 设备上应用的发布请求，于是新增了一个 listener: 0.0.0.0

但是新增一个 listener 之后， B 设备上的应用无法连接 Broker 了

### 问题原因

B  设备上的应用连接 Broker 是通过了 `SSL` 协议，Broker 里面涉及到的相关配置有 
`certfile` `keyfile`  `require_certificate`  `cafile`   等。

当我们在原来的 listener 下面增加一个新的 listener 时,  新的配置文件变为

```conf
listener xxxx 127.0.0.1
listener xxxx 0.0.0.1

certfile "server.cert"
keyfile "server.key"
require_certificate true
cafile "ca.crt"
```

增加一个 listener 之后，下面的配置变成了对 0.0.0.1 的配置，原有的 127.0.0.1 的配置失效，导致出现了 `protocol error` .

### 解决方案

每一个 listener 都需要单独配置对应的功能，因此新的配置文件，需要配置成如下的形式：

```conf
listener xxxx 127.0.0.1
certfile "server.cert"
keyfile "server.key"
require_certificate true
cafile "ca.crt"

listener xxxx 0.0.0.1
certfile "server.cert"
keyfile "server.key"
require_certificate true
cafile "ca.crt"
```

