# Nanobot Glossary

## A

**Agent**
An AI personality configured with specific instructions, an LLM model, and connected MCP servers. Agents are the main entities users interact with in Nanobot.

**Anthropic**
An AI company that develops Claude, a family of large language models. Nanobot supports Anthropic's API for agent intelligence.

**API (Application Programming Interface)**
A set of rules and protocols for building software applications. Nanobot provides HTTP APIs and implements the MCP API.

**API Key**
A secret token used to authenticate requests to LLM providers (OpenAI, Anthropic) or MCP servers.

## B

**Base URL**
The root URL for API requests. Can be customized to use different endpoints (e.g., custom OpenAI-compatible servers).

**Bearer Token**
An authentication token passed in HTTP Authorization headers. Used in OAuth flows and API authentication.

## C

**Chat**
A conversational interface between a user and an agent. Nanobot provides a web-based chat UI.

**Client**
In the MCP protocol, the client is the component that connects to MCP servers. Nanobot acts as an MCP client to connect to various MCP servers.

**CLI (Command-Line Interface)**
A text-based interface for interacting with Nanobot. Provides commands like `nanobot run`, `nanobot call`, etc.

**Context**
Additional information available during agent execution, such as session data, environment variables, and previous conversation history.

**CORS (Cross-Origin Resource Sharing)**
A security mechanism that allows web applications from one domain to access resources from another domain. Nanobot enables CORS for API access.

## D

**DSN (Data Source Name)**
A string that specifies how to connect to a database. Nanobot uses SQLite by default with a file-based DSN.

**Dynamic Instructions**
Agent instructions that can include variables and expressions, allowing instructions to adapt based on context.

## E

**Elicitation**
An MCP-UI feature that allows MCP servers to request input from users through forms and dialogs.

**Entrypoint**
The primary agent(s) exposed when publishing a Nanobot configuration. Determines which agents are accessible.

**Environment Variable**
A variable set in the operating system environment, used to configure Nanobot without hardcoding values in configuration files.

**Event**
A message sent from the server to clients, typically via Server-Sent Events (SSE). Events notify clients of changes like new messages or errors.

**Extends**
A configuration feature that allows one YAML file to inherit from another, enabling configuration reuse and composition.

**Expression**
A string with embedded variables that are evaluated at runtime. Uses syntax like `${input.field}` or `${env.VAR}`.

## F

**Flow**
A multi-step workflow that chains multiple agent calls or tool executions together. Flows enable complex, automated processes.

**Flusher**
An HTTP interface that allows pushing data to clients immediately rather than buffering. Used in SSE streaming.

## G

**Goroutine**
A lightweight thread in Go. Nanobot uses goroutines for concurrent operations like handling multiple MCP connections.

**GORM**
A Go ORM (Object-Relational Mapping) library used by Nanobot for database operations.

**GPT (Generative Pre-trained Transformer)**
A type of large language model developed by OpenAI. Nanobot supports various GPT models (GPT-4, GPT-3.5, etc.).

## H

**Handler**
A function that processes specific types of requests or messages. Nanobot has handlers for HTTP requests and MCP messages.

**Header**
Metadata sent with HTTP requests or responses. Used for authentication, content type, and other request properties.

**Host**
In MCP terminology, the host is the service that connects MCP servers with LLMs and users. Nanobot is an MCP Host.

**HTTP (Hypertext Transfer Protocol)**
The protocol used for web communication. Nanobot uses HTTP for its API and to connect to HTTP-based MCP servers.

## I

**Initialization**
The setup phase when an MCP client connects to a server. Involves exchanging capabilities and protocol versions.

**Instructions**
System prompts that define an agent's behavior, personality, and capabilities. Instructions guide how the LLM responds.

**Integration**
Connecting Nanobot with external services, tools, or applications through MCP servers or APIs.

## J

**JSON (JavaScript Object Notation)**
A lightweight data format used for configuration files, API requests, and MCP protocol messages.

