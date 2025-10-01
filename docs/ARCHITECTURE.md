# Nanobot Architecture

## System Architecture Overview

Nanobot follows a layered architecture that separates concerns and enables flexibility:

```
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Web UI     │  │   CLI        │  │   HTTP API   │      │
│  │  (Svelte)    │  │   (Cobra)    │  │   (SSE)      │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                    Application Layer                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Server     │  │   Runtime    │  │   Agents     │      │
│  │  (MCP Host)  │  │   Manager    │  │   Executor   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                    Integration Layer                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  MCP Client  │  │  LLM Client  │  │   Session    │      │
│  │  (Protocol)  │  │  (Adapters)  │  │   Manager    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│                    Infrastructure Layer                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Database   │  │   File I/O   │  │   Network    │      │
│  │   (SQLite)   │  │   (Config)   │  │   (HTTP)     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└──────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Main Entry Point (`main.go`)

The application starts from `main.go`:

```go
func main() {
    if len(os.Args) > 2 && os.Args[1] == "_exec" {
        // Supervisor mode for managing child processes
        supervise.Daemon()
    } else {
        // Normal CLI mode
        cmd.Main(cli.New())
    }
}
```

**Responsibilities**:
- Bootstrap the application
- Handle supervisor mode (for MCP stdio servers)
- Initialize the CLI framework

### 2. CLI Layer (`pkg/cli/`)

The CLI provides the command-line interface using Cobra:

**Key Commands**:
- `nanobot run` - Start the MCP host server
- `nanobot call` - Invoke a tool directly
- `nanobot sessions` - Manage sessions
- `nanobot targets` - List available targets

**Configuration**:
```go
type Nanobot struct {
    DefaultModel        string            // LLM model
    OpenAIAPIKey        string            // OpenAI credentials
    AnthropicAPIKey     string            // Anthropic credentials
    MaxConcurrency      int               // Parallel execution limit
    State               string            // Database path
    // ... more fields
}
```

### 3. MCP Package (`pkg/mcp/`)

The MCP package implements the Model Context Protocol:

#### Session Management (`session.go`)

```go
type Session struct {
    ctx               context.Context
    wire              Wire              // Communication channel
    handler           MessageHandler    // Message processor
    pendingRequest    PendingRequests   // Request tracking
    InitializeResult  InitializeResult  // Handshake result
    attributes        map[string]any    // Session state
    sessionManager    SessionStore      // Persistence
}
```

**Wire Types**:
- **HTTP Wire** (`httpclient.go`): For HTTP/HTTPS MCP servers
- **Stdio Wire** (`stdio.go`): For local process MCP servers

#### Client (`client.go`)

```go
type Client struct {
    Session *Session
}

func NewClient(ctx context.Context, serverName string, 
               config Server, opts ...ClientOption) (*Client, error)
```

**Client Options**:
- `OnSampling`: Handle LLM delegation requests
- `OnElicit`: Handle user input requests
- `OnRoots`: Provide root URIs
- `SessionState`: Resume previous session

#### Protocol Types (`types.go`)

Key protocol structures:
- `InitializeRequest/Result`: Protocol handshake
- `CallToolRequest/Result`: Tool invocation
- `ListResourcesRequest/Result`: Resource discovery
- `SubscribeRequest`: Resource change notifications
- `ElicitRequest/Result`: User input (MCP-UI extension)

### 4. Server Package (`pkg/server/`)

The server implements the MCP Host functionality:

```go
type Server struct {
    handlers []handler          // Message handlers
    runtime  *runtime.Runtime   // Execution engine
    data     *sessiondata.Data  // Session data
    config   types.ConfigFactory // Configuration
    manager  *session.Manager   // Session storage
}
```

**Handler Registration**:
```go
s.handlers = []handler{
    handle[mcp.InitializeRequest]("initialize", s.handleInitialize),
    handle[mcp.ListToolsRequest]("tools/list", s.handleListTools),
    handle[mcp.CallToolRequest]("tools/call", s.handleCallTool),
    handle[mcp.ListPromptsRequest]("prompts/list", s.handleListPrompts),
    handle[mcp.ListResourcesRequest]("resources/list", s.handleListResources),
    // ... more handlers
}
```

### 5. Runtime Package (`pkg/runtime/`)

The runtime manages tool execution and MCP client lifecycle:

```go
type Runtime struct {
    llmConfig llm.Config
    opt       Options
}

