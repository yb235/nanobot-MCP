# Nanobot API Documentation

## HTTP API Reference

Nanobot exposes HTTP APIs for integration with external applications and UI interactions.

## Base URL

```
http://localhost:8080
```

(Configurable via `--listen-address` flag)

## Authentication

Currently, Nanobot uses session-based authentication:
- Session ID passed via URL path
- Optional OAuth for MCP servers
- Future: API key authentication

## API Endpoints

### 1. Events Stream (SSE)

**Endpoint**: `GET /api/events/{thread_id}`

**Description**: Server-Sent Events stream for real-time conversation updates.

**Path Parameters**:
- `thread_id` (string): Session/conversation identifier

**Response**: `text/event-stream`

**Event Types**:

#### `history-start`
Marks the beginning of historical message replay.
```
event: history-start
data: null
```

#### `message`
A chat message (user or assistant).
```
event: message
data: {
  "id": "msg_123",
  "role": "assistant",
  "created": "2024-01-01T00:00:00Z",
  "items": [
    {
      "type": "text",
      "text": "Hello! How can I help you?"
    }
  ]
}
```

#### `history-end`
Marks the end of historical message replay.
```
event: history-end
data: null
```

#### `chat-in-progress`
Indicates ongoing message generation.
```
event: chat-in-progress
data: {
  "messageId": "msg_123"
}
```

#### `chat-done`
Message generation completed.
```
event: chat-done
data: {
  "messageId": "msg_123"
}
```

#### `elicitation/create`
Request for user input (MCP-UI).
```
event: elicitation/create
data: {
  "id": "elicit_123",
  "message": "Please enter your API key",
  "requestedSchema": {
    "type": "object",
    "properties": {
      "apiKey": {
        "type": "string",
        "title": "API Key",
        "description": "Your API key"
      }
    }
  }
}
```

#### `error`
An error occurred.
```
event: error
data: {
  "message": "Failed to execute tool"
}
```

**Example Client** (JavaScript):
```javascript
const eventSource = new EventSource('/api/events/abc-123');

eventSource.addEventListener('message', (e) => {
  const message = JSON.parse(e.data);
  console.log('Message:', message);
});

eventSource.addEventListener('chat-done', (e) => {
  console.log('Chat completed');
});

eventSource.onerror = (e) => {
  console.error('Connection error:', e);
};
```

### 2. Version

**Endpoint**: `GET /api/version`

**Description**: Get server version information.

**Response**: `application/json`
```json
{
  "version": "0.1.0",
  "gitCommit": "abc123",
  "buildDate": "2024-01-01"
}
```

### 3. MCP Proxy (Internal)

These endpoints are used internally by the UI to communicate with the MCP server.

**Base Path**: `/mcp/ui`

The UI communicates with Nanobot's internal MCP server using the MCP protocol over HTTP.

**Example MCP Request**:
```
POST /mcp/ui
Content-Type: application/json

{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "nanobot.agentui.send",
    "arguments": {
      "prompt": "Hello, world!"
    }
  }
}
```

