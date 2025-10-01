# Nanobot Workflow Guide

## Complete Request Lifecycle

This document explains how requests flow through Nanobot from start to finish, covering all major workflows.

## Workflow 1: Starting Nanobot

### Step-by-Step Startup Process

```
1. User runs: nanobot run ./nanobot.yaml

2. Main entry point (main.go)
   └─> Initialize CLI (pkg/cli/root.go)
   
3. Load configuration
   ├─> Parse command-line flags
   ├─> Load environment variables from nanobot.env
   ├─> Read YAML configuration file
   ├─> Process "extends" to merge configs
   ├─> Apply profile-specific overrides
   └─> Validate final configuration

4. Initialize Runtime (pkg/runtime/runtime.go)
   ├─> Create LLM client configuration
   │   ├─> OpenAI setup (if API key present)
   │   └─> Anthropic setup (if API key present)
   ├─> Set max concurrency limits
   └─> Configure OAuth and token storage

5. Initialize Session Manager (pkg/session/manager.go)
   ├─> Connect to SQLite database (nanobot.db)
   ├─> Run migrations if needed
   └─> Prepare session cache

6. Start MCP Server (pkg/server/server.go)
   ├─> Register message handlers
   │   ├─> initialize
   │   ├─> tools/list
   │   ├─> tools/call
   │   ├─> prompts/list
   │   ├─> resources/list
   │   ├─> resources/subscribe
   │   └─> ... more handlers
   └─> Start session manager

7. Start HTTP API (pkg/api/handler.go)
   ├─> Setup routes
   │   ├─> GET /api/events/{thread_id}
   │   └─> GET /api/version
   ├─> Enable CORS
   └─> Listen on port 8080

8. Server ready
   └─> Web UI available at http://localhost:8080
```

## Workflow 2: Chat Interaction (Web UI)

### Complete Chat Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. USER SENDS MESSAGE                                       │
└─────────────────────────────────────────────────────────────┘
User types: "What's the weather in San Francisco?"
   │
   ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. UI SENDS REQUEST (ui/src/lib/chat.svelte.ts)            │
└─────────────────────────────────────────────────────────────┘
ChatService.send()
   ├─> Create EventSource for SSE
   │   GET /api/events/{thread_id}
   ├─> Send MCP message via tool call
   │   POST /api/mcp/ui
   │   {
   │     "method": "tools/call",
   │     "params": {
   │       "name": "nanobot.agentui.send",
   │       "arguments": {
   │         "prompt": "What's the weather..."
   │       }
   │     }
   │   }
   └─> Listen for SSE events
   
   ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. API RECEIVES REQUEST (pkg/api/events.go)                │
└─────────────────────────────────────────────────────────────┘
Events(rw, req)
   ├─> Extract thread_id from URL
   ├─> Create/resume session
   │   ├─> Load from database if exists
   │   └─> Create new if first message
   ├─> Create MCP client
   │   ├─> Connect to configured agent
   │   └─> Initialize MCP protocol
   └─> Start SSE stream
       ├─> Send history-start event
       ├─> Replay previous messages
       └─> Send history-end event
   
   ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. MCP CLIENT INITIALIZATION (pkg/mcp/client.go)           │
└─────────────────────────────────────────────────────────────┘
NewClient(ctx, serverName, config)
   ├─> Create wire connection
   │   ├─> HTTP: httpclient.go
   │   └─> Stdio: stdio.go
   ├─> Send initialize request
   │   {
   │     "method": "initialize",
   │     "params": {
   │       "protocolVersion": "2024-11-05",
   │       "clientInfo": {
   │         "name": "nanobot",
   │         "version": "..."
   │       },
   │       "capabilities": {
   │         "sampling": {},
   │         "elicitation": {}
   │       }
   │     }
   │   }
   ├─> Receive initialize result
   │   {
   │     "capabilities": {
   │       "tools": { "listChanged": true },
   │       "resources": { "subscribe": true }
   │     }
   │   }
   └─> Send initialized notification
   
   ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. SERVER HANDLES TOOL CALL (pkg/server/server.go)         │
└─────────────────────────────────────────────────────────────┘
handleCallTool(ctx, msg, payload)
   ├─> Parse tool name: "nanobot.agentui.send"
   ├─> Route to appropriate handler
   │   └─> AgentUI server (pkg/servers/agentui/)
   ├─> Load agent configuration
   │   {
   │     "name": "assistant",
   │     "model": "gpt-4.1",
   │     "mcpServers": ["weather-service"],
   │     "instructions": "You are a helpful..."
   │   }
   └─> Start agent execution
   
   ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. AGENT EXECUTION (pkg/agents/run.go)                     │
└─────────────────────────────────────────────────────────────┘
Run(ctx, input, options)
   ├─> Create MCP clients for each server
   │   └─> Connect to "weather-service"
   ├─> List available tools
   │   ├─> tools/list request to weather-service
   │   └─> Receive: [{ "name": "get_weather", ... }]
   ├─> Build LLM context
   │   ├─> System: agent instructions
   │   ├─> Tools: available MCP tools
   │   └─> History: previous messages
   └─> Call LLM
   
   ▼
