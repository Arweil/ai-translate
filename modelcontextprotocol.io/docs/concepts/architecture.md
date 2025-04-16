# 核心架构

了解MCP如何连接客户端、服务器和LLM

Model Context Protocol (MCP)建立在灵活、可扩展的架构之上，实现了LLM应用程序和集成之间的无缝通信。本文档涵盖了核心架构组件和概念。

## 概述

MCP遵循客户端-服务器架构，其中：

* **主机**是启动连接的LLM应用程序（如Claude Desktop或IDE）
* **客户端**在主机应用程序内部与服务器保持1:1连接
* **服务器**为客户端提供上下文、工具和提示

<svg aria-roledescription="flowchart-v2" role="graphics-document document" viewBox="0 0 580.625 284" style="max-width: 580.625px;" class="flowchart" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns="http://www.w3.org/2000/svg" width="100%" id="r3"><style>#r3{font-family:inherit;font-size:16px;fill:#333;}#r3 .error-icon{fill:#552222;}#r3 .error-text{fill:#552222;stroke:#552222;}#r3 .edge-thickness-normal{stroke-width:1px;}#r3 .edge-thickness-thick{stroke-width:3.5px;}#r3 .edge-pattern-solid{stroke-dasharray:0;}#r3 .edge-thickness-invisible{stroke-width:0;fill:none;}#r3 .edge-pattern-dashed{stroke-dasharray:3;}#r3 .edge-pattern-dotted{stroke-dasharray:2;}#r3 .marker{fill:#333333;stroke:#333333;}#r3 .marker.cross{stroke:#333333;}#r3 svg{font-family:inherit;font-size:16px;}#r3 p{margin:0;}#r3 .label{font-family:inherit;color:#333;}#r3 .cluster-label text{fill:#333;}#r3 .cluster-label span{color:#333;}#r3 .cluster-label span p{background-color:transparent;}#r3 .label text,#r3 span{fill:#333;color:#333;}#r3 .node rect,#r3 .node circle,#r3 .node ellipse,#r3 .node polygon,#r3 .node path{fill:#ECECFF;stroke:#9370DB;stroke-width:1px;}#r3 .rough-node .label text,#r3 .node .label text,#r3 .image-shape .label,#r3 .icon-shape .label{text-anchor:middle;}#r3 .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#r3 .rough-node .label,#r3 .node .label,#r3 .image-shape .label,#r3 .icon-shape .label{text-align:center;}#r3 .node.clickable{cursor:pointer;}#r3 .root .anchor path{fill:#333333!important;stroke-width:0;stroke:#333333;}#r3 .arrowheadPath{fill:#333333;}#r3 .edgePath .path{stroke:#333333;stroke-width:2.0px;}#r3 .flowchart-link{stroke:#333333;fill:none;}#r3 .edgeLabel{background-color:rgba(232,232,232, 0.8);text-align:center;}#r3 .edgeLabel p{background-color:rgba(232,232,232, 0.8);}#r3 .edgeLabel rect{opacity:0.5;background-color:rgba(232,232,232, 0.8);fill:rgba(232,232,232, 0.8);}#r3 .labelBkg{background-color:rgba(232, 232, 232, 0.5);}#r3 .cluster rect{fill:#ffffde;stroke:#aaaa33;stroke-width:1px;}#r3 .cluster text{fill:#333;}#r3 .cluster span{color:#333;}#r3 div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:inherit;font-size:12px;background:hsl(80, 100%, 96.2745098039%);border:1px solid #aaaa33;border-radius:2px;pointer-events:none;z-index:100;}#r3 .flowchartTitleText{text-anchor:middle;font-size:18px;fill:#333;}#r3 rect.text{fill:none;stroke-width:0;}#r3 .icon-shape,#r3 .image-shape{background-color:rgba(232,232,232, 0.8);text-align:center;}#r3 .icon-shape p,#r3 .image-shape p{background-color:rgba(232,232,232, 0.8);padding:2px;}#r3 .icon-shape rect,#r3 .image-shape rect{opacity:0.5;background-color:rgba(232,232,232, 0.8);fill:rgba(232,232,232, 0.8);}#r3 :root{--mermaid-font-family:inherit;}</style><g><marker orient="auto" markerHeight="8" markerWidth="8" markerUnits="userSpaceOnUse" refY="5" refX="5" viewBox="0 0 10 10" class="marker flowchart-v2" id="r3_flowchart-v2-pointEnd"><path style="stroke-width: 1; stroke-dasharray: 1, 0;" class="arrowMarkerPath" d="M 0 0 L 10 5 L 0 10 z"></path></marker><marker orient="auto" markerHeight="8" markerWidth="8" markerUnits="userSpaceOnUse" refY="5" refX="4.5" viewBox="0 0 10 10" class="marker flowchart-v2" id="r3_flowchart-v2-pointStart"><path style="stroke-width: 1; stroke-dasharray: 1, 0;" class="arrowMarkerPath" d="M 0 5 L 10 10 L 10 0 z"></path></marker><marker orient="auto" markerHeight="11" markerWidth="11" markerUnits="userSpaceOnUse" refY="5" refX="11" viewBox="0 0 10 10" class="marker flowchart-v2" id="r3_flowchart-v2-circleEnd"><circle style="stroke-width: 1; stroke-dasharray: 1, 0;" class="arrowMarkerPath" r="5" cy="5" cx="5"></circle></marker><marker orient="auto" markerHeight="11" markerWidth="11" markerUnits="userSpaceOnUse" refY="5" refX="-1" viewBox="0 0 10 10" class="marker flowchart-v2" id="r3_flowchart-v2-circleStart"><circle style="stroke-width: 1; stroke-dasharray: 1, 0;" class="arrowMarkerPath" r="5" cy="5" cx="5"></circle></marker><marker orient="auto" markerHeight="11" markerWidth="11" markerUnits="userSpaceOnUse" refY="5.2" refX="12" viewBox="0 0 11 11" class="marker cross flowchart-v2" id="r3_flowchart-v2-crossEnd"><path style="stroke-width: 2; stroke-dasharray: 1, 0;" class="arrowMarkerPath" d="M 1,1 l 9,9 M 10,1 l -9,9"></path></marker><marker orient="auto" markerHeight="11" markerWidth="11" markerUnits="userSpaceOnUse" refY="5.2" refX="-1" viewBox="0 0 11 11" class="marker cross flowchart-v2" id="r3_flowchart-v2-crossStart"><path style="stroke-width: 2; stroke-dasharray: 1, 0;" class="arrowMarkerPath" d="M 1,1 l 9,9 M 10,1 l -9,9"></path></marker><g class="root"><g class="clusters"><g data-look="classic" id="subGraph2" class="cluster"><rect height="124" width="200.7578125" y="8" x="371.8671875" style=""></rect><g transform="translate(414.58203125, 8)" class="cluster-label"><foreignObject height="24" width="115.328125"><div style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel"><p>Server Process</p></span></div></foreignObject></g></g><g data-look="classic" id="subGraph1" class="cluster"><rect height="124" width="200.7578125" y="152" x="371.8671875" style=""></rect><g transform="translate(414.58203125, 152)" class="cluster-label"><foreignObject height="24" width="115.328125"><div style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel"><p>Server Process</p></span></div></foreignObject></g></g><g data-look="classic" id="Host" class="cluster"><rect height="268" width="194.3125" y="8" x="8" style=""></rect><g transform="translate(87.57421875, 8)" class="cluster-label"><foreignObject height="24" width="35.1640625"><div style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel"><p>Host</p></span></div></foreignObject></g></g></g><g class="edgePaths"><path marker-end="url(#r3_flowchart-v2-pointEnd)" marker-start="url(#r3_flowchart-v2-pointStart)" style="" class="edge-thickness-normal edge-pattern-solid edge-thickness-normal edge-pattern-solid flowchart-link" id="L_client1_server1_0" d="M181.313,214L184.813,214C188.313,214,195.313,214,212.942,214C230.572,214,258.831,214,287.09,214C315.349,214,343.608,214,361.238,214C378.867,214,385.867,214,389.367,214L392.867,214"></path><path marker-end="url(#r3_flowchart-v2-pointEnd)" marker-start="url(#r3_flowchart-v2-pointStart)" style="" class="edge-thickness-normal edge-pattern-solid edge-thickness-normal edge-pattern-solid flowchart-link" id="L_client2_server2_1" d="M181.313,70L184.813,70C188.313,70,195.313,70,212.942,70C230.572,70,258.831,70,287.09,70C315.349,70,343.608,70,361.238,70C378.867,70,385.867,70,389.367,70L392.867,70"></path></g><g class="edgeLabels"><g transform="translate(287.08984375, 214)" class="edgeLabel"><g transform="translate(-59.77734375, -12)" class="label"><foreignObject height="24" width="119.5546875"><div style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;" class="labelBkg" xmlns="http://www.w3.org/1999/xhtml"><span class="edgeLabel"><p>Transport Layer</p></span></div></foreignObject></g></g><g transform="translate(287.08984375, 70)" class="edgeLabel"><g transform="translate(-59.77734375, -12)" class="label"><foreignObject height="24" width="119.5546875"><div style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;" class="labelBkg" xmlns="http://www.w3.org/1999/xhtml"><span class="edgeLabel"><p>Transport Layer</p></span></div></foreignObject></g></g></g><g class="nodes"><g transform="translate(105.15625, 214)" id="flowchart-client1-48" class="node default"><rect height="54" width="144.3125" y="-27" x="-72.15625" style="" class="basic label-container"></rect><g transform="translate(-42.15625, -12)" style="" class="label"><rect></rect><foreignObject height="24" width="84.3125"><div style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel"><p>MCP Client</p></span></div></foreignObject></g></g><g transform="translate(105.15625, 70)" id="flowchart-client2-49" class="node default"><rect height="54" width="144.3125" y="-27" x="-72.15625" style="" class="basic label-container"></rect><g transform="translate(-42.15625, -12)" style="" class="label"><rect></rect><foreignObject height="24" width="84.3125"><div style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel"><p>MCP Client</p></span></div></foreignObject></g></g><g transform="translate(472.24609375, 214)" id="flowchart-server1-50" class="node default"><rect height="54" width="150.7578125" y="-27" x="-75.37890625" style="" class="basic label-container"></rect><g transform="translate(-45.37890625, -12)" style="" class="label"><rect></rect><foreignObject height="24" width="90.7578125"><div style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel"><p>MCP Server</p></span></div></foreignObject></g></g><g transform="translate(472.24609375, 70)" id="flowchart-server2-51" class="node default"><rect height="54" width="150.7578125" y="-27" x="-75.37890625" style="" class="basic label-container"></rect><g transform="translate(-45.37890625, -12)" style="" class="label"><rect></rect><foreignObject height="24" width="90.7578125"><div style="display: table-cell; white-space: nowrap; line-height: 1.5; max-width: 200px; text-align: center;" xmlns="http://www.w3.org/1999/xhtml"><span class="nodeLabel"><p>MCP Server</p></span></div></foreignObject></g></g></g></g></g></svg>