**Response**:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Response from agent"
      }
    ]
  }
}
```

## MCP Protocol Reference

Nanobot implements the Model Context Protocol for agent communication.

### Protocol Version

Current: `2024-11-05`

### Transport

- **HTTP/HTTPS**: For remote MCP servers
- **stdio**: For local process MCP servers

### Message Format

All MCP messages follow JSON-RPC 2.0:

```json
{
  "jsonrpc": "2.0",
  "id": "unique-id",
  "method": "method-name",
  "params": { }
}
```

### Core Methods

#### 1. Initialize

**Request**:
```json
{
  "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "roots": { "listChanged": true },
      "sampling": {},
      "elicitation": {}
    },
    "clientInfo": {
      "name": "nanobot",
      "version": "0.1.0"
    }
  }
}
```

**Response**:
```json
{
  "result": {
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "tools": { "listChanged": true },
      "resources": { "subscribe": true },
      "prompts": { "listChanged": false }
    },
    "serverInfo": {
      "name": "my-mcp-server",
      "version": "1.0.0"
    },
    "instructions": "Optional instructions for using this server"
  }
}
```

#### 2. List Tools

**Request**:
```json
{
  "method": "tools/list",
  "params": {}
}
```

**Response**:
```json
{
  "result": {
    "tools": [
      {
        "name": "get_weather",
        "description": "Get current weather for a location",
        "inputSchema": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "City name"
            }
          },
          "required": ["location"]
        }
      }
    ]
  }
}
```

#### 3. Call Tool

**Request**:
```json
{
  "method": "tools/call",
  "params": {
    "name": "get_weather",
    "arguments": {
      "location": "San Francisco"
    }
  }
}
```

**Response**:
```json
{
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Temperature: 72°F, Conditions: Sunny"
      }
    ],
    "isError": false
  }
}
```

#### 4. List Resources

**Request**:
```json
{
  "method": "resources/list",
  "params": {}
}
```

**Response**:
```json
{
  "result": {
    "resources": [
      {
        "uri": "file:///docs/README.md",
        "name": "README",
        "description": "Project documentation",
        "mimeType": "text/markdown"
      }
    ]
  }
}
```

#### 5. Read Resource

**Request**:
```json
{
  "method": "resources/read",
  "params": {
    "uri": "file:///docs/README.md"
  }
}
```

**Response**:
```json
{
  "result": {
    "contents": [
      {
        "uri": "file:///docs/README.md",
        "mimeType": "text/markdown",
        "text": "# Project Documentation\n..."
      }
    ]
  }
}
```

#### 6. Subscribe to Resource

**Request**:
```json
{
  "method": "resources/subscribe",
  "params": {
    "uri": "file:///logs/app.log"
  }
}
```

**Response**:
```json
{
  "result": {}
}
```

**Notification** (when resource changes):
```json
{
  "method": "notifications/resources/updated",
  "params": {
    "uri": "file:///logs/app.log"
  }
}
```

#### 7. List Prompts

**Request**:
```json
{
  "method": "prompts/list",
  "params": {}
}
```

**Response**:
```json
{
  "result": {
    "prompts": [
      {
        "name": "code_review",
        "description": "Review code for issues",
        "arguments": [
          {
            "name": "code",
            "description": "Code to review",
            "required": true
          }
        ]
      }
    ]
  }
}
```

#### 8. Get Prompt

**Request**:
```json
{
  "method": "prompts/get",
  "params": {
    "name": "code_review",
    "arguments": {
      "code": "function hello() { console.log('hi'); }"
    }
  }
}
```

**Response**:
```json
{
  "result": {
    "messages": [
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "Review this code: function hello() { ... }"
        }
      }
    ]
  }
}
```

### MCP-UI Extensions

#### Elicitation (User Input)

**Request** (from server to client):
```json
{
  "method": "elicitation/elicit",
  "params": {
    "message": "Please provide your configuration",
    "requestedSchema": {
      "type": "object",
      "properties": {
        "apiKey": {
          "type": "string",
          "title": "API Key"
        },
        "endpoint": {
          "type": "string",
          "title": "Endpoint URL"
        }
      }
    }
  }
}
```

**Response** (from client):
```json
{
  "result": {
    "action": "accept",
    "content": {
      "apiKey": "sk-...",
      "endpoint": "https://api.example.com"
    }
  }
}
```

Actions: `accept`, `decline`, `cancel`

#### Sampling (LLM Delegation)

**Request** (from server to client):
```json
{
  "method": "sampling/createMessage",
  "params": {
    "messages": [
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "What is 2+2?"
        }
      }
    ],
    "systemPrompt": "You are a math tutor",
    "maxTokens": 100
  }
}
```

**Response** (from client):
```json
{
  "result": {
    "role": "assistant",
    "content": {
      "type": "text",
      "text": "2 + 2 = 4"
    },
    "model": "gpt-4.1",
    "stopReason": "endTurn"
  }
}
```

## Data Types

### ChatMessage

```typescript
interface ChatMessage {
  id: string;
  created?: string;  // ISO 8601 timestamp
  role: 'user' | 'assistant';
  items?: ChatMessageItem[];
  hasMore?: boolean;
}
```

### ChatMessageItem

```typescript
type ChatMessageItem = 
  | ChatMessageItemText
  | ChatMessageItemImage
  | ChatMessageItemAudio
  | ChatMessageItemToolCall
  | ChatMessageItemResource;