┌─────────────────────────────────────────────────────────────┐
│ 7. LLM INTERACTION (pkg/llm/)                               │
└─────────────────────────────────────────────────────────────┘
LLMClient.Call(ctx, input)
   ├─> Format request for provider
   │   OpenAI:
   │   {
   │     "model": "gpt-4.1",
   │     "messages": [...],
   │     "tools": [{
   │       "type": "function",
   │       "function": {
   │         "name": "get_weather",
   │         "parameters": {...}
   │       }
   │     }],
   │     "stream": true
   │   }
   │
   ├─> Send HTTP request to LLM API
   ├─> Stream response chunks
   │   ├─> Text: "Let me check the weather..."
   │   └─> Tool call: {
   │         "name": "get_weather",
   │         "arguments": "{\"location\":\"San Francisco\"}"
   │       }
   └─> Return to agent
   
   ▼
┌─────────────────────────────────────────────────────────────┐
│ 8. TOOL EXECUTION VIA MCP (pkg/mcp/client.go)              │
└─────────────────────────────────────────────────────────────┘
CallTool(ctx, name, arguments)
   ├─> Send to MCP server
   │   {
   │     "method": "tools/call",
   │     "params": {
   │       "name": "get_weather",
   │       "arguments": {
   │         "location": "San Francisco"
   │       }
   │     }
   │   }
   ├─> MCP server executes
   │   └─> Calls actual weather API
   ├─> Receive result
   │   {
   │     "content": [{
   │       "type": "text",
   │       "text": "72°F, sunny"
   │     }]
   │   }
   └─> Return to agent
   
   ▼
┌─────────────────────────────────────────────────────────────┐
│ 9. AGENT CONTINUES (pkg/agents/run.go)                     │
└─────────────────────────────────────────────────────────────┘
   ├─> Inject tool result into LLM context
   ├─> Call LLM again
   │   Messages:
   │   - User: "What's the weather..."
   │   - Assistant: tool_call(get_weather)
   │   - Tool: "72°F, sunny"
   │
   ├─> LLM generates final response
   │   "The weather in San Francisco is currently
   │    72°F and sunny. It's a beautiful day!"
   │
   └─> Stream complete
   
   ▼
┌─────────────────────────────────────────────────────────────┐
│ 10. RESPONSE STREAMING (pkg/api/events.go)                 │
└─────────────────────────────────────────────────────────────┘
   ├─> Convert to SSE events
   │   
   │   event: message
   │   data: {"role":"assistant","items":[{"type":"text",
   │          "text":"Let me check..."}]}
   │   
   │   event: message  
   │   data: {"role":"assistant","items":[{"type":"tool",
   │          "name":"get_weather","arguments":"..."}]}
   │   
   │   event: message
   │   data: {"role":"assistant","items":[{"type":"text",
   │          "text":"The weather is 72°F and sunny."}]}
   │
   │   event: chat-done
   │   data: {}
   │
   └─> Save to session history
   
   ▼
┌─────────────────────────────────────────────────────────────┐
│ 11. UI UPDATES (ui/src/lib/chat.svelte.ts)                 │
└─────────────────────────────────────────────────────────────┘
EventSource.onmessage
   ├─> Parse event
   ├─> Update ChatService state
   │   ├─> Append to messages array
   │   ├─> Render incrementally
   │   └─> Show tool calls
   └─> Update UI reactively
   
   ▼
┌─────────────────────────────────────────────────────────────┐
│ 12. USER SEES RESPONSE                                      │
└─────────────────────────────────────────────────────────────┘
Display:
   Assistant: "The weather in San Francisco is currently
              72°F and sunny. It's a beautiful day!"
   
   [Tool used: get_weather]
```

## Workflow 3: CLI Tool Invocation

### Direct Tool Call Flow

```
1. User runs: nanobot call weather-service::get_weather --location "San Francisco"

2. CLI parses command (pkg/cli/call.go)
   ├─> Extract server ref: "weather-service"
   ├─> Extract tool name: "get_weather"
   └─> Parse arguments: {"location": "San Francisco"}

3. Load configuration
   ├─> Read nanobot.yaml
   └─> Find MCP server config

4. Create runtime (pkg/runtime/runtime.go)
   └─> Initialize without session

5. Call tool (runtime.CallFromCLI)
   ├─> Create temporary MCP client
   ├─> Connect to server
   ├─> Initialize protocol
   ├─> Call tool
   ├─> Wait for result
   └─> Close connection

6. Print result
   └─> JSON output to stdout
```

## Workflow 4: Resource Subscription

### Real-time Resource Updates

```
1. Agent requests resource subscription
   {
     "method": "resources/subscribe",
     "params": {
       "uri": "file:///logs/app.log"
     }
   }

2. MCP server handles subscription
   ├─> Register subscription
   ├─> Start monitoring resource
   └─> Return success

3. Resource changes
   └─> File is modified

4. MCP server sends notification
   {
     "method": "notifications/resources/updated",
     "params": {
       "uri": "file:///logs/app.log"
     }
   }

5. Nanobot receives notification
   ├─> Update session state
   └─> Trigger SSE event to UI