## 核心组件

### 协议层

协议层处理消息帧、请求/响应链接和高级通信模式。

* TypeScript
* Python

```typescript
class Protocol<Request, Notification, Result> {
    // 处理传入请求
    setRequestHandler<T>(schema: T, handler: (request: T, extra: RequestHandlerExtra) => Promise<Result>): void

    // 处理传入通知
    setNotificationHandler<T>(schema: T, handler: (notification: T) => Promise<void>): void

    // 发送请求并等待响应
    request<T>(request: Request, schema: T, options?: RequestOptions): Promise<T>

    // 发送单向通知
    notification(notification: Notification): Promise<void>
}
```

```python
class Session(BaseSession[RequestT, NotificationT, ResultT]):
    async def send_request(
        self,
        request: RequestT,
        result_type: type[Result]
    ) -> Result:
        """
        Send request and wait for response. Raises McpError if response contains error.
        """
        # Request handling implementation

    async def send_notification(
        self,
        notification: NotificationT
    ) -> None:
        """Send one-way notification that doesn't expect response."""
        # Notification handling implementation

    async def _received_request(
        self,
        responder: RequestResponder[ReceiveRequestT, ResultT]
    ) -> None:
        """Handle incoming request from other side."""
        # Request handling implementation

    async def _received_notification(
        self,
        notification: ReceiveNotificationT
    ) -> None:
        """Handle incoming notification from other side."""
        # Notification handling implementation
```

