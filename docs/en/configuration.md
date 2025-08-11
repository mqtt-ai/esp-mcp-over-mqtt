# Configuration

## MQTT Topic Structure

The SDK uses a hierarchical topic structure for MCP communication:

### Core Topics
- **Control Topic**: `$mcp-server/{client_id}/{server_name}`
  - Used for MCP protocol messages and tool invocations
  - All JSON-RPC requests and responses are sent on this topic
  
- **Presence Topic**: `$mcp-server/presence/{client_id}/{server_name}`
  - Used for server presence and status updates
  - Server publishes empty message on this topic when going offline
  
- **Capability Topic**: `$mcp-server/capability/{client_id}/{server_name}`
  - Used for server capability announcements
  - Contains server description, available tools, and resources

### Topic Examples
```
$mcp-server/esp32_client_001/esp32_sensor
$mcp-server/presence/esp32_client_001/esp32_sensor
$mcp-server/capability/esp32_client_001/esp32_sensor
```

## Connection Parameters

### MQTT Client Configuration
```c
esp_mqtt_client_config_t mqtt_cfg = {
    .broker.address.uri      = "mqtt://broker.example.com",
    .session.keepalive       = 10,           // 10 seconds
    .session.protocol_ver    = MQTT_PROTOCOL_V_5,
    .credentials.client_id   = "esp32_client_001",
    .buffer.size             = 81920,        // 80KB buffer
};
```

### Key Parameters
- **Keepalive**: 10 seconds - Heartbeat interval for connection maintenance
- **Protocol Version**: MQTT 5.0 - Required for advanced features
- **Buffer Size**: 81920 bytes - Maximum message size support
- **Last Will**: Automatically configured for presence detection

### Connection Properties
The SDK automatically adds MQTT 5.0 user properties:
```c
static esp_mqtt5_user_property_item_t connect_property_arr[] = {
    { "MCP-COMPONENT-TYPE", "mcp-server" }
};
```

## Security Configuration

### TLS/SSL Support
```c
// Enable TLS connection
mcp_server_t *server = mcp_server_init(
    "server_name",
    "description",
    "mqtts://broker.example.com:8883",  // Note: mqtts://
    "client_id",
    "username",
    "password",
    client_cert_pem                      // PEM certificate
);
```

### Authentication Methods
1. **Username/Password**: Basic MQTT authentication
2. **Client Certificate**: TLS mutual authentication
3. **Topic Permissions**: Broker-level access control

## Network Configuration

### WiFi Setup
```c
// Configure WiFi before initializing MCP server
wifi_config_t wifi_config = {
    .sta = {
        .ssid = "your_ssid",
        .password = "your_password",
    },
};

ESP_ERROR_CHECK(esp_wifi_set_config(WIFI_IF_STA, &wifi_config));
ESP_ERROR_CHECK(esp_wifi_start());
```

### Broker Connection
- **Local Network**: Use local MQTT broker IP address
- **Cloud Service**: Use cloud MQTT service (e.g., AWS IoT, Azure IoT Hub)
- **Public Broker**: Use public MQTT broker for testing

## Performance Tuning

### Buffer Sizes
- **MQTT Buffer**: 81920 bytes (configurable)
- **JSON Buffer**: Dynamic allocation based on message size
- **Tool Response**: Limited by MQTT message size

### Keepalive Settings
- **Default**: 10 seconds
- **Adjustment**: Increase for stable networks, decrease for mobile scenarios
- **Impact**: Affects connection stability and battery life

## Error Handling

### Connection Failures
- **Network Issues**: Automatic reconnection with exponential backoff
- **Authentication Errors**: Check credentials and permissions
- **Broker Unavailable**: Verify broker status and network connectivity

### MQTT Errors
- **Topic Access Denied**: Check broker permissions
- **Message Too Large**: Reduce tool response size
- **QoS Issues**: Verify broker QoS support

## Monitoring and Debugging

### Log Levels
```c
// Set ESP-IDF log level
esp_log_level_set("mcp_server", ESP_LOG_INFO);
esp_log_level_set("mqtt", ESP_LOG_DEBUG);
```

### Connection Status
- Monitor presence topic for server status
- Check MQTT client connection state
- Verify tool registration success

### Performance Metrics
- Message throughput
- Response latency
- Memory usage
- Network stability 