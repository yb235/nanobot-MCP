# Nanobot Quick Reference

## Essential Commands

### Installation
```bash
# Homebrew
brew install nanobot-ai/tap/nanobot

# From source
git clone https://github.com/nanobot-ai/nanobot.git
cd nanobot && make
```

### Running
```bash
# Start with config
nanobot run config.yaml

# With custom settings
nanobot run config.yaml \
  --default-model gpt-4.1 \
  --listen-address 0.0.0.0:8080 \
  --debug

# Call tool directly
nanobot call server::tool --arg value

# List sessions
nanobot sessions list

# Validate config
nanobot run --validate config.yaml
```

## Configuration Templates

### Minimal Agent
```yaml
agents:
  assistant:
    model: "gpt-4.1"
    instructions: "You are a helpful assistant."

mcpServers: {}
```

### Agent with MCP Server
```yaml
agents:
  assistant:
    name: "My Assistant"
    model: "gpt-4.1"
    instructions: "You are helpful and use available tools."
    mcpServers:
      - "tools"

mcpServers:
  tools:
    url: "https://example.com/mcp"
```

### Multi-Agent Flow
```yaml
agents:
  researcher:
    model: "gpt-4.1"
    instructions: "Research topics thoroughly"
  writer:
    model: "gpt-4.1"
    instructions: "Write clear content"

flows:
  pipeline:
    input:
      fields:
        topic: { description: "Topic to research" }
    steps:
      - agent: { name: "researcher" }
        input: "${input.topic}"
        output: "research"
      - agent: { name: "writer" }
        input: "Write about: ${steps[0].output}"
```

### OAuth-Protected Server
```yaml
mcpServers:
  secure-api:
    url: "https://api.example.com/mcp"
    auth:
      oauthClientId: "${env.CLIENT_ID}"
      oauthClientSecret: "${env.CLIENT_SECRET}"
      oauthAuthorizeUrl: "https://auth.example.com/authorize"
      oauthScopes: ["read", "write"]
```

### Stdio Server (Local)
```yaml
mcpServers:
  local-tools:
    command: "python"
    args: ["-m", "mcp_server"]
    env:
      DEBUG: "1"
```

## Environment Variables

### Required
```bash
# Choose one based on model
export OPENAI_API_KEY=sk-...
export ANTHROPIC_API_KEY=sk-ant-...
```

### Optional
```bash
# Custom endpoints
export OPENAI_BASE_URL=https://custom-api.com
export ANTHROPIC_BASE_URL=https://custom-api.com

# Nanobot settings
export NANOBOT_DEFAULT_MODEL=gpt-4.1
export NANOBOT_STATE=./data/nanobot.db
```

## MCP Protocol Quick Reference

### Initialize
```json
{
  "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "roots": { "listChanged": true },
      "sampling": {},
      "elicitation": {}
    }
  }
}
```

### List Tools
```json
{
  "method": "tools/list",
  "params": {}
}
```

### Call Tool
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

### List Resources
```json
{
  "method": "resources/list",
  "params": {}
}
```

### Read Resource
```json
{
  "method": "resources/read",
  "params": {
    "uri": "file:///path/to/resource"
  }
}
```

### Subscribe to Resource
```json
{
  "method": "resources/subscribe",
  "params": {
    "uri": "file:///logs/app.log"
  }
}
```

## API Endpoints

### SSE Events Stream
```
GET /api/events/{thread_id}
Response: text/event-stream
```

**Event Types**:
- `history-start` - Start of history
- `message` - Chat message
- `history-end` - End of history
- `chat-in-progress` - Message generating
- `chat-done` - Message complete
- `elicitation/create` - User input request
- `error` - Error occurred

### Version
```
GET /api/version
Response: application/json
```

## Code Snippets

### JavaScript: Connect to SSE
```javascript
const eventSource = new EventSource('/api/events/thread-123');

eventSource.addEventListener('message', (e) => {
  const message = JSON.parse(e.data);
  console.log('Message:', message);
});

eventSource.addEventListener('chat-done', () => {
  console.log('Done!');
});

// Clean up
eventSource.close();
```

### Python: Call MCP Tool
```python
import requests

response = requests.post('http://localhost:8080/mcp/ui', json={
    'jsonrpc': '2.0',
    'id': 1,
    'method': 'tools/call',
    'params': {
        'name': 'nanobot.agentui.send',
        'arguments': {
            'prompt': 'Hello, world!'
        }
    }
})

print(response.json())
```

### Go: Create MCP Client
```go
import "github.com/nanobot-ai/nanobot/pkg/mcp"

client, err := mcp.NewClient(ctx, "server-name", mcp.Server{
    URL: "https://api.example.com/mcp",
})
if err != nil {
    return err
}
defer client.Close(false)

// List tools
tools, err := client.ListTools(ctx)

// Call tool
result, err := client.CallTool(ctx, "tool-name", map[string]any{
    "param": "value",
})
```

## File Locations

```
~/.config/nanobot/     # Config directory
  └── nanobot.yaml     # Configuration file

./nanobot.db           # SQLite database (default)
./nanobot.env          # Environment file
./logs/                # Log files (if configured)
```

## Common Patterns

### Agent with Custom Model Settings
```yaml
agents:
  precise:
    model: "gpt-4.1"
    temperature: 0.1      # More deterministic
    maxTokens: 4000
```

### Dynamic Instructions
```yaml
agents:
  contextual:
    instructions: |
      You are an assistant.
      Current time: {{env.TIME}}
      User: {{context.user}}
```