6. UI receives update
   ├─> Refresh resource display
   └─> Notify user
```

## Workflow 5: Multi-Step Flow Execution

### Flow with Multiple Agents

Example flow configuration:
```yaml
flows:
  research_report:
    input:
      fields:
        topic: { description: "Research topic" }
    steps:
      - agent:
          name: researcher
        input: "Research: ${input.topic}"
        output: research_data
      
      - agent:
          name: analyst
        input: "Analyze: ${steps[0].output}"
        output: analysis
      
      - agent:
          name: writer
        input: "Write report on ${input.topic} using ${steps[1].output}"
        output: final_report
```

**Execution Flow**:

```
1. User invokes flow
   └─> "Generate report on AI trends"

2. Flow executor starts (pkg/agents/run.go)
   ├─> Parse flow definition
   └─> Initialize step counter

3. Step 1: Research
   ├─> Load "researcher" agent
   ├─> Substitute: input.topic = "AI trends"
   ├─> Execute agent with context
   ├─> LLM generates research data
   └─> Store output in step[0].output

4. Step 2: Analysis
   ├─> Load "analyst" agent
   ├─> Substitute: steps[0].output (research data)
   ├─> Execute agent
   ├─> LLM analyzes research
   └─> Store output in step[1].output

5. Step 3: Report Writing
   ├─> Load "writer" agent
   ├─> Substitute: input.topic + steps[1].output
   ├─> Execute agent
   ├─> LLM writes final report
   └─> Return as flow result

6. Stream all intermediate results
   └─> UI shows progress through each step
```

## Workflow 6: Session Management

### Session Lifecycle

```
1. First message in conversation
   ├─> Generate session ID (UUID)
   ├─> Create session record
   │   {
   │     "sessionID": "abc-123...",
   │     "accountID": "user@example.com",
   │     "description": "",
   │     "config": {...},
   │     "created": "2024-01-01T00:00:00Z"
   │   }
   └─> Save to database

2. Subsequent messages
   ├─> Load session by ID
   ├─> Resume MCP connections
   └─> Access conversation history

3. Session updates
   ├─> After each message
   ├─> Update description (auto-generated)
   └─> Save configuration changes

4. Session closure
   ├─> User closes browser/app
   ├─> MCP connections cleaned up
   └─> Session persists in database

5. Session deletion
   ├─> User deletes conversation
   └─> Remove from database
```

## Workflow 7: OAuth Authentication

### OAuth Flow for MCP Server

```
1. MCP server requires OAuth
   └─> Configuration:
       {
         "url": "https://api.example.com/mcp",
         "auth": {
           "oauthClientId": "...",
           "oauthAuthorizeUrl": "..."
         }
       }

2. First connection attempt
   ├─> No token available
   └─> Return authorization URL

3. User clicks auth link
   ├─> Redirect to OAuth provider
   ├─> User authorizes
   └─> Callback to Nanobot
       GET /oauth/callback?code=...&state=...

4. Token exchange (pkg/mcp/oauth.go)
   ├─> Exchange code for token
   ├─> Store encrypted token
   └─> Associate with session

5. Retry connection
   ├─> Load token from storage
   ├─> Add Authorization header
   └─> Connect successfully

6. Token refresh
   ├─> Detect expired token (401)
   ├─> Use refresh token
   ├─> Update stored token
   └─> Retry request
```

## Workflow 8: Error Handling

### Error Recovery Flow

```
1. Error occurs in MCP tool call
   └─> MCP server returns error
       {
         "error": {
           "code": -32000,
           "message": "Tool execution failed"
         }
       }

2. Agent receives error
   ├─> Log error with context
   └─> Return error to LLM

3. LLM handles error
   ├─> Receives error in tool result
   ├─> Generates user-friendly message
   └─> May suggest alternatives

4. UI displays error
   ├─> Show error message
   ├─> Offer retry option
   └─> Log for debugging

5. Recovery options
   ├─> User retries
   ├─> User reformulates question
   └─> Admin checks MCP server logs
```

## Performance Characteristics

### Typical Latencies

1. **Initial Connection**: 100-500ms
   - MCP handshake
   - Tool discovery
   - Session setup

2. **Simple Query**: 1-3 seconds
   - LLM call without tools
   - Streaming response

3. **Tool Call**: 2-10 seconds
   - LLM decides to use tool
   - MCP tool execution
   - LLM processes result

4. **Complex Flow**: 10-60 seconds
   - Multiple agent steps
   - Multiple tool calls
   - Large context processing

### Optimization Tips

1. **Connection Pooling**: Reuse MCP clients
2. **Caching**: Cache tool lists and prompts
3. **Streaming**: Use streaming for immediate feedback
4. **Concurrency**: Parallel tool calls when possible
5. **Lazy Loading**: Connect to MCP servers on demand

## Summary

Understanding these workflows helps with:
- **Debugging**: Trace issues through the system
- **Optimization**: Identify bottlenecks
- **Development**: Add new features correctly
- **Operations**: Monitor and maintain deployments

Each workflow builds on the architecture and uses the components described in other documentation files.
