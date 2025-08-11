# 配置说明

## MQTT 主题结构

SDK 使用分层主题结构进行 MCP 通信：

### 核心主题
- **控制主题**: `$mcp-server/{client_id}/{server_name}`
  - 用于 MCP 协议消息和工具调用
  - 所有 JSON-RPC 请求和响应都在此主题上发送
  
- **存在主题**: `$mcp-server/presence/{client_id}/{server_name}`
  - 用于服务器存在和状态更新
  - 服务器离线时在此主题上发布空消息
  
- **能力主题**: `$mcp-server/capability/{client_id}/{server_name}`
  - 用于服务器能力公告
  - 包含服务器描述、可用工具和资源

### 主题示例
```
$mcp-server/esp32_client_001/esp32_sensor
$mcp-server/presence/esp32_client_001/esp32_sensor
$mcp-server/capability/esp32_client_001/esp32_sensor
```

## 连接参数

### MQTT 客户端配置
```c
esp_mqtt_client_config_t mqtt_cfg = {
    .broker.address.uri      = "mqtt://broker.example.com",
    .session.keepalive       = 10,           // 10 秒
    .session.protocol_ver    = MQTT_PROTOCOL_V_5,
    .credentials.client_id   = "esp32_client_001",
    .buffer.size             = 81920,        // 80KB 缓冲区
};
```

### 关键参数
- **Keepalive**: 10 秒 - 连接维护的心跳间隔
- **协议版本**: MQTT 5.0 - 高级功能必需
- **缓冲区大小**: 81920 字节 - 最大消息大小支持
- **最后遗嘱**: 自动配置用于存在检测

### 连接属性
SDK 自动添加 MQTT 5.0 用户属性：
```c
static esp_mqtt5_user_property_item_t connect_property_arr[] = {
    { "MCP-COMPONENT-TYPE", "mcp-server" }
};
```

## 安全配置

### TLS/SSL 支持
```c
// 启用 TLS 连接
mcp_server_t *server = mcp_server_init(
    "server_name",
    "description",
    "mqtts://broker.example.com:8883",  // 注意: mqtts://
    "client_id",
    "username",
    "password",
    client_cert_pem                      // PEM 证书
);
```

### 认证方法
1. **用户名/密码**: 基本 MQTT 认证
2. **客户端证书**: TLS 相互认证
3. **主题权限**: Broker 级访问控制

## 网络配置

### WiFi 设置
```c
// 在初始化 MCP 服务器之前配置 WiFi
wifi_config_t wifi_config = {
    .sta = {
        .ssid = "your_ssid",
        .password = "your_password",
    },
};

ESP_ERROR_CHECK(esp_wifi_set_config(WIFI_IF_STA, &wifi_config));
ESP_ERROR_CHECK(esp_wifi_start());
```

### Broker 连接
- **本地网络**: 使用本地 MQTT broker IP 地址
- **云服务**: 使用云 MQTT 服务（如 AWS IoT、Azure IoT Hub）
- **公共 Broker**: 使用公共 MQTT broker 进行测试

## 性能调优

### 缓冲区大小
- **MQTT 缓冲区**: 81920 字节（可配置）
- **JSON 缓冲区**: 基于消息大小的动态分配
- **工具响应**: 受 MQTT 消息大小限制

### Keepalive 设置
- **默认值**: 10 秒
- **调整**: 稳定网络增加，移动场景减少
- **影响**: 影响连接稳定性和电池寿命

## 错误处理

### 连接失败
- **网络问题**: 自动重连，指数退避
- **认证错误**: 检查凭据和权限
- **Broker 不可用**: 验证 broker 状态和网络连接

### MQTT 错误
- **主题访问被拒绝**: 检查 broker 权限
- **消息过大**: 减少工具响应大小
- **QoS 问题**: 验证 broker QoS 支持

## 监控和调试

### 日志级别
```c
// 设置 ESP-IDF 日志级别
esp_log_level_set("mcp_server", ESP_LOG_INFO);
esp_log_level_set("mqtt", ESP_LOG_DEBUG);
```

### 连接状态
- 监控存在主题了解服务器状态
- 检查 MQTT 客户端连接状态
- 验证工具注册成功

### 性能指标
- 消息吞吐量
- 响应延迟
- 内存使用
- 网络稳定性 