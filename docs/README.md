# Nanobot Documentation

Welcome to the comprehensive documentation for Nanobot - a standalone MCP Host for building AI agents with MCP and MCP-UI support.

## Documentation Overview

This documentation is designed for first-time users and developers who want to understand and work with Nanobot. It covers everything from basic concepts to detailed technical implementation.

## Getting Started

If you're new to Nanobot, start here:

1. **[Overview](./OVERVIEW.md)** - Introduction to Nanobot, core concepts, and what makes it unique
2. **[Quick Start](#quick-start-guide)** - Get up and running in minutes
3. **[Configuration Guide](./CONFIGURATION.md)** - Learn how to configure agents, MCP servers, and more

## Documentation Index

### For Users

- **[Overview](./OVERVIEW.md)** - Complete introduction to Nanobot
  - What is Nanobot?
  - Core concepts (Agents, MCP Servers, Flows)
  - Key features and use cases
  - Getting started guide

- **[Quick Reference](./QUICK_REFERENCE.md)** - Essential commands and snippets
  - Common commands
  - Configuration templates
  - Code examples
  - Troubleshooting tips

- **[Glossary](./GLOSSARY.md)** - Terminology and definitions
  - All key terms explained
  - Acronyms reference
  - Common confusion clarified

- **[Configuration Guide](./CONFIGURATION.md)** - Comprehensive configuration reference
  - Agent configuration
  - MCP server setup (HTTP, stdio, OAuth)
  - Flows and multi-step workflows
  - Environment variables
  - Profiles and inheritance
  - Best practices and examples

- **[API Documentation](./API.md)** - HTTP and MCP API reference
  - REST API endpoints
  - Server-Sent Events (SSE)
  - MCP protocol methods
  - Data types and structures
  - Error handling
  - Example integrations

### For Developers

- **[Architecture](./ARCHITECTURE.md)** - System design and components
  - System architecture overview
  - Core components (MCP, Server, Runtime, Agents)
  - Data flow and communication patterns
  - State management
  - Security and scalability

- **[Workflow Guide](./WORKFLOW.md)** - Complete request lifecycles
  - Startup process
  - Chat interaction flow
  - CLI tool invocation
  - Resource subscriptions
  - Multi-step flows
  - Session management
  - OAuth authentication
  - Error handling

- **[Development Guide](./DEVELOPMENT.md)** - Contributing to Nanobot
  - Development setup
  - Project structure
  - Key packages explained
  - Development workflow
  - Testing strategies
  - Code style guidelines
  - Building and debugging
  - Contributing process

## Quick Start Guide

### Installation

#### Homebrew (macOS/Linux)
```bash
brew install nanobot-ai/tap/nanobot
```

#### Build from Source
```bash
git clone https://github.com/nanobot-ai/nanobot.git
cd nanobot
make
```

### Basic Configuration

Create `nanobot.yaml`:

```yaml
agents:
  assistant:
    name: "My Assistant"
    model: "gpt-4.1"
    instructions: |
      You are a helpful AI assistant.
      Be concise and friendly.
    mcpServers:
      - "example-tools"

mcpServers:
  example-tools:
    url: "https://example.com/mcp"
```

### Set API Key

```bash
# For OpenAI
export OPENAI_API_KEY=sk-...

# For Anthropic
export ANTHROPIC_API_KEY=sk-ant-...
```

### Run

```bash
nanobot run nanobot.yaml
```

Access the UI at: http://localhost:8080

## Key Concepts

### MCP Host

Nanobot is an **MCP Host** - the service that:
- Connects to MCP servers (tools/resources)
- Integrates with LLM providers
- Manages conversation state
- Provides user interfaces

### Agents

**Agents** are AI personalities configured with:
- Instructions (system prompts)
- LLM model selection
- Connected MCP servers
- UI customization

### MCP Servers

**MCP Servers** provide capabilities:
- **Tools**: Functions agents can execute
- **Resources**: Data sources agents can read
- **Prompts**: Reusable templates

### Flows

**Flows** enable multi-step workflows:
- Chain multiple agents
- Conditional logic
- Parallel execution
- Data transformation

## Architecture Overview

```
┌─────────────────────────────────────────┐
│          User Interfaces                │
│    (Web UI, CLI, HTTP API)              │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│         Nanobot Host                    │
│  ┌──────────┐  ┌──────────┐           │
│  │ Session  │  │ Runtime  │            │
│  │ Manager  │  │ Manager  │            │
│  └──────────┘  └──────────┘            │
└──────────────┬──────────────────────────┘
               │
      ┌────────┼────────┐
      │        │        │
┌─────▼──┐ ┌──▼────┐ ┌─▼──────┐
│MCP Svr1│ │MCP Sr2│ │LLM API │
│(Tools) │ │(Data) │ │(GPT-4) │
└────────┘ └───────┘ └────────┘
```

## Common Use Cases

### 1. Custom Chatbot

Create specialized AI chatbots with domain-specific tools:

```yaml
agents:
  support:
    name: "Support Bot"
    model: "gpt-4.1"
    instructions: "Help users with technical issues"
    mcpServers:
      - "ticketing-system"
      - "knowledge-base"
```

### 2. Data Analysis

Build agents that query and analyze data:

```yaml
agents:
  analyst:
    model: "gpt-4.1"
    instructions: "Analyze data and provide insights"
    mcpServers:
      - "database-connector"
      - "visualization-tools"
```

### 3. Multi-Agent Pipeline

Create complex workflows with multiple specialists:

```yaml
flows:
  content-pipeline:
    steps:
      - agent: { name: "researcher" }
        input: "Research ${input.topic}"
      - agent: { name: "writer" }
        input: "Write about ${steps[0].output}"
      - agent: { name: "editor" }
        input: "Edit ${steps[1].output}"
```

## Configuration Examples

See the [examples/](../examples/) directory for real-world configurations:

- **[blackjack.yaml](../examples/blackjack.yaml)** - Interactive game agent
- **[shopping.yaml](../examples/shopping.yaml)** - E-commerce assistant
- **[huggingface.yaml](../examples/huggingface.yaml)** - Model interaction

## API Quick Reference

### SSE Events Stream

```javascript
const eventSource = new EventSource('/api/events/thread-id');

eventSource.addEventListener('message', (e) => {
  const msg = JSON.parse(e.data);
  console.log('Message:', msg);
});
```

### MCP Tool Call

```javascript
fetch('/mcp/ui', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    method: 'tools/call',
    params: {
      name: 'nanobot.agentui.send',
      arguments: { prompt: 'Hello!' }
    }
  })
});
```

## Development Quick Reference

### Project Structure

```
nanobot/
├── pkg/               # Go packages
│   ├── agents/       # Agent execution
│   ├── api/          # HTTP API
│   ├── mcp/          # MCP protocol
│   ├── llm/          # LLM providers
│   └── ...
├── ui/               # Web UI (Svelte)
└── examples/         # Example configs
```

### Build and Run

```bash
# Build
make

# Run with example
./bin/nanobot run examples/random.yaml

# Run tests
go test ./...

# UI development
cd ui && npm run dev
```

## Troubleshooting

### Common Issues

**Connection Refused**
- Check MCP server is running
- Verify URL in configuration
- Test network connectivity

**Authentication Failed**
- Verify API keys set correctly
- Check OAuth configuration
- Review token expiration

**UI Not Loading**
- Ensure port 8080 is available
- Check for build errors
- Clear browser cache

**Database Locked**
- Close other Nanobot instances
- Delete `.db-*` lock files

### Debug Mode

```bash
nanobot run --debug config.yaml
```

### Trace MCP Messages

```bash
nanobot run --trace config.yaml
```

## Support and Community

### Getting Help

- **Documentation**: Read these guides thoroughly
- **GitHub Issues**: Report bugs and request features
- **Examples**: Study example configurations
- **Source Code**: Review implementation details

### Contributing

We welcome contributions! See [Development Guide](./DEVELOPMENT.md) for:
- Development setup
- Code style guidelines
- Testing requirements
- Pull request process

### Resources

- **Website**: https://nanobot.ai
- **GitHub**: https://github.com/nanobot-ai/nanobot
- **MCP Spec**: https://modelcontextprotocol.io
- **License**: Apache 2.0

## Documentation Roadmap

Future documentation planned:

- [ ] Video tutorials
- [ ] Interactive examples
- [ ] Advanced patterns guide
- [ ] Deployment best practices
- [ ] Performance tuning guide
- [ ] Security hardening guide
- [ ] Migration guides

## Version History

- **v0.1.0** (Alpha) - Initial release
  - Core MCP implementation
  - Basic agent support
  - Web UI
  - OpenAI/Anthropic integration

## Feedback

Documentation feedback is appreciated! Please:
- Open issues for corrections
- Suggest improvements
- Share your use cases
- Contribute examples

## Document Summary

| Document | Purpose | Length | Audience |
|----------|---------|--------|----------|
| [README](./README.md) | Documentation hub | Short | Everyone |
| [Overview](./OVERVIEW.md) | Introduction and concepts | Medium | New users |
| [Quick Reference](./QUICK_REFERENCE.md) | Commands and snippets | Medium | Daily users |
| [Glossary](./GLOSSARY.md) | Terminology definitions | Medium | New users |
| [Configuration](./CONFIGURATION.md) | Complete config reference | Long | Users/Admins |
| [API](./API.md) | HTTP and MCP API reference | Long | Developers |
| [Architecture](./ARCHITECTURE.md) | System design | Long | Developers |
| [Workflow](./WORKFLOW.md) | Request lifecycles | Long | Developers |
| [Development](./DEVELOPMENT.md) | Contributing guide | Long | Contributors |

## Next Steps

### For First-Time Users
1. Read the [Overview](./OVERVIEW.md) for a complete introduction
2. Check the [Glossary](./GLOSSARY.md) if terms are confusing
3. Follow the Quick Start above to get running
4. Explore [Examples](../examples/) for inspiration

### For Regular Users
1. Bookmark [Quick Reference](./QUICK_REFERENCE.md) for daily use
2. Study [Configuration Guide](./CONFIGURATION.md) for your use case
3. Check [API Documentation](./API.md) for integration needs

### For Developers
1. Understand the [Architecture](./ARCHITECTURE.md)
2. Learn [Workflow](./WORKFLOW.md) patterns
3. Follow [Development Guide](./DEVELOPMENT.md) to contribute

---

**Welcome to Nanobot!** We're excited to see what you build. 🚀
