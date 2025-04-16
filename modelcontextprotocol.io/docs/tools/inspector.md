# Inspector（检查器）

深入指南：使用MCP检查器测试和调试Model Context Protocol服务器

MCP检查器是一个用于测试和调试MCP服务器的交互式开发者工具。虽然调试指南将检查器作为整体调试工具包的一部分进行介绍，但本文档将详细探讨检查器的特性和功能。

## 入门指南

### 安装和基本使用

检查器可以直接通过`npx`运行，无需安装：

```bash
npx @modelcontextprotocol/inspector <command>
```

```bash
npx @modelcontextprotocol/inspector <command> <arg1> <arg2>
```

#### 检查来自NPM或PyPi的服务器

从NPM或PyPi启动服务器包的常见方式：

* NPM包
* PyPi包

```bash
npx -y @modelcontextprotocol/inspector npx <package-name> <args>
# 示例
npx -y @modelcontextprotocol/inspector npx server-postgres postgres://127.0.0.1/testdb
```

```bash
npx @modelcontextprotocol/inspector uvx <package-name> <args>
# For example
npx @modelcontextprotocol/inspector uvx mcp-server-git --repository ~/code/mcp/servers.git
```

#### 检查本地开发的服务器

要检查本地开发或作为仓库下载的服务器，最常见的方式是：

* TypeScript
* Python

```bash
npx @modelcontextprotocol/inspector node path/to/server/index.js args...
```

```bash
npx @modelcontextprotocol/inspector \
  uv \
  --directory path/to/server \
  run \
  package-name \
  args...
```

请仔细阅读任何附带的README以获取最准确的说明。

## 功能概述

![MCP检查器界面](https:/mintlify.s3.us-west-1.amazonaws.com/mcp/images/mcp-inspector.png)

检查器提供了几个用于与MCP服务器交互的功能：

### 服务器连接面板

* 允许选择连接服务器的传输方式
* 对于本地服务器，支持自定义命令行参数和环境

### 资源标签页

* 列出所有可用资源
* 显示资源元数据（MIME类型、描述）
* 允许资源内容检查
* 支持订阅测试

### 提示标签页

* 显示可用的提示模板
* 显示提示参数和描述
* 启用带有自定义参数的提示测试
* 预览生成的消息

### 工具标签页

* 列出可用工具
* 显示工具模式和描述
* 启用带有自定义输入的工具测试
* 显示工具执行结果

### 通知面板

* 展示从服务器记录的所有日志
* 显示从服务器接收的通知

## 最佳实践

### 开发工作流程

1. 开始开发
   * 启动带有服务器的检查器
   * 验证基本连接
   * 检查功能协商
2. 迭代测试
   * 进行服务器更改
   * 重新构建服务器
   * 重新连接检查器
   * 测试受影响的功能
   * 监控消息
3. 测试边缘情况
   * 无效输入
   * 缺少提示参数
   * 并发操作
   * 验证错误处理和错误响应

## 下一步

* [检查器仓库](https://github.com/modelcontext/inspector) - 查看MCP检查器源代码
* [调试指南](../debugging) - 了解更广泛的调试策略 