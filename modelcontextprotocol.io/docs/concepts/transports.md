# Transports（传输）

了解MCP的通信机制

传输在模型上下文协议(MCP)中为客户端和服务器之间的通信提供基础。传输层处理消息如何发送和接收的底层机制。

## 消息格式

MCP使用JSON-RPC 2.0作为其传输格式。传输层负责将MCP协议消息转换为JSON-RPC格式以进行传输，并将接收到的JSON-RPC消息转换回MCP协议消息。

有三种类型的JSON-RPC消息：

### 请求

```typescript
{
  jsonrpc: "2.0",
  id: number | string,
  method: string,
  params?: object
}
```

### 响应

```typescript
{
  jsonrpc: "2.0",
  id: number | string,
  result?: object,
  error?: {
    code: number,
    message: string,
    data?: unknown
  }
}
```

### 通知

```typescript
{
  jsonrpc: "2.0",
  method: string,
  params?: object
}
```

## 内置传输类型

MCP包含两种标准传输实现：

### 标准输入/输出 (stdio)

stdio传输通过标准输入和输出流实现通信。这对于本地集成和命令行工具特别有用。

使用stdio的场景：

* 构建命令行工具
* 实现本地集成
* 需要简单的进程通信
* 使用shell脚本

```typescript
const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {}
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

### 服务器发送事件 (SSE)

SSE传输支持服务器到客户端的流式传输，同时使用HTTP POST请求进行客户端到服务器的通信。

使用SSE的场景：

* 只需要服务器到客户端的流式传输
* 在受限网络环境中工作
* 实现简单的更新

TypeScript (Server)
```typescript
import express from "express";

const app = express();

const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {}
});

let transport: SSEServerTransport | null = null;

app.get("/sse", (req, res) => {
  transport = new SSEServerTransport("/messages", res);
  server.connect(transport);
});

app.post("/messages", (req, res) => {
  if (transport) {
    transport.handlePostMessage(req, res);
  }
});

app.listen(3000);
```

TypeScript (Client)
```typescript
const client = new Client({
  name: "example-client",
  version: "1.0.0"
}, {
  capabilities: {}
});

const transport = new SSEClientTransport(
  new URL("http://localhost:3000/sse")
);
await client.connect(transport);
```

Python (Server)
```python
from mcp.server.sse import SseServerTransport
from starlette.applications import Starlette
from starlette.routing import Route

app = Server("example-server")
sse = SseServerTransport("/messages")

async def handle_sse(scope, receive, send):
    async with sse.connect_sse(scope, receive, send) as streams:
        await app.run(streams[0], streams[1], app.create_initialization_options())

async def handle_messages(scope, receive, send):
    await sse.handle_post_message(scope, receive, send)

starlette_app = Starlette(
    routes=[
        Route("/sse", endpoint=handle_sse),
        Route("/messages", endpoint=handle_messages, methods=["POST"]),
    ]
)
```

Python (Client)
```python
async with sse_client("http://localhost:8000/sse") as streams:
    async with ClientSession(streams[0], streams[1]) as session:
        await session.initialize()
```

## 自定义传输

MCP使自定义传输的实现变得简单。任何传输实现只需要符合Transport接口：

你可以实现自定义传输用于：

* 自定义网络协议
* 专门的通信通道
* 与现有系统集成
* 性能优化

```typescript
interface Transport {
  // 开始处理消息
  start(): Promise<void>;

  // 发送JSON-RPC消息
  send(message: JSONRPCMessage): Promise<void>;

  // 关闭连接
  close(): Promise<void>;

  // 回调
  onclose?: () => void;
  onerror?: (error: Error) => void;
  onmessage?: (message: JSONRPCMessage) => void;
}
```

## 错误处理

传输实现应该处理各种错误场景：

1. 连接错误
2. 消息解析错误
3. 协议错误
4. 网络超时
5. 资源清理

错误处理示例：

```typescript
class ExampleTransport implements Transport {
  async start() {
    try {
      // 连接逻辑
    } catch (error) {
      this.onerror?.(new Error(`Failed to connect: ${error}`));
      throw error;
    }
  }

  async send(message: JSONRPCMessage) {
    try {
      // 发送逻辑
    } catch (error) {
      this.onerror?.(new Error(`Failed to send message: ${error}`));
      throw error;
    }
  }
}
```

## 最佳实践

在实现或使用MCP传输时：

1. 正确处理连接生命周期
2. 实现适当的错误处理
3. 在连接关闭时清理资源
4. 使用适当的超时设置
5. 在发送前验证消息
6. 记录传输事件以便调试
7. 在适当的情况下实现重连逻辑
8. 处理消息队列中的背压
9. 监控连接健康状况
10. 实现适当的安全措施

## 安全考虑

在实现传输时：

### 认证和授权

* 实现适当的认证机制
* 验证客户端凭证
* 使用安全的令牌处理
* 实现授权检查

### 数据安全

* 使用TLS进行网络传输
* 加密敏感数据
* 验证消息完整性
* 实现消息大小限制
* 净化输入数据

### 网络安全

* 实现速率限制
* 使用适当的超时
* 处理拒绝服务场景
* 监控异常模式
* 实现适当的防火墙规则

## 调试传输

调试传输问题的提示：

1. 启用调试日志
2. 监控消息流
3. 检查连接状态
4. 验证消息格式
5. 测试错误场景
6. 使用网络分析工具
7. 实现健康检查
8. 监控资源使用
9. 测试边缘情况
10. 使用适当的错误跟踪 