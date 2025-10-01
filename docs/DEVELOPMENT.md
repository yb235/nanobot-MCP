# Nanobot Development Guide

## Getting Started

This guide covers how to set up a development environment, understand the codebase, and contribute to Nanobot.

## Prerequisites

### Required

- **Go**: 1.21 or later
- **Node.js**: 18 or later (for UI development)
- **npm**: 9 or later
- **Git**: For version control

### Optional

- **Docker**: For containerized development
- **Make**: For build automation
- **SQLite**: For database inspection

## Development Setup

### 1. Clone Repository

```bash
git clone https://github.com/nanobot-ai/nanobot.git
cd nanobot
```

### 2. Install Dependencies

#### Go Dependencies
```bash
go mod download
```

#### UI Dependencies
```bash
cd ui
npm install
cd ..
```

### 3. Build

```bash
make
```

This creates `./bin/nanobot` executable.

### 4. Run Development Server

#### Option A: Full Build
```bash
make
./bin/nanobot run examples/random.yaml
```

#### Option B: UI Development Mode
```bash
# Terminal 1: Run backend
rm -rf ui/dist  # Force UI dev mode
make
./bin/nanobot run examples/random.yaml

# Terminal 2: Run UI dev server
cd ui
npm run dev
```

The UI dev server runs on port 5173, and Nanobot proxies UI requests to it.

## Project Structure

```
nanobot/
├── main.go                 # Entry point
├── Makefile               # Build automation
├── go.mod                 # Go dependencies
├── examples/              # Example configurations
│   ├── blackjack.yaml
│   ├── shopping.yaml
│   └── ...
├── pkg/                   # Go packages
│   ├── agents/           # Agent execution
│   ├── api/              # HTTP API
│   ├── cli/              # Command-line interface
│   ├── config/           # Configuration loading
│   ├── llm/              # LLM provider integrations
│   ├── mcp/              # MCP protocol implementation
│   ├── runtime/          # Runtime manager
│   ├── server/           # MCP server implementation
│   ├── session/          # Session management
│   └── ...
├── ui/                    # Web UI (Svelte)
│   ├── src/
│   │   ├── lib/          # Shared components
│   │   ├── routes/       # Page routes
│   │   └── ...
│   ├── package.json
│   └── vite.config.ts
└── docs/                  # Documentation
```

## Key Packages

### pkg/mcp

**Purpose**: MCP protocol implementation

**Key Files**:
- `client.go`: MCP client
- `session.go`: Session management
- `types.go`: Protocol types
- `httpclient.go`: HTTP transport
- `stdio.go`: Stdio transport

**Example Usage**:
```go
client, err := mcp.NewClient(ctx, "server-name", config)
if err != nil {
    return err
}
defer client.Close(false)

// List tools
tools, err := client.ListTools(ctx)

// Call tool
result, err := client.CallTool(ctx, "tool-name", arguments)
```

### pkg/llm

**Purpose**: LLM provider abstractions

**Key Files**:
- `llm.go`: Core interfaces
- `anthropic/client.go`: Anthropic integration
- `responses/client.go`: OpenAI integration

**Adding a New Provider**:

```go
// 1. Implement Client interface
type MyProviderClient struct {
    apiKey string
    baseURL string
}

func (c *MyProviderClient) Call(ctx context.Context, 
    input Input, opts ...CallOption) (Output, error) {
    // Implementation
}

// 2. Register in llm package
func NewMyProviderClient(config Config) Client {
    return &MyProviderClient{
        apiKey: config.APIKey,
        baseURL: config.BaseURL,
    }
}

// 3. Add to runtime selection logic
```

### pkg/agents

**Purpose**: Agent execution logic

**Key Files**:
- `run.go`: Main execution loop
- `toolcall.go`: Tool call handling

**Execution Flow**:
```go
func Run(ctx context.Context, input string, opts ...RunOption) (*Result, error) {
    // 1. Create MCP clients
    // 2. Build LLM context
    // 3. Stream LLM responses
    // 4. Handle tool calls
    // 5. Return result
}
```

### pkg/server

**Purpose**: MCP Host server implementation

**Key Files**:
- `server.go`: Message routing
- `tools.go`: Tool handlers
- `resources.go`: Resource handlers

**Adding a Handler**:
```go
func (s *Server) init() {
    s.handlers = append(s.handlers,
        handle[MyRequest]("my/method", s.handleMyMethod))
}

func (s *Server) handleMyMethod(ctx context.Context, 
    msg mcp.Message, payload MyRequest) error {
    // Process request
    result := MyResult{ /* ... */ }
    return msg.Reply(ctx, result)
}
```

### pkg/api

**Purpose**: HTTP API layer

**Key Files**:
- `handler.go`: Server setup
- `events.go`: SSE endpoint
- `routes.go`: Route definitions