关键类包括：

* `Protocol`
* `Client`
* `Server`

### 传输层

传输层处理客户端和服务器之间的实际通信。MCP支持多种传输机制：

1. **Stdio传输**
   * 使用标准输入/输出进行通信
   * 适用于本地进程
2. **带SSE的HTTP传输**
   * 使用Server-Sent Events进行服务器到客户端的消息传输
   * 使用HTTP POST进行客户端到服务器的消息传输

所有传输都使用JSON-RPC 2.0来交换消息。有关Model Context Protocol消息格式的详细信息，请参阅规范。

### 消息类型

MCP有以下主要消息类型：

1. **请求**期望从另一方得到响应：
```typescript
interface Request {
  method: string;
  params?: { ... };
}
```
2. **结果**是对请求的成功响应：
```typescript
interface Result {
  [key: string]: unknown;
}
```
3. **错误**表示请求失败：
```typescript
interface Error {
  code: number;
  message: string;
  data?: unknown;
}
```
4. **通知**是不期望响应的单向消息：
```typescript
interface Notification {
  method: string;
  params?: { ... };
}
```

## 连接生命周期

### 1. 初始化
<svg aria-roledescription="sequence" role="graphics-document document" viewBox="-50 -10 480 371" style="max-width: 480px;" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns="http://www.w3.org/2000/svg" width="100%" id="r4"><g><rect class="actor actor-bottom" ry="3" rx="3" name="Server" height="65" width="150" stroke="#666" fill="#eaeaea" y="285" x="230"></rect><text style="text-anchor: middle; font-size: 16px; font-weight: 400; font-family: inherit;" class="actor actor-box" alignment-baseline="central" dominant-baseline="central" y="317.5" x="305"><tspan dy="0" x="305">Server</tspan></text></g><g><rect class="actor actor-bottom" ry="3" rx="3" name="Client" height="65" width="150" stroke="#666" fill="#eaeaea" y="285" x="0"></rect><text style="text-anchor: middle; font-size: 16px; font-weight: 400; font-family: inherit;" class="actor actor-box" alignment-baseline="central" dominant-baseline="central" y="317.5" x="75"><tspan dy="0" x="75">Client</tspan></text></g><g><line name="Server" stroke="#999" stroke-width="0.5px" class="actor-line 200" y2="285" x2="305" y1="65" x1="305" id="actor1"></line><g id="root-1"><rect class="actor actor-top" ry="3" rx="3" name="Server" height="65" width="150" stroke="#666" fill="#eaeaea" y="0" x="230"></rect><text style="text-anchor: middle; font-size: 16px; font-weight: 400; font-family: inherit;" class="actor actor-box" alignment-baseline="central" dominant-baseline="central" y="32.5" x="305"><tspan dy="0" x="305">Server</tspan></text></g></g><g><line name="Client" stroke="#999" stroke-width="0.5px" class="actor-line 200" y2="285" x2="75" y1="65" x1="75" id="actor0"></line><g id="root-0"><rect class="actor actor-top" ry="3" rx="3" name="Client" height="65" width="150" stroke="#666" fill="#eaeaea" y="0" x="0"></rect><text style="text-anchor: middle; font-size: 16px; font-weight: 400; font-family: inherit;" class="actor actor-box" alignment-baseline="central" dominant-baseline="central" y="32.5" x="75"><tspan dy="0" x="75">Client</tspan></text></g></g><style>#r4{font-family:inherit;font-size:16px;fill:#333;}#r4 .error-icon{fill:#552222;}#r4 .error-text{fill:#552222;stroke:#552222;}#r4 .edge-thickness-normal{stroke-width:1px;}#r4 .edge-thickness-thick{stroke-width:3.5px;}#r4 .edge-pattern-solid{stroke-dasharray:0;}#r4 .edge-thickness-invisible{stroke-width:0;fill:none;}#r4 .edge-pattern-dashed{stroke-dasharray:3;}#r4 .edge-pattern-dotted{stroke-dasharray:2;}#r4 .marker{fill:#333333;stroke:#333333;}#r4 .marker.cross{stroke:#333333;}#r4 svg{font-family:inherit;font-size:16px;}#r4 p{margin:0;}#r4 .actor{stroke:hsl(259.6261682243, 59.7765363128%, 87.9019607843%);fill:#ECECFF;}#r4 text.actor&gt;tspan{fill:black;stroke:none;}#r4 .actor-line{stroke:hsl(259.6261682243, 59.7765363128%, 87.9019607843%);}#r4 .messageLine0{stroke-width:1.5;stroke-dasharray:none;stroke:#333;}#r4 .messageLine1{stroke-width:1.5;stroke-dasharray:2,2;stroke:#333;}#r4 #arrowhead path{fill:#333;stroke:#333;}#r4 .sequenceNumber{fill:white;}#r4 #sequencenumber{fill:#333;}#r4 #crosshead path{fill:#333;stroke:#333;}#r4 .messageText{fill:#333;stroke:none;}#r4 .labelBox{stroke:hsl(259.6261682243, 59.7765363128%, 87.9019607843%);fill:#ECECFF;}#r4 .labelText,#r4 .labelText&gt;tspan{fill:black;stroke:none;}#r4 .loopText,#r4 .loopText&gt;tspan{fill:black;stroke:none;}#r4 .loopLine{stroke-width:2px;stroke-dasharray:2,2;stroke:hsl(259.6261682243, 59.7765363128%, 87.9019607843%);fill:hsl(259.6261682243, 59.7765363128%, 87.9019607843%);}#r4 .note{stroke:#aaaa33;fill:#fff5ad;}#r4 .noteText,#r4 .noteText&gt;tspan{fill:black;stroke:none;}#r4 .activation0{fill:#f4f4f4;stroke:#666;}#r4 .activation1{fill:#f4f4f4;stroke:#666;}#r4 .activation2{fill:#f4f4f4;stroke:#666;}#r4 .actorPopupMenu{position:absolute;}#r4 .actorPopupMenuPanel{position:absolute;fill:#ECECFF;box-shadow:0px 8px 16px 0px rgba(0,0,0,0.2);filter:drop-shadow(3px 5px 2px rgb(0 0 0 / 0.4));}#r4 .actor-man line{stroke:hsl(259.6261682243, 59.7765363128%, 87.9019607843%);fill:#ECECFF;}#r4 .actor-man circle,#r4 line{stroke:hsl(259.6261682243, 59.7765363128%, 87.9019607843%);fill:#ECECFF;stroke-width:2px;}#r4 :root{--mermaid-font-family:inherit;}</style><g></g><defs><symbol height="24" width="24" id="computer"><path d="M2 2v13h20v-13h-20zm18 11h-16v-9h16v9zm-10.228 6l.466-1h3.524l.467 1h-4.457zm14.228 3h-24l2-6h2.104l-1.33 4h18.45l-1.297-4h2.073l2 6zm-5-10h-14v-7h14v7z" transform="scale(.5)"></path></symbol></defs><defs><symbol clip-rule="evenodd" fill-rule="evenodd" id="database"><path d="M12.258.001l.256.004.255.005.253.008.251.01.249.012.247.015.246.016.242.019.241.02.239.023.236.024.233.027.231.028.229.031.225.032.223.034.22.036.217.038.214.04.211.041.208.043.205.045.201.046.198.048.194.05.191.051.187.053.183.054.18.056.175.057.172.059.168.06.163.061.16.063.155.064.15.066.074.033.073.033.071.034.07.034.069.035.068.035.067.035.066.035.064.036.064.036.062.036.06.036.06.037.058.037.058.037.055.038.055.038.053.038.052.038.051.039.05.039.048.039.047.039.045.04.044.04.043.04.041.04.04.041.039.041.037.041.036.041.034.041.033.042.032.042.03.042.029.042.027.042.026.043.024.043.023.043.021.043.02.043.018.044.017.043.015.044.013.044.012.044.011.045.009.044.007.045.006.045.004.045.002.045.001.045v17l-.001.045-.002.045-.004.045-.006.045-.007.045-.009.044-.011.045-.012.044-.013.044-.015.044-.017.043-.018.044-.02.043-.021.043-.023.043-.024.043-.026.043-.027.042-.029.042-.03.042-.032.042-.033.042-.034.041-.036.041-.037.041-.039.041-.04.041-.041.04-.043.04-.044.04-.045.04-.047.039-.048.039-.05.039-.051.039-.052.038-.053.038-.055.038-.055.038-.058.037-.058.037-.06.037-.06.036-.062.036-.064.036-.064.036-.066.035-.067.035-.068.035-.069.035-.07.034-.071.034-.073.033-.074.033-.15.066-.155.064-.16.063-.163.061-.168.06-.172.059-.175.057-.18.056-.183.054-.187.053-.191.051-.194.05-.198.048-.201.046-.205.045-.208.043-.211.041-.214.04-.217.038-.22.036-.223.034-.225.032-.229.031-.231.028-.233.027-.236.024-.239.023-.241.02-.242.019-.246.016-.247.015-.249.012-.251.01-.253.008-.255.005-.256.004-.258.001-.258-.001-.256-.004-.255-.005-.253-.008-.251-.01-.249-.012-.247-.015-.245-.016-.243-.019-.241-.02-.238-.023-.236-.024-.234-.027-.231-.028-.228-.031-.226-.032-.223-.034-.22-.036-.217-.038-.214-.04-.211-.041-.208-.043-.204-.045-.201-.046-.198-.048-.195-.05-.19-.051-.187-.053-.184-.054-.179-.056-.176-.057-.172-.059-.167-.06-.164-.061-.159-.063-.155-.064-.151-.066-.074-.033-.072-.033-.072-.034-.07-.034-.069-.035-.068-.035-.067-.035-.066-.035-.064-.036-.063-.036-.062-.036-.061-.036-.06-.037-.058-.037-.057-.037-.056-.038-.055-.038-.053-.038-.052-.038-.051-.039-.049-.039-.049-.039-.046-.039-.046-.04-.044-.04-.043-.04-.041-.04-.04-.041-.039-.041-.037-.041-.036-.041-.034-.041-.033-.042-.032-.042-.03-.042-.029-.042-.027-.042-.026-.043-.024-.043-.023-.043-.021-.043-.02-.043-.018-.044-.017-.043-.015-.044-.013-.044-.012-.044-.011-.045-.009-.044-.007-.045-.006-.045-.004-.045-.002-.045-.001-.045v-17l.001-.045.002-.045.004-.045.006-.045.007-.045.009-.044.011-.045.012-.044.013-.044.015-.044.017-.043.018-.044.02-.043.021-.043.023-.043.024-.043.026-.043.027-.042.029-.042.03-.042.032-.042.033-.042.034-.041.036-.041.037-.041.039-.041.04-.041.041-.04.043-.04.044-.04.046-.04.046-.039.049-.039.049-.039.051-.039.052-.038.053-.038.055-.038.056-.038.057-.037.058-.037.06-.037.061-.036.062-.036.063-.036.064-.036.066-.035.067-.035.068-.035.069-.035.07-.034.072-.034.072-.033.074-.033.151-.066.155-.064.159-.063.164-.061.167-.06.172-.059.176-.057.179-.056.184-.054.187-.053.19-.051.195-.05.198-.048.201-.046.204-.045.208-.043.211-.041.214-.04.217-.038.22-.036.223-.034.226-.032.228-.031.231-.028.234-.027.236-.024.238-.023.241-.02.243-.019.245-.016.247-.015.249-.012.251-.01.253-.008.255-.005.256-.004.258-.001.258.001zm-9.258 20.499v.01l.001.021.003.021.004.022.005.021.006.022.007.022.009.023.01.022.011.023.012.023.013.023.015.023.016.024.017.023.018.024.019.024.021.024.022.025.023.024.024.025.052.049.056.05.061.051.066.051.07.051.075.051.079.052.084.052.088.052.092.052.097.052.102.051.105.052.11.052.114.051.119.051.123.051.127.05.131.05.135.05.139.048.144.049.147.047.152.047.155.047.16.045.163.045.167.043.171.043.176.041.178.041.183.039.187.039.19.037.194.035.197.035.202.033.204.031.209.03.212.029.216.027.219.025.222.024.226.021.23.02.233.018.236.016.24.015.243.012.246.01.249.008.253.005.256.004.259.001.26-.001.257-.004.254-.005.25-.008.247-.011.244-.012.241-.014.237-.016.233-.018.231-.021.226-.021.224-.024.22-.026.216-.027.212-.028.21-.031.205-.031.202-.034.198-.034.194-.036.191-.037.187-.039.183-.04.179-.04.175-.042.172-.043.168-.044.163-.045.16-.046.155-.046.152-.047.148-.048.143-.049.139-.049.136-.05.131-.05.126-.05.123-.051.118-.052.114-.051.11-.052.106-.052.101-.052.096-.052.092-.052.088-.053.083-.051.079-.052.074-.052.07-.051.065-.051.06-.051.056-.05.051-.05.023-.024.023-.025.021-.024.02-.024.019-.024.018-.024.017-.024.015-.023.014-.024.013-.023.012-.023.01-.023.01-.022.008-.022.006-.022.006-.022.004-.022.004-.021.001-.021.001-.021v-4.127l-.077.055-.08.053-.083.054-.085.053-.087.052-.09.052-.093.051-.095.05-.097.05-.1.049-.102.049-.105.048-.106.047-.109.047-.111.046-.114.045-.115.045-.118.044-.12.043-.122.042-.124.042-.126.041-.128.04-.13.04-.132.038-.134.038-.135.037-.138.037-.139.035-.142.035-.143.034-.144.033-.147.032-.148.031-.15.03-.151.03-.153.029-.154.027-.156.027-.158.026-.159.025-.161.024-.162.023-.163.022-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.011-.178.01-.179.008-.179.008-.181.006-.182.005-.182.004-.184.003-.184.002h-.37l-.184-.002-.184-.003-.182-.004-.182-.005-.181-.006-.179-.008-.179-.008-.178-.01-.176-.011-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.022-.162-.023-.161-.024-.159-.025-.157-.026-.156-.027-.155-.027-.153-.029-.151-.03-.15-.03-.148-.031-.146-.032-.145-.033-.143-.034-.141-.035-.14-.035-.137-.037-.136-.037-.134-.038-.132-.038-.13-.04-.128-.04-.126-.041-.124-.042-.122-.042-.12-.044-.117-.043-.116-.045-.113-.045-.112-.046-.109-.047-.106-.047-.105-.048-.102-.049-.1-.049-.097-.05-.095-.05-.093-.052-.09-.051-.087-.052-.085-.053-.083-.054-.08-.054-.077-.054v4.127zm0-5.654v.011l.001.021.003.021.004.021.005.022.006.022.007.022.009.022.01.022.011.023.012.023.013.023.015.024.016.023.017.024.018.024.019.024.021.024.022.024.023.025.024.024.052.05.056.05.061.05.066.051.07.051.075.052.079.051.084.052.088.052.092.052.097.052.102.052.105.052.11.051.114.051.119.052.123.05.127.051.131.05.135.049.139.049.144.048.147.048.152.047.155.046.16.045.163.045.167.044.171.042.176.042.178.04.183.04.187.038.19.037.194.036.197.034.202.033.204.032.209.03.212.028.216.027.219.025.222.024.226.022.23.02.233.018.236.016.24.014.243.012.246.01.249.008.253.006.256.003.259.001.26-.001.257-.003.254-.006.25-.008.247-.01.244-.012.241-.015.237-.016.233-.018.231-.02.226-.022.224-.024.22-.025.216-.027.212-.029.21-.03.205-.032.202-.033.198-.035.194-.036.191-.037.187-.039.183-.039.179-.041.175-.042.172-.043.168-.044.163-.045.16-.045.155-.047.152-.047.148-.048.143-.048.139-.05.136-.049.131-.05.126-.051.123-.051.118-.051.114-.052.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.051.07-.052.065-.051.06-.05.056-.051.051-.049.023-.025.023-.024.021-.025.02-.024.019-.024.018-.024.017-.024.015-.023.014-.023.013-.024.012-.022.01-.023.01-.023.008-.022.006-.022.006-.022.004-.021.004-.022.001-.021.001-.021v-4.139l-.077.054-.08.054-.083.054-.085.052-.087.053-.09.051-.093.051-.095.051-.097.05-.1.049-.102.049-.105.048-.106.047-.109.047-.111.046-.114.045-.115.044-.118.044-.12.044-.122.042-.124.042-.126.041-.128.04-.13.039-.132.039-.134.038-.135.037-.138.036-.139.036-.142.035-.143.033-.144.033-.147.033-.148.031-.15.03-.151.03-.153.028-.154.028-.156.027-.158.026-.159.025-.161.024-.162.023-.163.022-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.011-.178.009-.179.009-.179.007-.181.007-.182.005-.182.004-.184.003-.184.002h-.37l-.184-.002-.184-.003-.182-.004-.182-.005-.181-.007-.179-.007-.179-.009-.178-.009-.176-.011-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.022-.162-.023-.161-.024-.159-.025-.157-.026-.156-.027-.155-.028-.153-.028-.151-.03-.15-.03-.148-.031-.146-.033-.145-.033-.143-.033-.141-.035-.14-.036-.137-.036-.136-.037-.134-.038-.132-.039-.13-.039-.128-.04-.126-.041-.124-.042-.122-.043-.12-.043-.117-.044-.116-.044-.113-.046-.112-.046-.109-.046-.106-.047-.105-.048-.102-.049-.1-.049-.097-.05-.095-.051-.093-.051-.09-.051-.087-.053-.085-.052-.083-.054-.08-.054-.077-.054v4.139zm0-5.666v.011l.001.02.003.022.004.021.005.022.006.021.007.022.009.023.01.022.011.023.012.023.013.023.015.023.016.024.017.024.018.023.019.024.021.025.022.024.023.024.024.025.052.05.056.05.061.05.066.051.07.051.075.052.079.051.084.052.088.052.092.052.097.052.102.052.105.051.11.052.114.051.119.051.123.051.127.05.131.05.135.05.139.049.144.048.147.048.152.047.155.046.16.045.163.045.167.043.171.043.176.042.178.04.183.04.187.038.19.037.194.036.197.034.202.033.204.032.209.03.212.028.216.027.219.025.222.024.226.021.23.02.233.018.236.017.24.014.243.012.246.01.249.008.253.006.256.003.259.001.26-.001.257-.003.254-.006.25-.008.247-.01.244-.013.241-.014.237-.016.233-.018.231-.02.226-.022.224-.024.22-.025.216-.027.212-.029.21-.03.205-.032.202-.033.198-.035.194-.036.191-.037.187-.039.183-.039.179-.041.175-.042.172-.043.168-.044.163-.045.16-.045.155-.047.152-.047.148-.048.143-.049.139-.049.136-.049.131-.051.126-.05.123-.051.118-.052.114-.051.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.052.07-.051.065-.051.06-.051.056-.05.051-.049.023-.025.023-.025.021-.024.02-.024.019-.024.018-.024.017-.024.015-.023.014-.024.013-.023.012-.023.01-.022.01-.023.008-.022.006-.022.006-.022.004-.022.004-.021.001-.021.001-.021v-4.153l-.077.054-.08.054-.083.053-.085.053-.087.053-.09.051-.093.051-.095.051-.097.05-.1.049-.102.048-.105.048-.106.048-.109.046-.111.046-.114.046-.115.044-.118.044-.12.043-.122.043-.124.042-.126.041-.128.04-.13.039-.132.039-.134.038-.135.037-.138.036-.139.036-.142.034-.143.034-.144.033-.147.032-.148.032-.15.03-.151.03-.153.028-.154.028-.156.027-.158.026-.159.024-.161.024-.162.023-.163.023-.165.021-.166.02-.167.019-.169.018-.169.017-.171.016-.173.015-.173.014-.175.013-.175.012-.177.01-.178.01-.179.009-.179.007-.181.006-.182.006-.182.004-.184.003-.184.001-.185.001-.185-.001-.184-.001-.184-.003-.182-.004-.182-.006-.181-.006-.179-.007-.179-.009-.178-.01-.176-.01-.176-.012-.175-.013-.173-.014-.172-.015-.171-.016-.17-.017-.169-.018-.167-.019-.166-.02-.165-.021-.163-.023-.162-.023-.161-.024-.159-.024-.157-.026-.156-.027-.155-.028-.153-.028-.151-.03-.15-.03-.148-.032-.146-.032-.145-.033-.143-.034-.141-.034-.14-.036-.137-.036-.136-.037-.134-.038-.132-.039-.13-.039-.128-.041-.126-.041-.124-.041-.122-.043-.12-.043-.117-.044-.116-.044-.113-.046-.112-.046-.109-.046-.106-.048-.105-.048-.102-.048-.1-.05-.097-.049-.095-.051-.093-.051-.09-.052-.087-.052-.085-.053-.083-.053-.08-.054-.077-.054v4.153zm8.74-8.179l-.257.004-.254.005-.25.008-.247.011-.244.012-.241.014-.237.016-.233.018-.231.021-.226.022-.224.023-.22.026-.216.027-.212.028-.21.031-.205.032-.202.033-.198.034-.194.036-.191.038-.187.038-.183.04-.179.041-.175.042-.172.043-.168.043-.163.045-.16.046-.155.046-.152.048-.148.048-.143.048-.139.049-.136.05-.131.05-.126.051-.123.051-.118.051-.114.052-.11.052-.106.052-.101.052-.096.052-.092.052-.088.052-.083.052-.079.052-.074.051-.07.052-.065.051-.06.05-.056.05-.051.05-.023.025-.023.024-.021.024-.02.025-.019.024-.018.024-.017.023-.015.024-.014.023-.013.023-.012.023-.01.023-.01.022-.008.022-.006.023-.006.021-.004.022-.004.021-.001.021-.001.021.001.021.001.021.004.021.004.022.006.021.006.023.008.022.01.022.01.023.012.023.013.023.014.023.015.024.017.023.018.024.019.024.02.025.021.024.023.024.023.025.051.05.056.05.06.05.065.051.07.052.074.051.079.052.083.052.088.052.092.052.096.052.101.052.106.052.11.052.114.052.118.051.123.051.126.051.131.05.136.05.139.049.143.048.148.048.152.048.155.046.16.046.163.045.168.043.172.043.175.042.179.041.183.04.187.038.191.038.194.036.198.034.202.033.205.032.21.031.212.028.216.027.22.026.224.023.226.022.231.021.233.018.237.016.241.014.244.012.247.011.25.008.254.005.257.004.26.001.26-.001.257-.004.254-.005.25-.008.247-.011.244-.012.241-.014.237-.016.233-.018.231-.021.226-.022.224-.023.22-.026.216-.027.212-.028.21-.031.205-.032.202-.033.198-.034.194-.036.191-.038.187-.038.183-.04.179-.041.175-.042.172-.043.168-.043.163-.045.16-.046.155-.046.152-.048.148-.048.143-.048.139-.049.136-.05.131-.05.126-.051.123-.051.118-.051.114-.052.11-.052.106-.052.101-.052.096-.052.092-.052.088-.052.083-.052.079-.052.074-.051.07-.052.065-.051.06-.05.056-.05.051-.05.023-.025.023-.024.021-.024.02-.025.019-.024.018-.024.017-.023.015-.024.014-.023.013-.023.012-.023.01-.023.01-.022.008-.022.006-.023.006-.021.004-.022.004-.021.001-.021.001-.021-.001-.021-.001-.021-.004-.021-.004-.022-.006-.021-.006-.023-.008-.022-.01-.022-.01-.023-.012-.023-.013-.023-.014-.023-.015-.024-.017-.023-.018-.024-.019-.024-.02-.025-.021-.024-.023-.024-.023-.025-.051-.05-.056-.05-.06-.05-.065-.051-.07-.052-.074-.051-.079-.052-.083-.052-.088-.052-.092-.052-.096-.052-.101-.052-.106-.052-.11-.052-.114-.052-.118-.051-.123-.051-.126-.051-.131-.05-.136-.05-.139-.049-.143-.048-.148-.048-.152-.048-.155-.046-.16-.046-.163-.045-.168-.043-.172-.043-.175-.042-.179-.041-.183-.04-.187-.038-.191-.038-.194-.036-.198-.034-.202-.033-.205-.032-.21-.031-.212-.028-.216-.027-.22-.026-.224-.023-.226-.022-.231-.021-.233-.018-.237-.016-.241-.014-.244-.012-.247-.011-.25-.008-.254-.005-.257-.004-.26-.001-.26.001z" transform="scale(.5)"></path></symbol></defs><defs><symbol height="24" width="24" id="clock"><path d="M12 2c5.514 0 10 4.486 10 10s-4.486 10-10 10-10-4.486-10-10 4.486-10 10-10zm0-2c-6.627 0-12 5.373-12 12s5.373 12 12 12 12-5.373 12-12-5.373-12-12-12zm5.848 12.459c.202.038.202.333.001.372-1.907.361-6.045 1.111-6.547 1.111-.719 0-1.301-.582-1.301-1.301 0-.512.77-5.447 1.125-7.445.034-.192.312-.181.343.014l.985 6.238 5.394 1.011z" transform="scale(.5)"></path></symbol></defs><defs><marker orient="auto-start-reverse" markerHeight="12" markerWidth="12" markerUnits="userSpaceOnUse" refY="5" refX="7.9" id="arrowhead"><path d="M -1 0 L 10 5 L 0 10 z"></path></marker></defs><defs><marker refY="4.5" refX="4" orient="auto" markerHeight="8" markerWidth="15" id="crosshead"><path style="stroke-dasharray: 0, 0;" d="M 1,2 L 6,7 M 6,2 L 1,7" stroke-width="1pt" stroke="#000000" fill="none"></path></marker></defs><defs><marker orient="auto" markerHeight="28" markerWidth="20" refY="7" refX="15.5" id="filled-head"><path d="M 18,7 L9,13 L14,7 L9,1 Z"></path></marker></defs><defs><marker orient="auto" markerHeight="40" markerWidth="60" refY="15" refX="15" id="sequencenumber"><circle r="6" cy="15" cx="15"></circle></marker></defs><g><rect class="note" height="40" width="280" stroke="#666" fill="#EDF2AE" y="225" x="50"></rect><text style="font-family: inherit; font-size: 16px; font-weight: 400;" dy="1em" class="noteText" alignment-baseline="middle" dominant-baseline="middle" text-anchor="middle" y="230" x="190"><tspan x="190">Connection ready for use</tspan></text></g><text style="font-family: inherit; font-size: 16px; font-weight: 400;" dy="1em" class="messageText" alignment-baseline="middle" dominant-baseline="middle" text-anchor="middle" y="80" x="189">initialize request</text><line style="fill: none;" marker-end="url(#arrowhead)" stroke="none" stroke-width="2" class="messageLine0" y2="115" x2="301" y1="115" x1="76"></line><text style="font-family: inherit; font-size: 16px; font-weight: 400;" dy="1em" class="messageText" alignment-baseline="middle" dominant-baseline="middle" text-anchor="middle" y="130" x="192">initialize response</text><line style="fill: none;" marker-end="url(#arrowhead)" stroke="none" stroke-width="2" class="messageLine0" y2="165" x2="79" y1="165" x1="304"></line><text style="font-family: inherit; font-size: 16px; font-weight: 400;" dy="1em" class="messageText" alignment-baseline="middle" dominant-baseline="middle" text-anchor="middle" y="180" x="189">initialized notification</text><line style="fill: none;" marker-end="url(#arrowhead)" stroke="none" stroke-width="2" class="messageLine0" y2="215" x2="301" y1="215" x1="76"></line></svg>

