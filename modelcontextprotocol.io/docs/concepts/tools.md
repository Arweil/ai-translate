# 工具

使LLM能够通过您的服务器执行操作

工具是模型上下文协议（Model Context Protocol，MCP）中的一个强大原语，它使服务器能够向客户端公开可执行的功能。通过工具，LLM可以与外部系统交互、执行计算并在现实世界中采取行动。

工具被设计为**模型控制**的，这意味着工具从服务器公开给客户端时，目的是让AI模型能够自动调用它们（在人工参与的循环中获得批准）。

## 概述

MCP中的工具允许服务器公开可执行的函数，这些函数可以被客户端调用并被LLM用来执行操作。工具的关键方面包括：

* **发现**：客户端可以通过`tools/list`端点列出可用工具
* **调用**：工具通过`tools/call`端点调用，服务器执行请求的操作并返回结果
* **灵活性**：工具可以从简单的计算到复杂的API交互

与资源一样，工具由唯一的名称标识，并可以包含指导其使用的描述。但是，与资源不同，工具代表可以修改状态或与外部系统交互的动态操作。

## 工具定义结构

每个工具都使用以下结构定义：

```typescript
{
  name: string;          // 工具的唯一标识符
  description?: string;  // 人类可读的描述
  inputSchema: {         // 工具参数的JSON Schema
    type: "object",
    properties: { ... }  // 工具特定的参数
  }
}
```

## 实现工具

这是在MCP服务器中实现基本工具的示例：

* TypeScript
* Python

```typescript
const server = new Server({
  name: "example-server",
  version: "1.0.0"
}, {
  capabilities: {
    tools: {}
  }
});

// 定义可用工具
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [{
      name: "calculate_sum",
      description: "将两个数字相加",
      inputSchema: {
        type: "object",
        properties: {
          a: { type: "number" },
          b: { type: "number" }
        },
        required: ["a", "b"]
      }
    }]
  };
});

// 处理工具执行
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "calculate_sum") {
    const { a, b } = request.params.arguments;
    return {
      content: [
        {
          type: "text",
          text: String(a + b)
        }
      ]
    };
  }
  throw new Error("未找到工具");
});
```

```python
app = Server("example-server")

@app.list_tools()
async def list_tools() -> list[types.Tool]:
    return [
        types.Tool(
            name="calculate_sum",
            description="Add two numbers together",
            inputSchema={
                "type": "object",
                "properties": {
                    "a": {"type": "number"},
                    "b": {"type": "number"}
                },
                "required": ["a", "b"]
            }
        )
    ]

@app.call_tool()
async def call_tool(
    name: str,
    arguments: dict
) -> list[types.TextContent | types.ImageContent | types.EmbeddedResource]:
    if name == "calculate_sum":
        a = arguments["a"]
        b = arguments["b"]
        result = a + b
        return [types.TextContent(type="text", text=str(result))]
    raise ValueError(f"Tool not found: {name}")
```

## 工具模式示例

以下是服务器可以提供的一些工具类型示例：

### 系统操作

与本地系统交互的工具：

```typescript
{
  name: "execute_command",
  description: "运行shell命令",
  inputSchema: {
    type: "object",
    properties: {
      command: { type: "string" },
      args: { type: "array", items: { type: "string" } }
    }
  }
}
```

### API集成

封装外部API的工具：

```typescript
{
  name: "github_create_issue",
  description: "创建GitHub issue",
  inputSchema: {
    type: "object",
    properties: {
      title: { type: "string" },
      body: { type: "string" },
      labels: { type: "array", items: { type: "string" } }
    }
  }
}
```

### 数据处理

转换或分析数据的工具：

```typescript
{
  name: "analyze_csv",
  description: "分析CSV文件",
  inputSchema: {
    type: "object",
    properties: {
      filepath: { type: "string" },
      operations: {
        type: "array",
        items: {
          enum: ["sum", "average", "count"]
        }
      }
    }
  }
}
```

## 最佳实践

在实现工具时：

1. 提供清晰、描述性的名称和描述
2. 使用详细的JSON Schema定义参数
3. 在工具描述中包含示例，以演示模型应如何使用它们
4. 实现适当的错误处理和验证
5. 对长时间操作使用进度报告
6. 保持工具操作的重点和原子性
7. 记录预期的返回值结构
8. 实现适当的超时
9. 考虑资源密集型操作的速率限制
10. 记录工具使用情况以进行调试和监控

## 安全注意事项

在公开工具时：

### 输入验证

* 根据schema验证所有参数
* 净化文件路径和系统命令
* 验证URL和外部标识符
* 检查参数大小和范围
* 防止命令注入

### 访问控制

* 在需要时实现身份验证
* 使用适当的授权检查
* 审计工具使用情况
* 限制请求速率
* 监控滥用情况

### 错误处理

* 不要向客户端公开内部错误
* 记录安全相关错误
* 适当处理超时
* 错误后清理资源
* 验证返回值

## 工具发现和更新

MCP支持动态工具发现：

1. 客户端可以随时列出可用工具
2. 服务器可以使用`notifications/tools/list_changed`通知客户端工具变更
3. 可以在运行时添加或删除工具
4. 可以更新工具定义（但应谨慎进行）

## 错误处理

工具错误应该在结果对象中报告，而不是作为MCP协议级别的错误。这允许LLM查看并可能处理错误。当工具遇到错误时：

1. 在结果中将`isError`设置为`true`
2. 在`content`数组中包含错误详情

以下是工具正确错误处理的示例：

* TypeScript
* Python

```typescript
try {
  // 工具操作
  const result = performOperation();
  return {
    content: [
      {
        type: "text",
        text: `操作成功：${result}`
      }
    ]
  };
} catch (error) {
  return {
    isError: true,
    content: [
      {
        type: "text",
        text: `错误：${error.message}`
      }
    ]
  };
}
```

```python
try:
    # Tool operation
    result = perform_operation()
    return types.CallToolResult(
        content=[
            types.TextContent(
                type="text",
                text=f"Operation successful: {result}"
            )
        ]
    )
except Exception as error:
    return types.CallToolResult(
        isError=True,
        content=[
            types.TextContent(
                type="text",
                text=f"Error: {str(error)}"
            )
        ]
    )
```

这种方法允许LLM看到发生了错误，并可能采取纠正措施或请求人工干预。

## 测试工具

MCP工具的全面测试策略应该涵盖：

* **功能测试**：验证工具使用有效输入正确执行，并适当处理无效输入
* **集成测试**：使用真实和模拟的依赖项测试工具与外部系统的交互
* **安全测试**：验证身份验证、授权、输入净化和速率限制
* **性能测试**：检查负载下的行为、超时处理和资源清理
* **错误处理**：确保工具通过MCP协议正确报告错误并清理资源