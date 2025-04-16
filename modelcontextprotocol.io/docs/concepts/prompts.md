# 提示

创建可重用的提示模板和工作流

提示使服务器能够定义可重用的提示模板和工作流，客户端可以轻松地将其呈现给用户和LLM。它们提供了一种强大的方式来标准化和共享常见的LLM交互。

提示被设计为**用户控制**的，这意味着它们从服务器公开给客户端时，目的是让用户能够明确地选择它们来使用。

## 概述

MCP中的提示是预定义的模板，可以：

* 接受动态参数
* 包含来自资源的上下文
* 链接多个交互
* 指导特定工作流
* 作为UI元素呈现（如斜杠命令）

## 提示结构

每个提示的定义包含：

```typescript
{
  name: string;              // 提示的唯一标识符
  description?: string;      // 人类可读的描述
  arguments?: [              // 可选的参数列表
    {
      name: string;          // 参数标识符
      description?: string;  // 参数描述
      required?: boolean;    // 参数是否必需
    }
  ]
}
```

## 发现提示

客户端可以通过`prompts/list`端点发现可用的提示：

```typescript
// 请求
{
  method: "prompts/list"
}

// 响应
{
  prompts: [
    {
      name: "analyze-code",
      description: "分析代码以寻找潜在的改进",
      arguments: [
        {
          name: "language",
          description: "编程语言",
          required: true
        }
      ]
    }
  ]
}
```

## 使用提示

要使用提示，客户端需要发出`prompts/get`请求：

```typescript
// 请求
{
  method: "prompts/get",
  params: {
    name: "analyze-code",
    arguments: {
      language: "python"
    }
  }
}

// 响应
{
  description: "分析Python代码以寻找潜在的改进",
  messages: [
    {
      role: "user",
      content: {
        type: "text",
        text: "请分析以下Python代码以寻找潜在的改进：\n\n```python\ndef calculate_sum(numbers):\n    total = 0\n    for num in numbers:\n        total = total + num\n    return total\n\nresult = calculate_sum([1, 2, 3, 4, 5])\nprint(result)\n```"
      }
    }
  ]
}
```

## 动态提示

提示可以是动态的，并包含：

### 嵌入的资源上下文

```json
{
  "name": "analyze-project",
  "description": "分析项目日志和代码",
  "arguments": [
    {
      "name": "timeframe",
      "description": "分析日志的时间段",
      "required": true
    },
    {
      "name": "fileUri",
      "description": "要审查的代码文件的URI",
      "required": true
    }
  ]
}
```

在处理`prompts/get`请求时：

```json
{
  "messages": [
    {
      "role": "user",
      "content": {
        "type": "text",
        "text": "分析这些系统日志和代码文件是否存在任何问题："
      }
    },
    {
      "role": "user",
      "content": {
        "type": "resource",
        "resource": {
          "uri": "logs://recent?timeframe=1h",
          "text": "[2024-03-14 15:32:11] ERROR: 在network.py:127中连接超时\n[2024-03-14 15:32:15] WARN: 重试连接（尝试2/3）\n[2024-03-14 15:32:20] ERROR: 超过最大重试次数",
          "mimeType": "text/plain"
        }
      }
    },
    {
      "role": "user",
      "content": {
        "type": "resource",
        "resource": {
          "uri": "file:///path/to/code.py",
          "text": "def connect_to_service(timeout=30):\n    retries = 3\n    for attempt in range(retries):\n        try:\n            return establish_connection(timeout)\n        except TimeoutError:\n            if attempt == retries - 1:\n                raise\n            time.sleep(5)\n\ndef establish_connection(timeout):\n    # 连接实现\n    pass",
          "mimeType": "text/x-python"
        }
      }
    }
  ]
}
```

### 多步工作流

```typescript
const debugWorkflow = {
  name: "debug-error",
  async getMessages(error: string) {
    return [
      {
        role: "user",
        content: {
          type: "text",
          text: `我遇到了这个错误：${error}`
        }
      },
      {
        role: "assistant",
        content: {
          type: "text",
          text: "我来帮你分析这个错误。到目前为止你尝试过什么？"
        }
      },
      {
        role: "user",
        content: {
          type: "text",
          text: "我尝试过重启服务，但错误仍然存在。"
        }
      }
    ];
  }
};
```

## 实现示例

这是在MCP服务器中实现提示的完整示例：

* TypeScript
* Python