1. 客户端发送带有协议版本和功能的`initialize`请求
2. 服务器响应其协议版本和功能
3. 客户端发送`initialized`通知作为确认
4. 开始正常的消息交换

### 2. 消息交换

初始化后，支持以下模式：

* **请求-响应**：客户端或服务器发送请求，另一方响应
* **通知**：任何一方发送单向消息

### 3. 终止

任何一方都可以终止连接：

* 通过`close()`进行清理关闭
* 传输断开
* 错误条件

## 错误处理

MCP定义了这些标准错误代码：

```typescript
enum ErrorCode {
  // 标准JSON-RPC错误代码
  ParseError = -32700,
  InvalidRequest = -32600,
  MethodNotFound = -32601,
  InvalidParams = -32602,
  InternalError = -32603
}
```

SDK和应用程序可以在-32000以上定义自己的错误代码。

错误通过以下方式传播：

* 对请求的错误响应
* 传输上的错误事件
* 协议级错误处理程序

## 实现示例

以下是实现MCP服务器的基本示例：

* TypeScript
* Python

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {
    resources: {}
  }
});

// 处理请求
server.setRequestHandler(ListResourcesRequestSchema, async () => {
  return {
    resources: [
      {
        uri: "example://resource",
        name: "Example Resource"
      }
    ]
  };
});

