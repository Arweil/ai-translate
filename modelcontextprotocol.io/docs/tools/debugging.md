# 调试

本指南全面介绍了如何调试Model Context Protocol (MCP)集成

在开发MCP服务器或将其与应用程序集成时，有效的调试是至关重要的。本指南涵盖了MCP生态系统中可用的调试工具和方法。

本指南适用于macOS。其他平台的指南即将推出。

## 调试工具概述

MCP提供了几种不同层级的调试工具：

1. **MCP检查器**
   * 交互式调试界面
   * 直接服务器测试
   * 详见检查器指南
2. **Claude桌面开发者工具**
   * 集成测试
   * 日志收集
   * Chrome开发者工具集成
3. **服务器日志**
   * 自定义日志实现
   * 错误跟踪
   * 性能监控

## 在Claude Desktop中调试

### 检查服务器状态

Claude.app界面提供基本的服务器状态信息：

1. 点击图标查看：
   * 已连接的服务器
   * 可用的提示和资源
2. 点击图标查看：
   * 向模型提供的工具

### 查看日志

从Claude Desktop查看详细的MCP日志：

```bash
# 实时跟踪日志
tail -n 20 -F ~/Library/Logs/Claude/mcp*.log
```

日志捕获：
* 服务器连接事件
* 配置问题
* 运行时错误
* 消息交换

### 使用Chrome开发者工具

在Claude Desktop中访问Chrome的开发者工具来调查客户端错误：

1. 创建一个`developer_settings.json`文件，将`allowDevTools`设置为true：

```bash
echo '{"allowDevTools": true}' > ~/Library/Application\ Support/Claude/developer_settings.json
```

2. 打开开发者工具：`Command-Option-Shift-i`

注意：你会看到两个开发者工具窗口：
* 主内容窗口
* 应用标题栏窗口

使用Console面板检查客户端错误。

使用Network面板检查：
* 消息负载
* 连接时序

## 常见问题

### 工作目录

当通过Claude Desktop使用MCP服务器时：

* 通过`claude_desktop_config.json`启动的服务器的工作目录可能未定义（如macOS上的`/`），因为Claude Desktop可能从任何位置启动
* 在配置和`.env`文件中始终使用绝对路径以确保可靠运行
* 对于直接通过命令行测试服务器，工作目录将是运行命令的位置

例如在`claude_desktop_config.json`中，使用：

```json
{
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/username/data"]
}
```

而不是使用相对路径如`./data`

### 环境变量

MCP服务器只自动继承一部分环境变量，如`USER`、`HOME`和`PATH`。

要覆盖默认变量或提供自己的变量，可以在`claude_desktop_config.json`中指定`env`键：

```json
{
  "myserver": {
    "command": "mcp-server-myapp",
    "env": {
      "MYAPP_API_KEY": "some_key",
    }
  }
}
```

### 服务器初始化

常见的初始化问题：

1. **路径问题**
   * 服务器可执行文件路径不正确
   * 缺少必需文件
   * 权限问题
   * 尝试为`command`使用绝对路径
2. **配置错误**
   * JSON语法无效
   * 缺少必需字段
   * 类型不匹配
3. **环境问题**
   * 缺少环境变量
   * 变量值不正确
   * 权限限制

### 连接问题

当服务器连接失败时：

1. 检查Claude Desktop日志
2. 验证服务器进程是否运行
3. 使用检查器单独测试
4. 验证协议兼容性

## 实现日志记录

### 服务器端日志记录

在构建使用本地stdio传输的服务器时，所有记录到stderr（标准错误）的消息都会被主机应用程序（如Claude Desktop）自动捕获。

本地MCP服务器不应该将消息记录到stdout（标准输出），因为这会干扰协议操作。

对于所有传输，你也可以通过发送日志消息通知来向客户端提供日志：

* Python
* TypeScript

```python
server.request_context.session.send_log_message(
  level="info",
  data="Server started successfully",
)
```

```typescript
server.sendLoggingMessage({
  level: "info",
  data: "Server started successfully",
});
```

需要记录的重要事件：
* 初始化步骤
* 资源访问
* 工具执行
* 错误情况
* 性能指标

### 客户端日志记录

在客户端应用程序中：

1. 启用调试日志
2. 监控网络流量
3. 跟踪消息交换
4. 记录错误状态

## 调试工作流程

### 开发周期

1. 初始开发
   * 使用检查器进行基本测试
   * 实现核心功能
   * 添加日志点
2. 集成测试
   * 在Claude Desktop中测试
   * 监控日志
   * 检查错误处理

### 测试更改

要高效测试更改：

* **配置更改**：重启Claude Desktop
* **服务器代码更改**：使用Command-R重新加载
* **快速迭代**：在开发期间使用检查器

## 最佳实践

### 日志策略

1. **结构化日志**
   * 使用一致的格式
   * 包含上下文
   * 添加时间戳
   * 跟踪请求ID
2. **错误处理**
   * 记录堆栈跟踪
   * 包含错误上下文
   * 跟踪错误模式
   * 监控恢复
3. **性能跟踪**
   * 记录操作时间
   * 监控资源使用
   * 跟踪消息大小
   * 测量延迟

### 安全注意事项

调试时：

1. **敏感数据**
   * 清理日志
   * 保护凭证
   * 屏蔽个人信息
2. **访问控制**
   * 验证权限
   * 检查认证
   * 监控访问模式

## 获取帮助

遇到问题时：

1. **第一步**
   * 检查服务器日志
   * 使用检查器测试
   * 审查配置
   * 验证环境
2. **支持渠道**
   * GitHub issues
   * GitHub discussions
3. **提供信息**
   * 日志摘录
   * 配置文件
   * 重现步骤
   * 环境详情

## 下一步

[MCP检查器](../inspector.md) - 学习使用MCP检查器 