type Options struct {
    Roots            []mcp.Root
    Profiles         []string
    MaxConcurrency   int
    CallbackHandler  mcp.CallbackHandler
    TokenStorage     mcp.TokenStorage
    OAuthRedirectURL string
    DSN              string
}
```

**Key Methods**:
- `CallFromCLI`: Execute tool from command line
- `getToolFromRef`: Resolve tool by server/name reference

### 6. Session Manager (`pkg/session/`)

Manages persistent session state:

```go
type Manager struct {
    DB           *Store              // Database connection
    liveSessions map[string]liveSession // Active sessions
}

type Session struct {
    SessionID   string
    AccountID   string
    Description string
    IsPublic    bool
    Config      ConfigWrapper
    Cwd         string
}
```

**Database Operations**:
- `Get(ctx, sessionID)`: Load session
- `Store(ctx, sessionID, session)`: Save session
- `Delete(ctx, sessionID)`: Remove session
- `List(ctx, accountID)`: List user sessions

### 7. LLM Package (`pkg/llm/`)

Abstracts LLM provider interactions:

```go
type Config struct {
    DefaultModel            string
    OpenAI                  openai.Config
    Anthropic              anthropic.Config
}

type Client interface {
    Call(ctx context.Context, input Input, opts ...CallOption) (Output, error)
}
```

**Provider Adapters**:
- **OpenAI** (`llm/responses`): OpenAI Responses API
- **Anthropic** (`llm/anthropic`): Claude Messages API

**Message Translation**:
- MCP messages → Provider-specific format
- Tool calls → Function calling
- Resources → Context injection

### 8. Agents Package (`pkg/agents/`)

Executes agent logic with MCP and LLM:

```go
func Run(ctx context.Context, input string, opts ...RunOption) (*Result, error)
```

**Agent Execution Flow**:
1. Load agent configuration
2. Create MCP clients for connected servers
3. Build LLM prompt with instructions
4. Stream LLM responses
5. Execute tool calls via MCP
6. Inject tool results back to LLM
7. Continue until completion

### 9. API Layer (`pkg/api/`)

Provides HTTP API for UI and external clients:

**Routes** (`routes.go`):
```go
mux.Handle("GET /api/events/{thread_id}", s.withContext(Events))
mux.Handle("GET /api/version", s.api(Version))
```

**Events Endpoint** (`events.go`):
- Server-Sent Events (SSE) stream
- Real-time conversation updates
- History replay on connection
- Progress notifications

**Context Management** (`handler.go`):
```go
type Context struct {
    ChatClient     *mcp.Client      // MCP client for agent
    SessionManager *session.Manager // Session persistence
    MCPServer      mcp.Server       // Server configuration
}
```

### 10. UI (`ui/`)

Svelte-based web interface:

**Architecture**:
- **SvelteKit**: SSR/SSG framework
- **TypeScript**: Type safety
- **Vite**: Build tool
- **Embedded**: Compiled into Go binary

**Key Components**:
- `ChatAPI` (`chat.svelte.ts`): Backend communication
- `ChatService` (`chat.svelte.ts`): State management
- Event handlers for SSE streams
- Message rendering with Markdown

## Data Flow

### Request Flow: Tool Call from UI

```
1. User types message in UI
   └─> POST /api/mcp/ui
   
2. API receives request
   └─> Create/resume session
   └─> Create MCP client
   
3. MCP client connects to agent
   └─> Initialize protocol
   └─> List available tools
   
4. Agent receives message
   └─> Call LLM with context
   
5. LLM decides to use tool
   └─> Return tool call request
   
6. Agent calls tool via MCP
   └─> tools/call to MCP server
   
7. MCP server executes tool
   └─> Return result
   
8. Agent injects result to LLM
   └─> LLM generates final response
   
9. Response streamed to UI
   └─> SSE events
   └─> UI updates incrementally
```

### Configuration Loading Flow

```
1. Parse command-line flags
   └─> CLI options override defaults
   
