# Nanobot Configuration Guide

## Configuration Overview

Nanobot uses YAML configuration files to define agents, MCP servers, flows, and other settings. This guide covers all configuration options in detail.

## Configuration File Structure

Basic structure of `nanobot.yaml`:

```yaml
# Optional: Authentication settings
auth:
  oauthClientId: "..."
  oauthClientSecret: "..."

# Optional: Extend other configuration files
extends:
  - "./base-config.yaml"

# Optional: Environment variable definitions
env:
  API_KEY:
    description: "API key for external service"
    optional: false

# Required: Agent definitions
agents:
  assistant:
    name: "My Assistant"
    model: "gpt-4.1"
    # ... more agent settings

# Required: MCP server connections
mcpServers:
  my-tools:
    url: "https://example.com/mcp"
    # ... more server settings

# Optional: Multi-step workflows
flows:
  my-workflow:
    # ... flow definition

# Optional: Reusable prompts
prompts:
  code-review:
    description: "Review code"
    template: "Review this code: {{code}}"

# Optional: Configuration profiles
profiles:
  production:
    # ... production-specific settings

# Optional: Publishing configuration
publish:
  entrypoint: "assistant"
  mcpServers:
    - "my-tools"
```

## Agents

Agents are AI personalities that can interact with users and use tools.

### Basic Agent

```yaml
agents:
  assistant:
    name: "General Assistant"
    description: "A helpful AI assistant"
    model: "gpt-4.1"
    instructions: |
      You are a helpful AI assistant.
      Always be polite and professional.
    mcpServers:
      - "general-tools"
```

### Agent Properties

#### Required Properties

| Property | Type | Description |
|----------|------|-------------|
| `model` | string | LLM model to use (e.g., `gpt-4.1`, `claude-3-opus-20240229`) |

#### Optional Properties

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `name` | string | Display name for the agent | Agent key name |
| `description` | string | Agent description for UI | "" |
| `instructions` | string | System prompt / instructions | "" |
| `icon` | string | Icon URL or data URI | null |
| `iconDark` | string | Dark mode icon | null |
| `starterMessages` | string[] | Suggested starting messages | [] |
| `mcpServers` | string[] | Connected MCP server names | [] |
| `chat` | boolean | Enable as chat agent | true |
| `temperature` | number | LLM temperature (0-2) | Provider default |
| `maxTokens` | number | Maximum tokens to generate | Provider default |
| `tools` | object[] | Additional tools | [] |

### Complete Agent Example

```yaml
agents:
  expert:
    name: "Domain Expert"
    description: "Specialized assistant for technical questions"
    icon: "data:image/svg+xml;base64,..."
    iconDark: "data:image/svg+xml;base64,..."
    model: "gpt-4.1"
    temperature: 0.7
    maxTokens: 2000
    
    instructions: |
      You are an expert technical assistant.
      
      Guidelines:
      - Provide accurate, detailed explanations
      - Use examples when helpful
      - Cite sources when possible
      - Ask clarifying questions
      
    starterMessages:
      - "Explain how TCP works"
      - "What are design patterns?"
      - "Help me debug this code"
    
    mcpServers:
      - "documentation"
      - "code-analysis"
      - "web-search"
    
    chat: true
```

### Dynamic Instructions

Instructions can include dynamic content:

```yaml
agents:
  timekeeper:
    instructions: |
      You are a helpful assistant.
      Current time: {{env.CURRENT_TIME}}
      User timezone: {{context.timezone}}
```

## MCP Servers

MCP servers provide tools, resources, and prompts to agents.

### HTTP/HTTPS Server

```yaml
mcpServers:
  remote-tools:
    url: "https://api.example.com/mcp"
    headers:
      Authorization: "Bearer ${env.API_KEY}"
      X-Custom-Header: "value"
```

### Stdio Server (Local Process)

```yaml
mcpServers:
  local-tools:
    command: "python"
    args:
      - "-m"
      - "mcp_server"
      - "--config"
      - "config.json"
    env:
      PYTHONPATH: "/path/to/modules"
      DEBUG: "1"
```

### MCP Server Properties

| Property | Type | Description | Required |
|----------|------|-------------|----------|
| `url` | string | HTTP(S) endpoint | For HTTP servers |
| `command` | string | Executable path/name | For stdio servers |
| `args` | string[] | Command arguments | No |
| `env` | map[string]string | Environment variables | No |
| `headers` | map[string]string | HTTP headers | No |

### OAuth-Protected Server