### Conditional Flow
```yaml
flows:
  conditional:
    steps:
      - if: "${input.type == 'research'}"
        then:
          - agent: { name: "researcher" }
            input: "${input.query}"
        else:
          - agent: { name: "assistant" }
            input: "${input.query}"
```

### Parallel Execution
```yaml
flows:
  parallel:
    steps:
      - parallel:
          - agent: { name: "agent1" }
            input: "Task 1"
          - agent: { name: "agent2" }
            input: "Task 2"
        output: "combined"
```

## Troubleshooting

### Connection Issues
```bash
# Test MCP server
curl https://api.example.com/mcp

# Check logs
nanobot run --debug config.yaml 2>&1 | tee debug.log

# Verify environment
env | grep -i "api_key\|nanobot"
```

### Database Issues
```bash
# Check database
sqlite3 nanobot.db ".tables"
sqlite3 nanobot.db "SELECT * FROM sessions;"

# Reset database
rm nanobot.db nanobot.db-*
```

### UI Issues
```bash
# Check if UI built
ls -la ui/dist/

# Rebuild UI
cd ui && npm install && npm run build

# Use UI dev mode
rm -rf ui/dist
cd ui && npm run dev
```

## Performance Tips

1. **Connection Pooling**: Reuse MCP clients
2. **Caching**: Cache tool lists and resources
3. **Streaming**: Use streaming for immediate feedback
4. **Concurrency**: Set appropriate max concurrency
5. **Lazy Loading**: Connect to servers on demand

## Security Checklist

- [ ] Store API keys in environment variables
- [ ] Use HTTPS for MCP servers in production
- [ ] Enable OAuth for sensitive servers
- [ ] Set appropriate CORS policies
- [ ] Encrypt database if storing sensitive data
- [ ] Use secure session IDs
- [ ] Validate all inputs
- [ ] Keep dependencies updated

## Development Shortcuts

```bash
# Run tests
go test ./...
go test -v ./pkg/mcp/...

# Format code
gofmt -w .
cd ui && npm run format

# Lint
golangci-lint run
cd ui && npm run lint

# Build
make

# Dev mode (UI hot reload)
rm -rf ui/dist && make
cd ui && npm run dev
```

## Useful Links

- **Docs**: [Complete Documentation](./README.md)
- **Overview**: [System Overview](./OVERVIEW.md)
- **Architecture**: [Architecture Guide](./ARCHITECTURE.md)
- **Configuration**: [Config Reference](./CONFIGURATION.md)
- **API**: [API Documentation](./API.md)
- **Workflow**: [Request Flows](./WORKFLOW.md)
- **Development**: [Dev Guide](./DEVELOPMENT.md)
- **Examples**: [Example Configs](../examples/)
- **MCP Spec**: https://modelcontextprotocol.io
- **GitHub**: https://github.com/nanobot-ai/nanobot

## Support Models

### OpenAI
- gpt-4.1
- gpt-4
- gpt-4-turbo
- gpt-3.5-turbo

### Anthropic
- claude-3-opus-20240229
- claude-3-sonnet-20240229
- claude-3-haiku-20240307

## Key Directories

```
nanobot/
├── pkg/
│   ├── agents/      - Agent execution
│   ├── api/         - HTTP API
│   ├── cli/         - CLI commands
│   ├── config/      - Configuration
│   ├── llm/         - LLM providers
│   ├── mcp/         - MCP protocol
│   ├── runtime/     - Runtime manager
│   ├── server/      - MCP server
│   ├── session/     - Session management
│   └── ...
├── ui/              - Web UI (Svelte)
├── examples/        - Example configs
└── docs/            - Documentation
```

## Quick Debugging

```bash
# Enable all logging
nanobot run --debug --trace config.yaml

# Check specific component
nanobot run config.yaml 2>&1 | grep -i "mcp\|llm\|agent"

# Inspect session
sqlite3 nanobot.db "
  SELECT sessionID, description, created 
  FROM sessions 
  ORDER BY created DESC 
  LIMIT 5;
"

# Test configuration
nanobot run --validate config.yaml
```

## Status Codes

### HTTP
- 200 - OK
- 400 - Bad Request
- 401 - Unauthorized
- 404 - Not Found
- 500 - Internal Server Error

### MCP (JSON-RPC)
- -32700 - Parse error
- -32600 - Invalid request
- -32601 - Method not found
- -32602 - Invalid params
- -32603 - Internal error
- -32000 - Server error

## Resource URIs

```
file:///path/to/file          # Local file
http://example.com/resource   # HTTP resource
https://api.com/data          # HTTPS resource
mcp://server/resource         # MCP-specific URI
```

## Expression Syntax

```yaml
# Access input
"${input.field}"

# Access previous step
"${steps[0].output}"

# Access environment
"${env.VAR_NAME}"

# Access context
"${context.property}"

# Conditional
"${input.value > 10 ? 'high' : 'low'}"
```

## Best Practices Summary

1. **Always** use environment variables for secrets
2. **Always** validate configuration before deploying
3. **Always** handle errors gracefully
4. **Prefer** streaming for long responses
5. **Prefer** reusing connections over creating new ones
6. **Document** your agent instructions clearly
7. **Test** MCP servers independently first
8. **Monitor** resource usage in production
9. **Use** profiles for different environments
10. **Keep** dependencies updated

---

**Need more details?** Check the [full documentation](./README.md).