**Adding an Endpoint**:
```go
// 1. Add route
func routes(s *server, mux *http.ServeMux) {
    mux.Handle("GET /api/myendpoint", s.api(MyHandler))
}

// 2. Implement handler
func MyHandler(rw http.ResponseWriter, req *http.Request) error {
    // Handle request
    json.NewEncoder(rw).Encode(response)
    return nil
}
```

### ui/src/lib

**Purpose**: Frontend components and logic

**Key Files**:
- `chat.svelte.ts`: Chat API and state
- `types.ts`: TypeScript types
- `markdown.ts`: Message rendering

**Adding UI Features**:
```typescript
// 1. Add to ChatService
class ChatService {
    async myNewFeature() {
        return this.api.callMCPTool('my-tool', { /* params */ });
    }
}

// 2. Use in component
<script>
    const service = new ChatService();
    
    async function handleAction() {
        await service.myNewFeature();
    }
</script>
```

## Development Workflow

### 1. Make Changes

Edit files in appropriate package:
```bash
# Backend changes
vim pkg/mcp/client.go

# UI changes
vim ui/src/lib/chat.svelte.ts
```

### 2. Build and Test

```bash
# Full build
make

# Run tests
go test ./...

# Run specific package tests
go test ./pkg/mcp/...

# UI tests
cd ui
npm test
```

### 3. Run Locally

```bash
./bin/nanobot run examples/random.yaml
```

### 4. Debug

#### Go Debugging with Delve
```bash
dlv debug ./main.go -- run examples/random.yaml
```

#### Print Debugging
```go
import "github.com/nanobot-ai/nanobot/pkg/log"

log.Debugf(ctx, "Value: %+v", value)
```

#### UI Debugging
- Browser DevTools
- Console logging
- Svelte DevTools extension

## Testing

### Unit Tests

**Location**: `*_test.go` files alongside source

**Example**:
```go
func TestClientInitialize(t *testing.T) {
    ctx := context.Background()
    client, err := mcp.NewClient(ctx, "test", config)
    if err != nil {
        t.Fatalf("Failed to create client: %v", err)
    }
    defer client.Close(false)
    
    // Test initialization
    if client.Session.InitializeResult.ProtocolVersion == "" {
        t.Error("Protocol version not set")
    }
}
```

**Run**:
```bash
go test ./pkg/mcp/
```

### Integration Tests

Test complete workflows:

```go
func TestAgentExecution(t *testing.T) {
    // Setup
    config := loadTestConfig()
    runtime, _ := runtime.NewRuntime(llmConfig)
    
    // Execute
    result, err := agents.Run(ctx, "test input", 
        agents.WithRuntime(runtime))
    
    // Verify
    if err != nil {
        t.Fatalf("Agent execution failed: %v", err)
    }
    if result.Output == "" {
        t.Error("No output generated")
    }
}
```

### UI Tests

```typescript
import { describe, it, expect } from 'vitest';
import { ChatService } from './chat.svelte';

describe('ChatService', () => {
    it('should send message', async () => {
        const service = new ChatService();
        await service.send('Hello');
        expect(service.messages.length).toBe(1);
    });
});
```

**Run**:
```bash
cd ui
npm test
```

## Code Style

### Go

Follow standard Go conventions:
- Use `gofmt` for formatting
- Follow effective Go guidelines
- Use meaningful variable names
- Add comments for exported functions

**Format**:
```bash
gofmt -w .
```

**Lint**:
```bash
golangci-lint run
```

### TypeScript/Svelte

Follow Svelte and TypeScript conventions:
- Use Prettier for formatting
- Follow TypeScript best practices
- Use runes mode ($state, $derived)

**Format**:
```bash
cd ui
npm run format
```

**Lint**:
```bash
cd ui
npm run lint
```

## Building

### Standard Build

```bash
make
```

This:
1. Builds UI (if needed)
2. Embeds UI in Go binary
3. Compiles Go binary

### Development Build

```bash
# Skip UI build (use dev server)
rm -rf ui/dist
make
```

### Release Build

```bash
# Uses goreleaser
goreleaser build --snapshot --clean
```

## Debugging Tips

### 1. Enable Debug Logging

```bash
nanobot run --debug config.yaml
```

### 2. Trace MCP Messages

```bash
nanobot run --trace config.yaml
```

### 3. Inspect Database

```bash
sqlite3 nanobot.db
> .schema
> SELECT * FROM sessions;
```

### 4. Monitor HTTP Traffic

```bash
# Use proxy or browser DevTools
# Watch /api/events for SSE stream
```

### 5. Check MCP Server Logs

For stdio servers:
```yaml
mcpServers:
  debug-server:
    command: "python"
    args: ["-u", "server.py"]  # -u for unbuffered output
    env:
      DEBUG: "1"
```

## Common Development Tasks

### Add a New MCP Method

