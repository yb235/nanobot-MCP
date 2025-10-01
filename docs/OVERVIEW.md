# Nanobot-MCP: Complete Overview

## Introduction

Nanobot is a comprehensive MCP (Model Context Protocol) Host implementation that enables building and deploying AI agents with MCP and MCP-UI support. It serves as a standalone, open-source platform that bridges MCP servers with Large Language Models (LLMs) to create powerful agent experiences.

## What is Nanobot?

Nanobot is an **MCP Host** - the critical service that:
- Connects to one or more MCP servers
- Integrates with LLM providers (OpenAI, Anthropic)
- Manages conversation context and state
- Provides a user interface (web-based chat)
- Handles authentication and session management
- Exposes agents through various interfaces (UI, API, MCP protocol)

### Key Differentiators

Unlike embedded MCP hosts in applications like VSCode, Claude, or ChatGPT, Nanobot is:
- **Standalone**: Can be deployed independently
- **Flexible**: Easily configurable for different use cases
- **Extensible**: Supports custom agents, flows, and integrations
- **Open Source**: Apache 2.0 licensed

## Core Concepts

### 1. MCP Host Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     User Interface                       │
│              (Web UI, CLI, API, MCP)                    │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                   Nanobot Host                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │   Session    │  │   Runtime    │  │     LLM      │ │
│  │  Management  │  │   Manager    │  │   Provider   │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
└──────────────────────┬──────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
┌───────▼──────┐ ┌────▼─────┐ ┌─────▼──────┐
│ MCP Server 1 │ │MCP Svr 2│ │MCP Server 3│
│   (Tools)    │ │(Prompts)│ │(Resources) │
└──────────────┘ └──────────┘ └────────────┘
```

### 2. Agents

**Agents** are configured AI personalities with specific:
- **Instructions**: System prompts defining behavior
- **Model**: LLM to use (e.g., gpt-4.1, claude-3-opus)
- **MCP Servers**: Connected tools/resources
- **UI Properties**: Name, description, icon, starter messages
- **Context**: Environment variables and configuration

### 3. MCP Servers

**MCP Servers** provide capabilities to agents:
- **Tools**: Functions agents can call (e.g., search, file operations)
- **Resources**: Data sources agents can read (e.g., files, databases)
- **Prompts**: Reusable prompt templates
- **Connection Types**: HTTP/HTTPS or stdio (local processes)

### 4. Flows

**Flows** are multi-step agent workflows:
- Chain multiple agent calls
- Transform data between steps
- Implement complex logic
- Can be exposed as tools to other agents

### 5. Sessions

**Sessions** manage conversation state:
- Persist conversation history
- Store session-specific attributes
- Handle authentication context
- Maintain MCP client connections

## Key Features

### MCP Protocol Support

Nanobot implements the full MCP specification:
- ✅ **Tools**: Call functions, structured inputs/outputs
- ✅ **Resources**: Read data sources, subscriptions
- ✅ **Prompts**: Template-based interactions
- ✅ **Sampling**: LLM delegation (experimental)
- ✅ **Elicitation**: User input requests (MCP-UI)
- ✅ **Logging**: Server-side logging
- ✅ **Session Management**: State persistence

### LLM Provider Support

- **OpenAI**: GPT-4, GPT-4.1, GPT-3.5, etc.
- **Anthropic**: Claude 3 (Opus, Sonnet, Haiku)
- Automatic provider selection based on model name
- Custom API endpoints and headers support

### User Interfaces

1. **Web UI** (Svelte-based)
   - Real-time chat interface
   - File uploads and attachments
   - Resource browsing
   - Agent selection
   - Multi-session support

2. **CLI** (Command-line)
   - Direct tool invocation
   - Session management
   - Configuration testing

3. **HTTP API**
   - Server-Sent Events (SSE) for real-time updates
   - RESTful endpoints
   - MCP proxy for external clients

### Deployment Options

- **Local Development**: Single binary with embedded UI
- **Docker**: Containerized deployment
- **Homebrew**: Easy installation on macOS/Linux
- **Build from Source**: Full customization

## Use Cases

### 1. Custom AI Chatbots
Create specialized chatbots with domain-specific tools:
- Customer support with CRM integration
- Data analysis with database access
- Content creation with API integrations

### 2. Multi-Agent Systems
Build complex workflows combining multiple agents:
- Research → Analysis → Report generation
- Data gathering → Processing → Visualization
- Code generation → Testing → Documentation

### 3. MCP Server Development
Test and debug MCP servers:
- Interactive tool testing
- Resource inspection
- Prompt development

### 4. Integration Platform
Bridge various systems through MCP:
- Legacy systems integration
- API aggregation
- Protocol translation

## Architecture Principles

### 1. Modularity
Each component has a clear responsibility:
- **pkg/mcp**: MCP protocol implementation
- **pkg/llm**: LLM provider abstractions
- **pkg/agents**: Agent execution logic
- **pkg/session**: State management
- **pkg/api**: HTTP API layer
- **ui**: Frontend application

### 2. Extensibility
Easy to extend with:
- Custom MCP servers
- New LLM providers
- Additional UI components
- Custom authentication

### 3. Configuration-Driven
Everything configurable via YAML:
- No code changes for new agents
- Environment-based configuration
- Profile support for different deployments

### 4. Production-Ready
Built for real-world use:
- Database-backed persistence (SQLite)
- OAuth 2.0 support
- Session encryption
- CORS handling
- Error recovery

## Getting Started

### Quick Start

1. **Install**:
   ```bash
   brew install nanobot-ai/tap/nanobot
   ```

2. **Configure** (`nanobot.yaml`):
   ```yaml
   agents:
     assistant:
       name: My Assistant
       model: gpt-4.1
       mcpServers: my-tools
   
   mcpServers:
     my-tools:
       url: https://example.com/mcp
   ```

3. **Set API Key**:
   ```bash
   export OPENAI_API_KEY=sk-...
   ```

4. **Run**:
   ```bash
   nanobot run ./nanobot.yaml
   ```

5. **Access**: http://localhost:8080

### Next Steps

- Explore [Architecture](./ARCHITECTURE.md) for system design
- Read [Configuration Guide](./CONFIGURATION.md) for detailed setup
- Check [API Documentation](./API.md) for integration
- Review [Development Guide](./DEVELOPMENT.md) for contributing

## Project Status

Nanobot is in **active development** (alpha stage):
- Core functionality is stable
- API may change between versions
- Production use requires testing
- Community contributions welcome

## Resources

- **Website**: https://nanobot.ai
- **GitHub**: https://github.com/nanobot-ai/nanobot
- **Examples**: See `examples/` directory
- **License**: Apache 2.0

## Summary

Nanobot provides a complete, production-ready MCP Host implementation that makes it easy to build, deploy, and manage AI agents with MCP integration. Whether you're building a simple chatbot or a complex multi-agent system, Nanobot provides the infrastructure you need.