2. Load environment file
   └─> Read nanobot.env
   └─> Apply environment variables
   
3. Load YAML configuration
   └─> Read nanobot.yaml
   └─> Merge with extends
   └─> Apply profile overlays
   
4. Validate configuration
   └─> Check required fields
   └─> Validate MCP server URLs
   └─> Verify agent references
   
5. Create runtime
   └─> Initialize LLM clients
   └─> Set up session manager
   └─> Prepare MCP client factory
```

## Communication Patterns

### 1. Synchronous RPC (CLI)

```
CLI Command → Runtime → MCP Client → MCP Server → Result
```

Used for:
- Direct tool invocation
- Configuration testing
- One-off operations

### 2. Asynchronous Streaming (UI)

```
UI Request → SSE Stream → Agent Loop → LLM Calls → Tool Calls → Updates
```

Used for:
- Interactive chat
- Long-running operations
- Real-time feedback

### 3. Event-Based Updates

```
MCP Server → Resource Change → Notification → Session → UI Update
```

Used for:
- Resource subscriptions
- Live data updates
- Multi-user coordination

## State Management

### Session State

**Storage**:
- SQLite database (`nanobot.db`)
- JSON-encoded configuration
- Encrypted sensitive data (optional)

**Contents**:
- Conversation history
- Agent configuration
- MCP server connections
- User preferences
- Resource subscriptions

### In-Memory State

**Runtime State**:
- Active MCP connections
- Pending requests
- Event listeners
- Cache for frequently accessed data

**Lifecycle**:
- Created on first request
- Persisted on updates
- Cleaned up on session close

## Security Architecture

### Authentication

**OAuth 2.0** (optional):
- Authorization code flow
- Token refresh handling
- Secure token storage
- Per-server credentials

### Session Security

- Session ID generation (UUID)
- Account isolation
- Public/private session control
- Encryption key configuration

### Network Security

- CORS support
- HTTPS for production
- OAuth callback handling
- Input validation

## Scalability Considerations

### Current Design

- Single-process server
- SQLite for state
- In-memory session cache
- HTTP/1.1 connections

### Scaling Options

1. **Horizontal Scaling**:
   - Shared database (PostgreSQL)
   - Redis for session cache
   - Load balancer distribution

2. **Vertical Scaling**:
   - Increase max concurrency
   - Connection pooling
   - Resource caching

3. **Optimization**:
   - Lazy MCP client creation
   - Response streaming
   - Connection reuse

## Error Handling

### Levels

1. **Protocol Errors**: MCP spec violations
2. **Application Errors**: Business logic failures
3. **System Errors**: Infrastructure issues

### Recovery Strategies

- Automatic retry with backoff
- Circuit breakers for failing servers
- Graceful degradation
- Error context propagation

## Testing Strategy

### Unit Tests

- Protocol message handling
- Configuration parsing
- State management
- LLM adapter logic

### Integration Tests

- MCP server communication
- LLM API calls
- Database operations
- End-to-end workflows

## Performance Characteristics

### Latency Sources

1. **LLM API calls**: 1-10 seconds
2. **MCP tool execution**: Variable (depends on tool)
3. **Database operations**: <10ms
4. **Network round-trips**: 10-100ms

### Optimization Techniques

- **Streaming**: Incremental response delivery
- **Caching**: Configuration and tool lists
- **Parallelism**: Concurrent tool calls
- **Connection pooling**: Reuse HTTP connections

## Monitoring and Observability

### Logging

**Levels**:
- `DEBUG`: Detailed trace
- `INFO`: Normal operations
- `WARN`: Recoverable issues
- `ERROR`: Failures

**Output**:
- Structured JSON (optional)
- Contextual information
- Request IDs for tracing

### Metrics (Future)

- Request latency
- Tool call success rate
- Session counts
- Error rates

## Summary

Nanobot's architecture prioritizes:
- **Modularity**: Clear component boundaries
- **Extensibility**: Easy to add providers/servers
- **Reliability**: Error handling and recovery
- **Performance**: Streaming and caching
- **Security**: Authentication and isolation

The design supports both development (easy testing) and production (robust deployment) use cases.
