# Protocol Series

## SSE 与 WS 的区别能简单描述一下吗？

**简短版**
`SSE` 是基于 `HTTP` 的单向服务端推送，适合通知、进度、日志等只读订阅；`WebSocket` 为全双工双向通信，适合聊天、协作编辑、游戏同步等需要客户端主动发送的实时场景。
选型：默认单向用 `SSE`；需要双向或二进制传输、低延迟交互时选 `WebSocket`。

**结论**
- `SSE` 是基于 `HTTP` 的单向服务端推送；`WebSocket` 是全双工、长连接的双向通信，适合需要客户端→服务端实时发送的场景。

**关键差异**
- 通信方向：`SSE` 仅服务端→客户端；`WS` 双向。
- 建立方式：`SSE` 使用常规 `HTTP` 长连接 (`text/event-stream`)；`WS` 通过 `Upgrade` 切换到 `ws/wss` 协议。
- 消息能力：`SSE` 文本事件、内置自动重连和事件 `id`；`WS` 帧结构、支持二进制、心跳与重连需自管。
- 网络与兼容：`SSE` 更易穿透代理/防火墙并与 `HTTP/2` 协作；`WS` 通用性强但可能需额外代理/负载配置。
- 性能与复杂度：`SSE` 实现简单、开销低，适合大量只读订阅；`WS` 适合高频、低延迟的双向交互。

**选型建议**
- 只需服务端推送：通知、日志、进度、行情快照 → 选 `SSE`。
- 需要双向实时：聊天、协作编辑、游戏同步、RPC → 选 `WebSocket`。

**示例（简化）**

```js
// SSE：服务端（Node/Express）
app.get('/events', (req, res) => {
  res.set({
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive'
  });
  let id = 0;
  const timer = setInterval(() => {
    id++;
    res.write(`id: ${id}\n`);
    res.write('event: tick\n');
    res.write(`data: ${Date.now()}\n\n`);
  }, 1000);
  req.on('close', () => clearInterval(timer));
});

// SSE：客户端
const es = new EventSource('/events');
es.addEventListener('tick', e => console.log('tick', e.data));
```

```js
// WebSocket：服务端（ws）
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });
wss.on('connection', ws => {
  ws.on('message', msg => ws.send(`echo:${msg}`));
});

// WebSocket：客户端
const ws = new WebSocket('ws://localhost:8080');
ws.onopen = () => ws.send('hello');
ws.onmessage = e => console.log(e.data);
```

**边界与注意**
- `SSE` 在旧版 IE 不支持；`WS` 浏览器支持广泛。
- `SSE` 仅服务器推送，客户端发消息需另用 `fetch/POST` 等。
- `WS` 在部分企业代理或负载均衡下需配置心跳、超时与会话粘性。

**总结**
- 默认用 `SSE` 处理单向推送；需要双向、低延迟与二进制传输时选择 `WebSocket`，并结合网络环境与后端架构做评估。