```typescript
import { Server } from "@modelcontextprotocol/sdk/server";
import {
  ListPromptsRequestSchema,
  GetPromptRequestSchema
} from "@modelcontextprotocol/sdk/types";

const PROMPTS = {
  "git-commit": {
    name: "git-commit",
    description: "生成Git提交消息",
    arguments: [
      {
        name: "changes",
        description: "Git差异或更改描述",
        required: true
      }
    ]
  },
  "explain-code": {
    name: "explain-code",
    description: "解释代码如何工作",
    arguments: [
      {
        name: "code",
        description: "要解释的代码",
        required: true
      },
      {
        name: "language",
        description: "编程语言",
        required: false
      }
    ]
  }
};

const server = new Server({
  name: "example-prompts-server",
  version: "1.0.0"
}, {
  capabilities: {
    prompts: {}
  }
});

// 列出可用提示
server.setRequestHandler(ListPromptsRequestSchema, async () => {
  return {
    prompts: Object.values(PROMPTS)
  };
});

// 获取特定提示
server.setRequestHandler(GetPromptRequestSchema, async (request) => {
  const prompt = PROMPTS[request.params.name];
  if (!prompt) {
    throw new Error(`未找到提示：${request.params.name}`);
  }

  if (request.params.name === "git-commit") {
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `为这些更改生成一个简洁但描述性的提交消息：\n\n${request.params.arguments?.changes}`
          }
        }
      ]
    };
  }

  if (request.params.name === "explain-code") {
    const language = request.params.arguments?.language || "未知";
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `解释这段${language}代码是如何工作的：\n\n${request.params.arguments?.code}`
          }
        }
      ]
    };
  }

  throw new Error("未找到提示实现");
});
```

```python
from mcp.server import Server
import mcp.types as types

# Define available prompts
PROMPTS = {
    "git-commit": types.Prompt(
        name="git-commit",
        description="Generate a Git commit message",
        arguments=[
            types.PromptArgument(
                name="changes",
                description="Git diff or description of changes",
                required=True
            )
        ],
    ),
    "explain-code": types.Prompt(
        name="explain-code",
        description="Explain how code works",
        arguments=[
            types.PromptArgument(
                name="code",
                description="Code to explain",
                required=True
            ),
            types.PromptArgument(
                name="language",
                description="Programming language",
                required=False
            )
        ],
    )
}

# Initialize server
app = Server("example-prompts-server")

@app.list_prompts()
async def list_prompts() -> list[types.Prompt]:
    return list(PROMPTS.values())

@app.get_prompt()
async def get_prompt(
    name: str, arguments: dict[str, str] | None = None
) -> types.GetPromptResult:
    if name not in PROMPTS:
        raise ValueError(f"Prompt not found: {name}")

    if name == "git-commit":
        changes = arguments.get("changes") if arguments else ""
        return types.GetPromptResult(
            messages=[
                types.PromptMessage(
                    role="user",
                    content=types.TextContent(
                        type="text",
                        text=f"Generate a concise but descriptive commit message "
                        f"for these changes:\n\n{changes}"
                    )
                )
            ]
        )

    if name == "explain-code":
        code = arguments.get("code") if arguments else ""
        language = arguments.get("language", "Unknown") if arguments else "Unknown"
        return types.GetPromptResult(
            messages=[
                types.PromptMessage(
                    role="user",
                    content=types.TextContent(
                        type="text",
                        text=f"Explain how this {language} code works:\n\n{code}"
                    )
                )
            ]
        )

    raise ValueError("Prompt implementation not found")
```

## 最佳实践

在实现提示时：

1. 使用清晰、描述性的提示名称
2. 为提示和参数提供详细描述
3. 验证所有必需参数
4. 优雅地处理缺失参数
5. 考虑提示模板的版本控制
6. 在适当时缓存动态内容
7. 实现错误处理
8. 记录预期的参数格式
9. 考虑提示的可组合性
10. 使用各种输入测试提示

## UI集成

提示可以在客户端UI中呈现为：

* 斜杠命令
* 快速操作
* 上下文菜单项
* 命令面板条目
* 引导式工作流
* 交互式表单

## 更新和变更

服务器可以通知客户端提示变更：

1. 服务器功能：`prompts.listChanged`
2. 通知：`notifications/prompts/list_changed`
3. 客户端重新获取提示列表

## 安全注意事项

在实现提示时：

* 验证所有参数
* 净化用户输入
* 考虑速率限制
* 实现访问控制
* 审计提示使用情况
* 适当处理敏感数据
* 验证生成的内容
* 实现超时
* 考虑提示注入风险
* 记录安全要求