// 连接传输
const transport = new StdioServerTransport();
await server.connect(transport);
```

```python
import asyncio
import mcp.types as types
from mcp.server import Server
from mcp.server.stdio import stdio_server

app = Server("example-server")

@app.list_resources()
async def list_resources() -> list[types.Resource]:
    return [
        types.Resource(
            uri="example://resource",
            name="Example Resource"
        )
    ]

async def main():
    async with stdio_server() as streams:
        await app.run(
            streams[0],
            streams[1],
            app.create_initialization_options()
        )

if __name__ == "__main__":
    asyncio.run(main)
```

## 最佳实践

### 传输选择

1. **本地通信**
   * 对本地进程使用stdio传输
   * 适用于同一机器通信
   * 简单的进程管理
2. **远程通信**
   * 在需要HTTP兼容性的场景中使用SSE
   * 考虑包括身份验证和授权在内的安全影响

### 消息处理

1. **请求处理**
   * 彻底验证输入
   * 使用类型安全的模式
   * 优雅地处理错误
   * 实现超时
2. **进度报告**
   * 对长时间操作使用进度令牌
   * 逐步报告进度
   * 在已知时包含总进度
3. **错误管理**
   * 使用适当的错误代码
   * 包含有用的错误消息
   * 在错误时清理资源

## 安全考虑

1. **传输安全**
   * 对远程连接使用TLS
   * 验证连接来源
   * 在需要时实现身份验证
2. **消息验证**
   * 验证所有传入消息
   * 净化输入
   * 检查消息大小限制
   * 验证JSON-RPC格式
3. **资源保护**
   * 实现访问控制
   * 验证资源路径
   * 监控资源使用
   * 限制请求速率
4. **错误处理**
   * 不要泄露敏感信息
   * 记录安全相关错误
   * 实现适当的清理
   * 处理DoS场景

## 调试和监控

1. **日志记录**
   * 记录协议事件
   * 跟踪消息流
   * 监控性能
   * 记录错误
2. **诊断**
   * 实现健康检查
   * 监控连接状态
   * 跟踪资源使用
   * 分析性能
3. **测试**
   * 测试不同的传输
   * 验证错误处理
   * 检查边缘情况
   * 负载测试服务器