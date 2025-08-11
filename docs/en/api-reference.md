# API Reference

## Main Functions

### `mcp_server_init()`
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

**Parameters:**
- `name`: Server name identifier
- `description`: Human-readable server description
- `broker_uri`: MQTT broker URI (mqtt:// or mqtts://)
- `client_id`: Unique client identifier
- `user`: MQTT username (optional)
- `password`: MQTT password (optional)
- `cert`: Client certificate for TLS (optional)

**Returns:**
- `mcp_server_t*`: Server instance handle on success, NULL on failure

### `mcp_server_register_tool()`
Register MCP tools with the server
```c
int mcp_server_register_tool(
    mcp_server_t *server,       // Server instance
    int n_tools,                // Number of tools
    mcp_tool_t *tools           // Tool array
);
```

**Parameters:**
- `server`: Server instance handle
- `n_tools`: Number of tools in the array
- `tools`: Array of tool definitions

**Returns:**
- `0`: Success
- `-1`: Failure

### `mcp_server_register_resources()`
Register MCP resources with the server
```c
int mcp_server_register_resources(
    mcp_server_t *server,       // Server instance
    int n_resources,            // Number of resources
    mcp_resource_t *resources,  // Resource array
    mcp_resource_read read_callback // Read callback function
);
```

**Parameters:**
- `server`: Server instance handle
- `n_resources`: Number of resources in the array
- `resources`: Array of resource definitions
- `read_callback`: Function pointer for reading resource content

**Returns:**
- `0`: Success
- `-1`: Failure

### `mcp_server_run()`
Start the MCP server and begin processing requests
```c
int mcp_server_run(mcp_server_t *server);
```

**Parameters:**
- `server`: Server instance handle

**Returns:**
- `0`: Success
- `-1`: Failure

### `mcp_server_close()`
Clean up and close the MCP server
```c
void mcp_server_close(mcp_server_t *server);
```

**Parameters:**
- `server`: Server instance handle

## Data Structures

### `mcp_tool_t`
Tool definition structure
```c
typedef struct {
    char *name;                    // Tool name
    char *description;             // Tool description
    int property_count;            // Number of parameters
    property_t *properties;        // Parameter definitions
    const char *(*call)(int n_args, property_t *args); // Execution function
} mcp_tool_t;
```

### `mcp_resource_t`
Resource definition structure
```c
typedef struct {
    char *uri;                     // Resource URI
    char *name;                    // Resource name
    char *description;             // Resource description
    char *mime_type;               // MIME type
    char *title;                   // Resource title
} mcp_resource_t;
```

### `property_t`
Property definition structure
```c
typedef struct {
    char *name;                    // Property name
    char *description;             // Property description
    property_type_e type;          // Property type
    property_value_u value;        // Property value
} property_t;
```

### Property Types
```c
typedef enum {
    PROPERTY_STRING = 0,          // String type
    PROPERTY_REAL,                // Floating point type
    PROPERTY_INTEGER,             // Integer type
    PROPERTY_BOOLEAN,             // Boolean type
} property_type_e;
```

### Property Values
```c
typedef union {
    double    real_value;         // Floating point value
    long long integer_value;      // Integer value
    char     *string_value;       // String value
    bool      boolean_value;      // Boolean value
} property_value_u;
```

## Error Handling

The SDK functions return integer values to indicate success or failure:
- `0`: Operation completed successfully
- `-1`: Operation failed

Check return values after function calls to ensure proper error handling.

## Memory Management

- The server instance and associated resources are managed internally
- Use `mcp_server_close()` to properly clean up resources
- Tool and resource arrays should remain valid during server lifetime
- String parameters are copied internally, so you can free them after function calls 