1. **Define types** in `pkg/mcp/types.go`:
```go
type MyNewRequest struct {
    Param string `json:"param"`
}

type MyNewResult struct {
    Output string `json:"output"`
}
```

2. **Add handler** in `pkg/server/server.go`:
```go
func (s *Server) init() {
    // Add to handlers
    s.handlers = append(s.handlers,
        handle[mcp.MyNewRequest]("my/new", s.handleMyNew))
}

func (s *Server) handleMyNew(ctx context.Context, 
    msg mcp.Message, payload mcp.MyNewRequest) error {
    result := mcp.MyNewResult{
        Output: "Processed: " + payload.Param,
    }
    return msg.Reply(ctx, result)
}
```

3. **Add client method** in `pkg/mcp/client.go`:
```go
func (c *Client) MyNew(ctx context.Context, param string) (*MyNewResult, error) {
    var result MyNewResult
    err := c.Session.Call(ctx, "my/new", MyNewRequest{
        Param: param,
    }, &result)
    return &result, err
}
```

### Add a Configuration Option

1. **Add to config struct** in `pkg/types/config.go`:
```go
type Agent struct {
    // Existing fields...
    MyNewOption string `json:"myNewOption,omitempty"`
}
```

2. **Add validation** in `pkg/types/validate.go`:
```go
func (a Agent) validate(name string, config Config) error {
    // Add validation logic
    if a.MyNewOption != "" && !isValid(a.MyNewOption) {
        return fmt.Errorf("invalid myNewOption: %s", a.MyNewOption)
    }
    return nil
}
```

3. **Use in agent execution**:
```go
func Run(ctx context.Context, input string, opts ...RunOption) {
    agent := types.AgentFromContext(ctx)
    // Use agent.MyNewOption
}
```

### Add a UI Component

1. **Create component** `ui/src/lib/components/MyComponent.svelte`:
```svelte
<script lang="ts">
    import { type MyData } from '../types';
    
    interface Props {
        data: MyData;
    }
    
    let { data }: Props = $props();
</script>

<div class="my-component">
    {data.field}
</div>

<style>
    .my-component {
        /* styles */
    }
</style>
```

2. **Use in route**:
```svelte
<script>
    import MyComponent from '$lib/components/MyComponent.svelte';
</script>

<MyComponent data={myData} />
```

## Contributing

### 1. Fork and Clone

```bash
git clone https://github.com/YOUR-USERNAME/nanobot.git
cd nanobot
git remote add upstream https://github.com/nanobot-ai/nanobot.git
```

### 2. Create Branch

```bash
git checkout -b feature/my-feature
```

### 3. Make Changes

- Write code
- Add tests
- Update documentation

### 4. Test

```bash
# Run all tests
make test

# Check formatting
make fmt

# Lint code
make lint
```

### 5. Commit

```bash
git add .
git commit -m "Add my feature"
```

**Commit Message Format**:
```
<type>: <description>

[optional body]

[optional footer]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

### 6. Push and PR

```bash
git push origin feature/my-feature
```

Create Pull Request on GitHub.

## Release Process

Releases are automated via GitHub Actions and GoReleaser.

### Version Bump

```bash
# Update version
git tag v0.2.0
git push origin v0.2.0
```

### GoReleaser

Automatically:
- Builds binaries for multiple platforms
- Creates GitHub release
- Uploads artifacts
- Updates Homebrew formula

## Troubleshooting Development

### Build Failures

**UI build fails**:
```bash
cd ui
rm -rf node_modules
npm install
```

**Go build fails**:
```bash
go mod tidy
go mod download
```

### Runtime Errors

**MCP connection fails**:
- Check MCP server is running
- Verify URL/command correct
- Check authentication

**Database locked**:
- Close other Nanobot instances
- Delete `nanobot.db-*` lock files

**UI not loading**:
- Check port 8080 available
- Verify UI built correctly
- Check browser console

## Resources

### Documentation

- [Architecture](./ARCHITECTURE.md)
- [API Reference](./API.md)
- [Configuration](./CONFIGURATION.md)
- [Workflow](./WORKFLOW.md)

### External Links

- [MCP Specification](https://modelcontextprotocol.io)
- [Go Documentation](https://golang.org/doc/)
- [Svelte Documentation](https://svelte.dev)
- [SvelteKit Documentation](https://kit.svelte.dev)

### Community

- GitHub Issues: Bug reports and features
- GitHub Discussions: Questions and ideas
- Discord: Real-time chat (coming soon)

## Best Practices

1. **Write Tests**: Cover new functionality
2. **Document**: Add comments and docs
3. **Follow Style**: Use formatters
4. **Small PRs**: Focus on one thing
5. **Test Locally**: Verify before pushing
6. **Update Docs**: Keep documentation current

## Getting Help

- Check existing documentation
- Search GitHub issues
- Ask in discussions
- Review example code
- Read source comments

Happy coding! 🚀
