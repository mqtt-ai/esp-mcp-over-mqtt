# ESP-IDF MCP over MQTT SDK

[English](README.md) | [中文](README_CN.md)

一个用于 ESP32 平台的 Model Context Protocol (MCP) over MQTT 5.0 实现，基于 ESP-IDF 框架开发。

## 什么是 MCP？

**Model Context Protocol (MCP)** 是一个开放协议，用于 AI 助手与外部数据源和工具之间的安全、结构化通信。MCP 允许 AI 助手：

- 访问外部数据源和知识库
- 调用外部工具和服务
- 在安全边界内操作
- 与多种数据源进行标准化交互

MCP 通过定义清晰的接口和协议，使 AI 助手能够安全地扩展其能力，同时保持对数据访问的控制。

## 为什么选择 MCP over MQTT？

MQTT 是一个轻量级的消息传输协议，专为物联网和边缘计算设计。将 MCP 与 MQTT 结合带来以下优势：

### 核心特性
- **轻量级传输**: 适合带宽受限的网络环境
- **可靠的消息传递**: 支持 QoS 级别和消息持久化
- **内置服务发现**: MCP 客户端可自动发现可用的 MCP 服务器
- **负载均衡和扩展性**: 支持水平扩展 MCP 服务器实例
- **灵活的访问控制**: 通过 MQTT 主题权限实现细粒度授权

### 适用场景
- **边缘计算**: 在资源受限的设备上部署 MCP 服务器
- **IoT 应用**: 为智能设备提供 AI 能力
- **云边协同**: 云端 AI 与边缘设备的无缝集成
- **实时通信**: 支持低延迟的 AI 工具调用

## SDK 功能特性

### 核心功能
- **MCP 服务器**: 完整的 MCP 协议实现
- **MQTT 5.0 支持**: 基于 ESP-IDF MQTT 客户端
- **工具注册**: 支持动态注册和调用 MCP 工具
- **资源管理**: 提供数据资源访问能力
- **JSON-RPC**: 基于 JSON-RPC 2.0 的通信协议

### 工具系统
```c
typedef struct {
    char *name;                    // 工具名称
    char *description;             // 工具描述
    int property_count;            // 参数数量
    property_t *properties;        // 参数定义
    const char *(*call)(int n_args, property_t *args); // 执行函数
} mcp_tool_t;
```

### 资源管理
```c
typedef struct {
    char *uri;                     // 资源 URI
    char *name;                    // 资源名称
    char *description;             // 资源描述
    char *mime_type;               // MIME 类型
    char *title;                   // 资源标题
} mcp_resource_t;
```

### 安全特性
- **TLS/SSL 支持**: 支持 MQTTS 安全连接
- **用户认证**: 用户名/密码认证
- **证书验证**: 客户端证书支持
- **访问控制**: 基于 MQTT 主题的权限管理

## 安装和依赖

### 系统要求
- ESP-IDF v5.0 或更高版本
- ESP32 系列芯片
- 支持 MQTT 5.0 的 MQTT Broker

### 依赖组件
```yaml
REQUIRES:
  - mqtt      # ESP-IDF MQTT 客户端
  - json      # JSON 解析库
```

### 安装步骤
1. 将本组件添加到你的 ESP-IDF 项目中
2. 在 `CMakeLists.txt` 中添加依赖
3. 配置 MQTT Broker 连接参数

## 快速开始

### 基本使用示例

```c
#include "mcp_server.h"

// 定义 MCP 工具
mcp_tool_t my_tools[] = {
    {
        .name = "get_temperature",
        .description = "获取设备温度",
        .property_count = 0,
        .properties = NULL,
        .call = get_temperature_callback
    }
};

// 初始化 MCP 服务器
mcp_server_t *server = mcp_server_init(
    "esp32_sensor",           // 服务器名称
    "ESP32 传感器 MCP 服务器", // 描述
    "mqtt://broker.example.com", // MQTT Broker URI
    "esp32_client_001",       // 客户端 ID
    "username",               // 用户名
    "password",               // 密码
    NULL                      // 证书（可选）
);

// 注册工具
mcp_server_register_tool(server, 1, my_tools);

// 启动服务器
mcp_server_run(server);
```

### 工具回调函数示例

```c
const char* get_temperature_callback(int n_args, property_t *args) {
    // 读取传感器数据
    float temp = read_temperature_sensor();
    
    // 返回 JSON 格式结果
    static char result[64];
    snprintf(result, sizeof(result), "{\"temperature\": %.2f}", temp);
    return result;
}
```

## API 参考

### 主要函数

#### `mcp_server_init()`
初始化 MCP 服务器实例
```c
mcp_server_t *mcp_server_init(
    const char *name,           // 服务器名称
    const char *description,    // 服务器描述
    const char *broker_uri,     // MQTT Broker URI
    const char *client_id,      // 客户端 ID
    const char *user,           // 用户名
    const char *password,       // 密码
    const char *cert            // 证书
);
```

#### `mcp_server_register_tool()`
注册 MCP 工具
```c
int mcp_server_register_tool(
    mcp_server_t *server,       // 服务器实例
    int n_tools,                // 工具数量
    mcp_tool_t *tools           // 工具数组
);
```

#### `mcp_server_register_resources()`
注册 MCP 资源
```c
int mcp_server_register_resources(
    mcp_server_t *server,       // 服务器实例
    int n_resources,            // 资源数量
    mcp_resource_t *resources,  // 资源数组
    mcp_resource_read read_callback // 读取回调函数
);
```

#### `mcp_server_run()`
启动 MCP 服务器
```c
int mcp_server_run(mcp_server_t *server);
```

## 配置说明

### MQTT 主题结构
- **控制主题**: `$mcp-server/{client_id}/{server_name}`
- **存在主题**: `$mcp-server/presence/{client_id}/{server_name}`
- **能力主题**: `$mcp-server/capability/{client_id}/{server_name}`

### 连接参数
- **Keepalive**: 10 秒
- **协议版本**: MQTT 5.0
- **缓冲区大小**: 81920 字节
- **最后遗嘱**: 自动发布离线状态

## 协议规范

本 SDK 实现了 [MCP over MQTT 协议规范](https://github.com/mqtt-ai/mcp-over-mqtt)，支持：

- **服务发现**: 自动发现和注册 MCP 服务器
- **负载均衡**: 支持多实例部署
- **状态管理**: 保持 MCP 服务器状态
- **权限控制**: 基于 MQTT 主题的访问控制

## 贡献指南

欢迎提交 Issue 和 Pull Request！在贡献代码前，请确保：

1. 代码符合 ESP-IDF 编码规范
2. 添加适当的测试用例
3. 更新相关文档
4. 遵循 MCP over MQTT 协议规范

## 许可证

本项目采用 [LICENSE](LICENSE) 许可证。

## 相关链接

- [MCP 官方文档](https://modelcontextprotocol.io/)
- [MCP over MQTT 规范](https://github.com/mqtt-ai/mcp-over-mqtt)
- [ESP-IDF 文档](https://docs.espressif.com/projects/esp-idf/)
- [MQTT 协议规范](https://mqtt.org/specification/)

## 支持

如果你在使用过程中遇到问题，请：

1. 查看 [Issues](../../issues) 页面
2. 提交新的 Issue 描述问题
3. 参考 MCP over MQTT 协议规范
4. 检查 ESP-IDF 版本兼容性

---

[MCP over MQTT Python SDK](https://github.com/emqx/mcp-python-sdk)