```

#### Text Item
```typescript
interface ChatMessageItemText {
  type: 'text';
  text: string;
}
```

#### Image Item
```typescript
interface ChatMessageItemImage {
  type: 'image';
  data: string;      // Base64 or URL
  mimeType: string;  // e.g., "image/png"
}
```

#### Tool Call Item
```typescript
interface ChatMessageItemToolCall {
  type: 'tool';
  name: string;
  arguments?: string;  // JSON string
  callID?: string;
  output?: {
    isError?: boolean;
    content?: ToolOutputItem[];
  };
}
```

#### Resource Item
```typescript
interface ChatMessageItemResource {
  type: 'resource';
  resource: {
    uri: string;
    name?: string;
    mimeType: string;
    text?: string;
    blob?: string;  // Base64
  };
}
```

### Elicitation

```typescript
interface Elicitation {
  id: string;
  message: string;
  requestedSchema: {
    type: 'object';
    properties: {
      [key: string]: {
        type: string;
        title?: string;
        description?: string;
      };
    };
  };
}
```

### Agent

```typescript
interface Agent {
  name?: string;
  description?: string;
  icon?: string;       // Data URI or URL
  iconDark?: string;
  starterMessages?: string[];
}
```

## Error Handling

### HTTP Errors

- `400 Bad Request`: Invalid parameters
- `401 Unauthorized`: Authentication required
- `404 Not Found`: Resource not found
- `500 Internal Server Error`: Server error

### MCP Errors

MCP uses JSON-RPC error codes:

```json
{
  "error": {
    "code": -32000,
    "message": "Server error",
    "data": {
      "details": "Additional error information"
    }
  }
}
```

**Standard Codes**:
- `-32700`: Parse error
- `-32600`: Invalid request
- `-32601`: Method not found
- `-32602`: Invalid params
- `-32603`: Internal error
- `-32000 to -32099`: Server-defined errors

## Rate Limiting

Currently not implemented. Future versions may include:
- Per-session rate limits
- Token bucket algorithm
- Configurable limits per agent

## CORS

CORS is enabled by default for all origins:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
```

Configure via server settings for production.

## WebSocket Support

Not currently implemented. Future feature for:
- Bidirectional communication
- Lower latency
- Better mobile support

## Example Integrations

### Python Client

```python
import requests
import json

def send_message(thread_id, message):
    """Send message via MCP protocol"""
    response = requests.post(
        'http://localhost:8080/mcp/ui',
        json={
            'jsonrpc': '2.0',
            'id': 1,
            'method': 'tools/call',
            'params': {
                'name': 'nanobot.agentui.send',
                'arguments': {
                    'prompt': message
                }
            }
        }
    )
    return response.json()

def listen_events(thread_id):
    """Listen to SSE stream"""
    response = requests.get(
        f'http://localhost:8080/api/events/{thread_id}',
        stream=True
    )
    
    for line in response.iter_lines():
        if line.startswith(b'data:'):
            data = json.loads(line[5:])
            print('Event:', data)
```

### JavaScript/TypeScript Client

```typescript
class NanobotClient {
  private baseUrl: string;
  
  constructor(baseUrl = 'http://localhost:8080') {
    this.baseUrl = baseUrl;
  }
  
  subscribe(threadId: string, onMessage: (msg: any) => void) {
    const eventSource = new EventSource(
      `${this.baseUrl}/api/events/${threadId}`
    );
    
    eventSource.addEventListener('message', (e) => {
      onMessage(JSON.parse(e.data));
    });
    
    return () => eventSource.close();
  }
  
  async sendMessage(message: string): Promise<any> {
    const response = await fetch(`${this.baseUrl}/mcp/ui`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        jsonrpc: '2.0',
        id: Date.now(),
        method: 'tools/call',
        params: {
          name: 'nanobot.agentui.send',
          arguments: { prompt: message }
        }
      })
    });
    
    return response.json();
  }
}
```

## Best Practices

1. **Connection Management**
   - Reuse SSE connections
   - Handle reconnection
   - Close unused streams

2. **Error Handling**
   - Always check for `error` field
   - Implement retry logic
   - Log errors with context

3. **Performance**
   - Use streaming for long responses
   - Debounce rapid requests
   - Cache tool lists

4. **Security**
   - Use HTTPS in production
   - Validate all inputs
   - Sanitize output for display

## Support

For API questions:
- GitHub Issues: https://github.com/nanobot-ai/nanobot/issues
- Documentation: https://nanobot.ai/docs
