# ESP-IDF MCP over MQTT SDK

[English](README.md) | [中文](README_CN.md)

An implementation of Model Context Protocol (MCP) over MQTT 5.0 for ESP32 platform, developed based on the ESP-IDF framework.

## What is MCP?

**Model Context Protocol (MCP)** is an open protocol for secure, structured communication between AI assistants and external data sources and tools. MCP allows AI assistants to:

- Access external data sources and knowledge bases
- Invoke external tools and services
- Operate within secure boundaries
- Interact with multiple data sources in a standardized way

MCP defines clear interfaces and protocols, enabling AI assistants to safely extend their capabilities while maintaining control over data access.

## Why MCP over MQTT?

MQTT is a lightweight messaging protocol designed for IoT and edge computing. Combining MCP with MQTT brings the following advantages:

### Core Features
- **Lightweight Transport**: Suitable for bandwidth-constrained network environments
- **Reliable Message Delivery**: Supports QoS levels and message persistence
- **Built-in Service Discovery**: MCP clients can automatically discover available MCP servers
- **Load Balancing and Scalability**: Supports horizontal scaling of MCP server instances
- **Flexible Access Control**: Implements fine-grained authorization through MQTT topic permissions

### Use Cases
- **Edge Computing**: Deploy MCP servers on resource-constrained devices
- **IoT Applications**: Provide AI capabilities for smart devices
- **Cloud-Edge Collaboration**: Seamless integration between cloud AI and edge devices
- **Real-time Communication**: Support low-latency AI tool invocation

## SDK Features

### Core Functionality
- **MCP Server**: Complete MCP protocol implementation
- **MQTT 5.0 Support**: Based on ESP-IDF MQTT client
- **Tool Registration**: Support dynamic registration and invocation of MCP tools
- **Resource Management**: Provide data resource access capabilities
- **JSON-RPC**: Communication protocol based on JSON-RPC 2.0

### Tool System
```c
typedef struct {
    char *name;                    // Tool name
    char *description;             // Tool description
    int property_count;            // Number of parameters
    property_t *properties;        // Parameter definitions
    const char *(*call)(int n_args, property_t *args); // Execution function
} mcp_tool_t;
```

### Resource Management
```c
typedef struct {
    char *uri;                     // Resource URI
    char *name;                    // Resource name
    char *description;             // Resource description
    char *mime_type;               // MIME type
    char *title;                   // Resource title
} mcp_resource_t;
```

### Security Features
- **TLS/SSL Support**: Support for MQTTS secure connections
- **User Authentication**: Username/password authentication
- **Certificate Verification**: Client certificate support
- **Access Control**: Permission management based on MQTT topics

## Installation and Dependencies

### System Requirements
- ESP-IDF v5.0 or higher
- ESP32 series chips
- MQTT Broker supporting MQTT 5.0

### Dependencies
```yaml
REQUIRES:
  - mqtt      # ESP-IDF MQTT client
  - json      # JSON parsing library
```

### Installation Steps
1. Add this component to your ESP-IDF project
2. Add dependencies in `CMakeLists.txt`
3. Configure MQTT Broker connection parameters

## Quick Start

### Basic Usage Example

```c
#include "mcp_server.h"

// Define MCP tools
mcp_tool_t my_tools[] = {
    {
        .name = "get_temperature",
        .description = "Get device temperature",
        .property_count = 0,
        .properties = NULL,
        .call = get_temperature_callback
    }
};

// Initialize MCP server
mcp_server_t *server = mcp_server_init(
    "esp32_sensor",           // Server name
    "ESP32 Sensor MCP Server", // Description
    "mqtt://broker.example.com", // MQTT Broker URI
    "esp32_client_001",       // Client ID
    "username",               // Username
    "password",               // Password
    NULL                      // Certificate (optional)
);

// Register tools
mcp_server_register_tool(server, 1, my_tools);

// Start server
mcp_server_run(server);
```

### Tool Callback Function Example

```c
const char* get_temperature_callback(int n_args, property_t *args) {
    // Read sensor data
    float temp = read_temperature_sensor();
    
    // Return JSON formatted result
    static char result[64];
    snprintf(result, sizeof(result), "{\"temperature\": %.2f}", temp);
    return result;
}
```

## API Reference

### Main Functions

#### `mcp_server_init()`
Initialize MCP server instance
```c
mcp_server_t *mcp_server_init(
    const char *name,           // Server name
    const char *description,    // Server description
    const char *broker_uri,     // MQTT Broker URI
    const char *client_id,      // Client ID
    const char *user,           // Username
    const char *password,       // Password
    const char *cert            // Certificate
);
```

#### `mcp_server_register_tool()`
Register MCP tools
```c
int mcp_server_register_tool(
    mcp_server_t *server,       // Server instance
    int n_tools,                // Number of tools
    mcp_tool_t *tools           // Tool array
);
```

#### `mcp_server_register_resources()`
Register MCP resources
```c
int mcp_server_register_resources(
    mcp_server_t *server,       // Server instance
    int n_resources,            // Number of resources
    mcp_resource_t *resources,  // Resource array
    mcp_resource_read read_callback // Read callback function
);
```

#### `mcp_server_run()`
Start MCP server
```c
int mcp_server_run(mcp_server_t *server);
```

## Configuration

### MQTT Topic Structure
- **Control Topic**: `$mcp-server/{client_id}/{server_name}`
- **Presence Topic**: `$mcp-server/presence/{client_id}/{server_name}`
- **Capability Topic**: `$mcp-server/capability/{client_id}/{server_name}`

### Connection Parameters
- **Keepalive**: 10 seconds
- **Protocol Version**: MQTT 5.0
- **Buffer Size**: 81920 bytes
- **Last Will**: Automatically publish offline status

## Protocol Specification

This SDK implements the [MCP over MQTT protocol specification](https://github.com/mqtt-ai/mcp-over-mqtt), supporting:

- **Service Discovery**: Automatic discovery and registration of MCP servers
- **Load Balancing**: Support for multi-instance deployment
- **State Management**: Maintain MCP server state
- **Access Control**: Permission control based on MQTT topics

## Contributing

We welcome Issue submissions and Pull Requests! Before contributing code, please ensure:

1. Code follows ESP-IDF coding standards
2. Add appropriate test cases
3. Update relevant documentation
4. Follow MCP over MQTT protocol specification

## License

This project is licensed under the [LICENSE](LICENSE) license.

## Related Links

- [MCP Official Documentation](https://modelcontextprotocol.io/)
- [MCP over MQTT Specification](https://github.com/mqtt-ai/mcp-over-mqtt)
- [ESP-IDF Documentation](https://docs.espressif.com/projects/esp-idf/)
- [MQTT Protocol Specification](https://mqtt.org/specification/)

## Support

If you encounter issues during use, please:

1. Check the [Issues](../../issues) page
2. Submit a new Issue describing the problem
3. Refer to MCP over MQTT protocol specification
4. Check ESP-IDF version compatibility

---

**Note**: This is an experimental implementation. It is recommended to thoroughly test before using in production environments.

[MCP over MQTT Python SDK](https://github.com/emqx/mcp-python-sdk) 