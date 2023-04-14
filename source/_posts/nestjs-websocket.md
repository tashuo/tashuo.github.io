---
title: nestjs websocket 实现私聊、系统推送
date: 2023-04-14 19:04:24
categories:
- typescript
- nestjs
tags:
- socket.io
- webscoket
---
## Nestjs websocket开发记录
[官方文档](https://docs.nestjs.com/websockets/gateways)介绍的比较详细了，但实际开发过程中还是遇到了不少问题，在此做一下记录.

## 背景知识
### websocket
> [https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)

### 流行的websocket库
[ws](https://github.com/websockets/ws)

[socket.io](https://github.com/socketio/socket.io)

### Nest配套
> In Nest, a gateway is simply a class annotated with @WebSocketGateway() decorator. Technically, gateways are platform-agnostic which makes them compatible with any WebSockets library once an adapter is created. There are two WS platforms supported out-of-the-box: socket.io and ws.

nest提供了`gateway`的概念，`gateway`与平台无关，底层可以适配任何websocket库，此处选用更为流行的`socket.io`做为示例，而且项目切换驱动的代码改动非常小，即便后面要改为`ws`也没有太大负担，架构设计甚是优雅.

## 开发
### 安装依赖
```bash
$ pnpm i --save @nestjs/websockets @nestjs/platform-socket.io socket.io
```
*使用`ws`库：*

```bash
$ pnpm i --save @nestjs/websockets @nestjs/platform-ws ws
```
<!-- more -->
### Gateway
`gateway`相当于websocket server，可以设置websocket监听的端口(注意此处不要与http端口冲突)、可以监听所有客户端连接的生命周期事件，业务层面所有类型的消息监听和发送也是在此类中完成，通过`@SubscribeMessage`装饰器可以声明方法监听的事件类型及处理逻辑.

`gateway`写好后在`Module`中声明为`Provider`就会生效，可以用Postman进行测试，需要注意的是，`socket.io`server必须使用`socket.io`client才能通信，所以测试时需要根据使用的库选用不同的客户端进行测试，Postman的websocket request支持原生和`socket.io`两种类型，别跟我一样选错了在那儿绞尽脑汁.

### 鉴权
和http一样，当一个websocket客户端发起连接请求和消息时服务端需要判断其合法性及身份，此处涉及到两个逻辑，**建立websocket连接**和**处理消息**.

websocket连接是由http请求`upgrade`成功后创建，对应于`gateway`的`handleConnection`方法，而nest的`guard`本身也适用于`gateway`，但不幸的是只作用于`@SubscribeMessage`装饰的方法，连接的生命周期事件不受影响，所以需要在`handleConnection`中单独做鉴权，其他`@SubscribeMessage`方法通过`guard`统一处理；

但后面又发现一个问题，`handleConnection`调用时服务端其实已经跟客户端建立连接
>WebSocket establishes the connection (either before or at the beginning of handleConnection)

在连接建立和鉴权逻辑执行完毕这段时间的所有消息不受鉴权逻辑影响，虽然鉴权逻辑可能不需要多少时间，但bug就是bug，本着严谨的态度，又找到一个更为合理的方案：`allowRequest`

[![ixXO7d.md.png](https://user-images.githubusercontent.com/11506653/82481833-ef199780-9aa3-11ea-93de-ae6e7b214072.png)](https://user-images.githubusercontent.com/11506653/82481833-ef199780-9aa3-11ea-93de-ae6e7b214072.png)

`allowRequest`也是在`upgrade`时调用，而且是成功后才会创建websocket连接，符合我们的鉴权要求，需要配合`adapter`使用，[https://docs.nestjs.com/websockets/adapter](https://docs.nestjs.com/websockets/adapter)

最终根据以上的调研，大致确定了以下的鉴权方案：

1. 自定义`Adapter`的`createIOServer`方法中通过`allowRequest`绑定的函数进行鉴权，用于确保所有建立的websocket连接都是合法的
2. `gateway`的`handleConnection`中根据jwt解析出用户信息，跟当前connection做关联，后面所有的`@SubscribeMessage`方法都可以直接使用该connection的用户信息
3. `@SubscribeMessage`所有方法使用通用的`guard`做鉴权，此处的鉴权本质上是对connection活跃性的探测，如果发现不活跃了直接断开连接让客户端重连，此处需要结合心跳逻辑，活跃性由心跳维持，假如长时间未收到心跳包，则判定该客户端连接异常或掉线.

#### 问题
该方案实行的前提是客户端在建立连接后不会更换token，假如由于客户端逻辑异常，同一条连接上出现了不同的token，整个业务数据就会有问题，因为当前方案是根据连接建立时传入的token绑定用户信息的，后续的通信不会对token进行校验，无论是否传了新的token或不传token都会作为该用户的消息进行处理.


可以在该仓库找到所有的代码：[https://github.com/tashuo/nest-websocket](https://github.com/tashuo/nest-websocket)