**JSON-RPC**
A remote procedure call protocol encoded in JSON. MCP uses JSON-RPC 2.0 for message formatting.

## L

**LLM (Large Language Model)**
An AI model trained on vast amounts of text data to understand and generate human-like text. Examples: GPT-4, Claude.

**Localhost**
The local computer's network address (127.0.0.1). Nanobot runs on localhost by default.

## M

**MCP (Model Context Protocol)**
An open protocol for connecting AI assistants to tools, data sources, and external systems. Nanobot implements MCP.

**MCP Server**
A service that provides tools, resources, or prompts via the MCP protocol. Can be local (stdio) or remote (HTTP).

**MCP-UI**
Extensions to the MCP protocol that add user interface capabilities like elicitation (forms) and file uploads.

**Message**
In MCP, a JSON-RPC message containing a method call, response, or notification. Also refers to chat messages between users and agents.

**Middleware**
Software that sits between different components, processing requests or responses. Used for CORS, authentication, etc.

**MIME Type**
A string that identifies the type of data (e.g., "text/plain", "application/json", "image/png").

**Model**
The specific LLM to use for an agent (e.g., "gpt-4.1", "claude-3-opus-20240229").

## N

**Nanobot**
The MCP Host application itself. A standalone server for building and running AI agents with MCP integration.

**Notification**
An MCP message that doesn't expect a response. Used for events like resource updates.

## O

**OAuth (Open Authorization)**
An authorization framework that allows applications to obtain limited access to user accounts. Used for securing MCP server connections.

**OpenAI**
An AI research company that develops GPT models. Nanobot integrates with OpenAI's API.

**Output**
The result from an agent execution, tool call, or flow step. Can be text, structured data, or resources.

## P

**Parallel Execution**
Running multiple operations simultaneously rather than sequentially. Flows can execute steps in parallel.

**Payload**
The data portion of a request or message, typically in JSON format.

**Persistence**
Storing data permanently, such as saving conversation history to a database.

**Profile**
A named configuration variant that overrides base settings. Used for environment-specific configurations (dev, production).

**Prompt**
1. Input text given to an LLM
2. In MCP, a reusable template for generating prompts

**Protocol**
A set of rules for communication between systems. MCP is a protocol for AI-tool communication.

**Publish**
Configuration that determines which agents, servers, and flows are exposed externally.

## R

**Resource**
In MCP, a data source that can be read by agents. Resources have URIs and MIME types (e.g., files, database records, API responses).

**REST (Representational State Transfer)**
An architectural style for APIs. Nanobot's HTTP API follows REST principles.

**Root**
A base URI or directory that an MCP client can access. Roots define the scope of file system access.

**Route**
A mapping between a URL pattern and a handler function in the HTTP API.

**Runtime**
The execution environment that manages agent operations, MCP connections, and LLM calls.

## S

**Sampling**
An MCP feature that allows servers to delegate LLM calls back to the host. Experimental feature in Nanobot.

**Server**
1. MCP Server: Provides tools/resources
2. HTTP Server: Nanobot's web API
3. Server (general): Any network service

**Session**
A persistent conversation context that includes history, configuration, and state. Sessions are stored in the database.

**SSE (Server-Sent Events)**
A web standard for servers to push real-time updates to clients. Nanobot uses SSE for streaming responses.

**State**
Data that persists across requests, such as conversation history, session attributes, or configuration.

**Stdio (Standard Input/Output)**
A transport mechanism for MCP where servers communicate via standard input and output streams. Used for local processes.

**Streaming**
Sending data incrementally as it's generated rather than waiting for the complete response. Improves perceived responsiveness.

**Subscription**
An MCP feature where clients can subscribe to resource changes and receive notifications when resources update.

**Svelte**
A modern JavaScript framework used for building Nanobot's web UI.

## T

