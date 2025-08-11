# Overview

MCP over MQTT is a project that allows you to control a device using the MCP protocol over MQTT 5.0. It is designed to be simple and easy to use, providing a way to interact with devices that support the MCP protocol.

# Why MQTT?

MQTT is a lightweight and widely used protocol for IoT and edge computing. It is designed for unreliable networks and low bandwidth, making it a good choice for edge devices and cloud services to communicate with each other.

Introducing MQTT as a transport layer for MCP enables the protocol to be used in a wider range of applications, including edge computing, IoT, and cloud services — anywhere MQTT is to be used.

# Features

The MQTT transport protocol supports all the features of MCP and adds the following features:

Built-in Service Registry and Discovery: MCP clients can discover available MCP servers from the MQTT broker.

Built-in Load Balancing and Scalability: MCP servers can be scaled horizontally by adding more MCP server instances, while keeping the the MCP server side stateful.

Additionally, by setting access control permissions on the MQTT topics used by MCP clients and servers in the MQTT broker, authorization for clients and servers can be very flexibly implemented.

# Python Client 

[MCP over MQTT Python SDK](https://github.com/emqx/mcp-python-sdk)