```yaml
mcpServers:
  oauth-api:
    url: "https://api.example.com/mcp"
    auth:
      oauthClientId: "${env.OAUTH_CLIENT_ID}"
      oauthClientSecret: "${env.OAUTH_CLIENT_SECRET}"
      oauthAuthorizeUrl: "https://auth.example.com/authorize"
      oauthScopes:
        - "read"
        - "write"
```

### Source-Based Server

Build from repository:

```yaml
mcpServers:
  custom-server:
    source:
      repo: "https://github.com/user/mcp-server"
      ref: "main"
      subdir: "server"
    command: "npm"
    args: ["start"]
```

## Flows

Flows define multi-step agent workflows.

### Basic Flow

```yaml
flows:
  research:
    input:
      fields:
        topic:
          description: "Research topic"
          required: true
    
    steps:
      - agent:
          name: "researcher"
        input: "Research: ${input.topic}"
        output: "research_data"
      
      - agent:
          name: "writer"
        input: "Write summary of: ${steps[0].output}"
```

### Flow Properties

| Property | Type | Description |
|----------|------|-------------|
| `input` | InputSchema | Input parameters |
| `steps` | Step[] | Sequential steps |
| `output` | string | Output expression |

### Step Types

#### Agent Step

```yaml
steps:
  - agent:
      name: "agent-name"
    input: "Input text or expression"
    output: "variable_name"
```

#### Tool Step

```yaml
steps:
  - tool:
      server: "server-name"
      tool: "tool-name"
    arguments:
      param1: "value"
      param2: "${input.field}"
    output: "result"
```

#### Conditional Step

```yaml
steps:
  - if: "${input.condition}"
    then:
      - agent:
          name: "agent-a"
        input: "..."
    else:
      - agent:
          name: "agent-b"
        input: "..."
```

#### Parallel Steps

```yaml
steps:
  - parallel:
      - agent: { name: "agent-1" }
        input: "Task 1"
      - agent: { name: "agent-2" }
        input: "Task 2"
    output: "combined_results"
```

### Expression Language

Access data in expressions:

- `${input.field}` - Input parameters
- `${steps[0].output}` - Previous step output
- `${env.VAR}` - Environment variables
- `${context.property}` - Context values

## Environment Variables

Define required environment variables:

```yaml
env:
  API_KEY:
    description: "API key for service"
    optional: false
    sensitive: true
  
  ENDPOINT_URL:
    description: "API endpoint"
    default: "https://api.example.com"
    optional: true
  
  LOG_LEVEL:
    description: "Logging level"
    options:
      - "debug"
      - "info"
      - "warn"
      - "error"
    default: "info"
```

### Environment Variable Properties

| Property | Type | Description | Default |
|----------|------|-------------|---------|
| `description` | string | Human-readable description | "" |
| `optional` | boolean | Whether variable is required | false |
| `sensitive` | boolean | Hide in UI/logs | false |
| `default` | string | Default value | "" |
| `options` | string[] | Allowed values | null |
| `useBearerToken` | boolean | Use as Bearer token | false |

## Prompts

Reusable prompt templates:

```yaml
prompts:
  code-review:
    description: "Review code for issues"
    input:
      code:
        description: "Code to review"
        required: true
      language:
        description: "Programming language"
        required: false
    template: |
      Review the following {{language}} code:
      
      ```
      {{code}}
      ```
      
      Provide feedback on:
      - Code quality
      - Potential bugs
      - Performance issues
      - Best practices
```

## Profiles

Environment-specific configurations:

```yaml
# Base configuration
agents:
  assistant:
    model: "gpt-4.1"

# Profile overrides
profiles:
  development:
    agents:
      assistant:
        model: "gpt-3.5-turbo"
  
  production:
    agents:
      assistant:
        model: "gpt-4.1"
        temperature: 0.5
```

**Usage**:
```bash
nanobot run --profile production config.yaml
```

## Publishing

Configure what to expose externally:

```yaml
publish:
  # Entrypoint agent(s)
  entrypoint:
    - "assistant"
    - "expert"
  
  # MCP servers to expose
  mcpServers:
    - "public-tools"
  
  # Flows to expose as tools
  flows:
    - "research"
```

## Configuration Inheritance

### Extends

Merge multiple configuration files:

```yaml
# base.yaml
agents:
  base-agent:
    model: "gpt-4.1"

# app.yaml
extends:
  - "./base.yaml"

agents:
  my-agent:
    model: "gpt-4.1"
```

Result: Both `base-agent` and `my-agent` are available.

### Merge Behavior

- **Arrays**: Concatenated
- **Objects**: Merged recursively
- **Primitives**: Override

## Command-Line Configuration

Override configuration via CLI:

```bash
nanobot run config.yaml \
  --default-model gpt-4.1 \
  --openai-api-key sk-... \
  --state ./custom.db \
  --listen-address 127.0.0.1:8080 \
  --profile production
```

### Available Flags

| Flag | Type | Description | Default |
|------|------|-------------|---------|
| `--default-model` | string | Default LLM model | gpt-4.1 |
| `--openai-api-key` | string | OpenAI API key | $OPENAI_API_KEY |
| `--openai-base-url` | string | Custom OpenAI endpoint | Official API |
| `--anthropic-api-key` | string | Anthropic API key | $ANTHROPIC_API_KEY |
| `--anthropic-base-url` | string | Custom Anthropic endpoint | Official API |
| `--state` | string | Database path | ./nanobot.db |
| `--listen-address` | string | Server address | 0.0.0.0:8080 |
| `--profile` | string | Configuration profile | default |
| `--debug` | boolean | Enable debug logging | false |
| `--quiet` | boolean | Minimal output | false |

## Environment File

Create `nanobot.env` for local configuration:

```bash
# LLM Providers
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Custom Configuration
API_KEY=abc123
ENDPOINT_URL=https://api.example.com

# Nanobot Settings
NANOBOT_DEFAULT_MODEL=gpt-4.1
NANOBOT_STATE=./data/nanobot.db
```

Load automatically or specify:
```bash
nanobot run --env-file ./custom.env config.yaml
```

## Validation

Validate configuration without running:

```bash
nanobot run --validate config.yaml
```

Common validation errors:
- Missing required fields
- Invalid MCP server URLs
- Circular extends
- Unknown agent references
- Invalid model names

## Best Practices

### 1. Security

```yaml
# ❌ Don't commit secrets
mcpServers:
  api:
    headers:
      Authorization: "Bearer sk-1234..."

# ✅ Use environment variables
mcpServers:
  api:
    headers:
      Authorization: "Bearer ${env.API_KEY}"
```

### 2. Organization

```yaml
# ✅ Use extends for shared configuration
extends:
  - "./common/agents.yaml"
  - "./common/servers.yaml"

agents:
  my-specific-agent:
    # ...
```

### 3. Defaults

```yaml
# ✅ Provide sensible defaults
env:
  LOG_LEVEL:
    default: "info"
    optional: true
```

### 4. Documentation

```yaml
# ✅ Document your configuration
agents:
  expert:
    description: |
      Specialized agent for technical support.
      
      Capabilities:
      - Code review
      - Debugging assistance
      - Architecture advice
    # ...
```

## Examples

### Example 1: Simple Chatbot

```yaml
agents:
  chatbot:
    name: "Friendly Chatbot"
    model: "gpt-4.1"
    instructions: "You are a friendly, helpful assistant."

mcpServers:
  # No external tools, just conversation
```

### Example 2: Research Assistant

```yaml
agents:
  researcher:
    name: "Research Assistant"
    model: "gpt-4.1"
    instructions: |
      You help users research topics.
      Use web search and read documents.
    mcpServers:
      - "web-search"
      - "documents"

mcpServers:
  web-search:
    url: "https://search-api.example.com/mcp"
  documents:
    url: "https://docs-api.example.com/mcp"
```

### Example 3: Multi-Agent Workflow

```yaml
agents:
  researcher:
    model: "gpt-4.1"
    instructions: "Research topics thoroughly"
  
  analyst:
    model: "gpt-4.1"
    instructions: "Analyze research data"
  
  writer:
    model: "gpt-4.1"
    instructions: "Write clear reports"

flows:
  full-report:
    input:
      fields:
        topic: { description: "Topic to research" }
    steps:
      - agent: { name: "researcher" }
        input: "${input.topic}"
        output: "research"
      - agent: { name: "analyst" }
        input: "Analyze: ${steps[0].output}"
        output: "analysis"
      - agent: { name: "writer" }
        input: "Write report on ${input.topic} using ${steps[1].output}"
```

## Troubleshooting

### Configuration Not Loading

1. Check YAML syntax
2. Verify file path
3. Check extends paths
4. Look for validation errors

### Agent Not Found

1. Verify agent name in configuration
2. Check profile selection
3. Ensure extends loaded correctly

### MCP Server Connection Failed

1. Test URL accessibility
2. Check authentication
3. Verify environment variables
4. Review server logs

### Environment Variables Not Set

1. Check `.env` file location
2. Verify variable names
3. Use `--env-file` flag
4. Check shell environment

## Reference

- [Examples](../examples/) - Real-world configurations
- [MCP Specification](https://modelcontextprotocol.io) - Protocol details
- [API Documentation](./API.md) - HTTP and MCP APIs