**Template**
A text pattern with placeholders that are filled in with actual values. Used in prompts and dynamic instructions.

**Thread**
Another term for a conversation or chat session. Thread IDs identify specific conversations.

**Token**
1. Authentication token for API access
2. Unit of text in LLM processing (words or subwords)

**Tool**
A function that an MCP server provides for agents to execute. Tools have names, descriptions, and input schemas.

**Tool Call**
When an LLM decides to invoke a tool rather than generating text. Includes the tool name and arguments.

**Transport**
The communication mechanism between MCP clients and servers (HTTP or stdio).

**Type**
The data type of a value (string, number, object, array, etc.). Used in schemas and validation.

## U

**URI (Uniform Resource Identifier)**
A string that identifies a resource. Used in MCP for resource references (e.g., "file:///path/to/file").

**UUID (Universally Unique Identifier)**
A 128-bit identifier that's unique across space and time. Used for session IDs in Nanobot.

## V

**Validation**
Checking that configuration, input, or data meets required rules and formats before processing.

**Variable**
A named value that can be referenced in expressions, templates, or configurations.

## W

**WebSocket**
A protocol for bidirectional communication between clients and servers. Not currently used in Nanobot but planned for future versions.

**Wire**
The underlying transport mechanism in MCP. Can be HTTP-based or stdio-based.

**Workflow**
A sequence of operations that accomplish a task. Flows define workflows in Nanobot.

## X

**XDG**
Cross-Desktop Group standards for Linux/Unix file locations. Nanobot respects XDG directories for configuration.

## Y

**YAML (YAML Ain't Markup Language)**
A human-readable data serialization format. Nanobot uses YAML for configuration files.

## Acronyms Quick Reference

- **AI**: Artificial Intelligence
- **API**: Application Programming Interface
- **CLI**: Command-Line Interface
- **CORS**: Cross-Origin Resource Sharing
- **DSN**: Data Source Name
- **HTTP**: Hypertext Transfer Protocol
- **JSON**: JavaScript Object Notation
- **LLM**: Large Language Model
- **MCP**: Model Context Protocol
- **MIME**: Multipurpose Internet Mail Extensions
- **OAuth**: Open Authorization
- **ORM**: Object-Relational Mapping
- **REST**: Representational State Transfer
- **RPC**: Remote Procedure Call
- **SSE**: Server-Sent Events
- **TLS**: Transport Layer Security
- **UI**: User Interface
- **URI**: Uniform Resource Identifier
- **URL**: Uniform Resource Locator
- **UUID**: Universally Unique Identifier
- **YAML**: YAML Ain't Markup Language

## Common Confusion

### Agent vs. Server
- **Agent**: AI personality that users interact with
- **Server**: MCP server that provides tools/resources to agents

### Client vs. Server (MCP)
- **Client**: Nanobot connecting to MCP servers
- **Server**: External service providing MCP capabilities

### Tool vs. Resource
- **Tool**: Function that agents execute (active)
- **Resource**: Data that agents read (passive)

### Session vs. Thread
- Same concept: A conversation context
- "Thread" is more user-facing terminology
- "Session" is more technical/backend terminology

### Flow vs. Workflow
- **Flow**: Nanobot's configuration for workflows
- **Workflow**: General term for multi-step processes

### Prompt vs. Instructions
- **Instructions**: System-level guidance for agent behavior
- **Prompt**: User input or template for generating text

### Profile vs. Environment
- **Profile**: Named configuration variant in Nanobot
- **Environment**: Operating system environment variables

## See Also

- [Overview](./OVERVIEW.md) - High-level introduction
- [Configuration](./CONFIGURATION.md) - Detailed config options
- [API Documentation](./API.md) - API reference
- [Architecture](./ARCHITECTURE.md) - System design
- [MCP Specification](https://modelcontextprotocol.io) - Protocol details

---

**Tip**: Use Ctrl+F (or Cmd+F) to search for specific terms in this glossary.
