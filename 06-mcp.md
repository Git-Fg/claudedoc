# MCP Servers

> **Sources:**
> - [Model Context Protocol Docs](https://modelcontextprotocol.io/docs) (official)
> - [Claude Code MCP Integration](https://code.claude.com/docs/en/mcp) (official)
> - [MCP Protocol Specification](https://modelcontextprotocol.io/specification/2024-11-05) (official)
> - [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) (official, v2 in development)

---

## How to test MCP ? 

# Basic usage
npx @modelcontextprotocol/inspector --cli node build/index.js

# With config file
npx @modelcontextprotocol/inspector --cli --config path/to/config.json --server myserver

# List available tools
npx @modelcontextprotocol/inspector --cli node build/index.js --method tools/list

# Call a specific tool
npx @modelcontextprotocol/inspector --cli node build/index.js --method tools/call --tool-name mytool --tool-arg key=value --tool-arg another=value2

# Call a tool with JSON arguments
npx @modelcontextprotocol/inspector --cli node build/index.js --method tools/call --tool-name mytool --tool-arg 'options={"format": "json", "max_tokens": 100}'

# List available resources
npx @modelcontextprotocol/inspector --cli node build/index.js --method resources/list

# List available prompts
npx @modelcontextprotocol/inspector --cli node build/index.js --method prompts/list

# Connect to a remote MCP server (default is SSE transport)
npx @modelcontextprotocol/inspector --cli https://my-mcp-server.example.com

# Connect to a remote MCP server (with Streamable HTTP transport)
npx @modelcontextprotocol/inspector --cli https://my-mcp-server.example.com --transport http --method tools/list

# Connect to a remote MCP server (with custom headers)
npx @modelcontextprotocol/inspector --cli https://my-mcp-server.example.com --transport http --method tools/list --header "X-API-Key: your-api-key"

# Call a tool on a remote server
npx @modelcontextprotocol/inspector --cli https://my-mcp-server.example.com --method tools/call --tool-name remotetool --tool-arg param=value

# List resources from a remote server
npx @modelcontextprotocol/inspector --cli https://my-mcp-server.example.com --method resources/list

## What is MCP?

**MCP = Model Context Protocol**

MCP is an open protocol that enables Claude Code to connect to external tools, resources, and prompts through standardized servers.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **MCP Server** | A server that exposes tools, resources, and prompts via the MCP protocol |
| **MCP Client** | Claude Code acts as an MCP client, connecting to servers |
| **Tools** | External tools that Claude Code can invoke through the server |
| **Resources** | Data sources that Claude Code can read (files, databases, etc.) |
| **Prompts** | Pre-defined prompt templates exposed by the server |

### Why MCP Matters

| For... | Benefit |
|--------|---------|
| **Developers** | Reduces time building AI integrations |
| **AI Applications** | Access to ecosystem of data sources and tools |
| **End-users** | More capable AI that can access data and take actions |

---

## MCP Architecture

```
┌─────────────────────────────────────┐
│         Claude Code (Client)        │
└──────────────┬──────────────────────┘
               │
               │ MCP Protocol
               ▼
┌─────────────────────────────────────┐
│          MCP Server 1               │
│  - Tools: filesystem, git, etc.     │
│  - Resources: file contents         │
│  - Prompts: pre-defined templates   │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│         External Systems            │
│  - APIs, databases, services        │
└─────────────────────────────────────┘
```

### Transport Types

| Transport | Description | Use Case |
|-----------|-------------|----------|
| **stdio** | Local process communication | Local tools, custom scripts |
| **Streamable HTTP** | HTTP POST + optional SSE streaming | Cloud APIs, remote services (recommended) |
| **SSE** | Server-Sent Events | Streaming responses (deprecated) |

---

## Installing MCP Servers

### Option 1: Streamable HTTP Server (Recommended)

```bash
# Basic syntax
claude mcp add --transport http <name> <url>

# Real example: Connect to Notion
claude mcp add --transport http notion https://mcp.notion.com/mcp

# With Bearer token
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

### Option 2: Stdio Server (Local)

```bash
# Basic syntax
claude mcp add [options] <name> -- <command> [args...]

# Example: Add filesystem server
claude mcp add --transport stdio --env API_KEY=YOUR_KEY filesystem \
  -- npx -y @modelcontextprotocol/server-filesystem

# Example: Add git server
claude mcp add --transport stdio git \
  -- npx -y @modelcontextprotocol/server-git
```

### Option 3: SSE Server (Deprecated)

```bash
# Basic syntax
claude mcp add --transport sse <name> <url>

# Example (not recommended)
claude mcp add --transport sse asana https://mcp.asana.com/sse
```

### Managing Servers

```bash
# List all configured servers
claude mcp list

# Get details for a specific server
claude mcp get github

# Remove a server
claude mcp remove github

# Within Claude Code: Check server status
/mcp
```

---

## MCP Installation Scopes

### Local Scope

Local-scoped servers are stored in `~/.claude.json`. They remain private to you and are only accessible in the current project.

```bash
# Add a local-scoped server (default)
claude mcp add --transport http stripe https://mcp.stripe.com

# Explicitly specify local scope
claude mcp add --transport http stripe --scope local https://mcp.stripe.com
```

### Project Scope

Project-scoped servers are stored in `.mcp.json` at your project root. They are designed to be checked into version control.

```bash
# Add a project-scoped server
claude mcp add --transport http paypal --scope project https://mcp.paypal.com/mcp
```

The resulting `.mcp.json` format:

```json
{
  "mcpServers": {
    "shared-server": {
      "command": "/path/to/server",
      "args": [],
      "env": {}
    }
  }
}
```

### User Scope

User-scoped servers are stored in `~/.claude.json` and provide cross-project accessibility.

```bash
# Add a user server
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic
```

### Choosing the Right Scope

| Scope | Use When |
|-------|----------|
| **Local** | Personal servers, experimental configs, sensitive credentials |
| **Project** | Team-shared servers, project-specific tools |
| **User** | Personal utilities needed across multiple projects |

---

## Tool Naming Convention

MCP tools follow the pattern: `mcp__<server>__<tool>`

| Example | Meaning |
|---------|---------|
| `mcp__memory__create_entities` | Memory server's create entities tool |
| `mcp__filesystem__read_file` | Filesystem server's read file tool |
| `mcp__github__search_repositories` | GitHub server's search tool |

### Matching MCP Tools in Hooks

Use regex patterns to target MCP tools:

| Pattern | Matches |
|---------|---------|
| `mcp__memory__.*` | All tools from memory server |
| `mcp__.*__write.*` | Any tool containing "write" from any server |
| `mcp__github__.*__search` | Search tools from GitHub server |

---

## Practical Examples

### Example 1: Connect to GitHub

```bash
# 1. Add the GitHub MCP server
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# 2. In Claude Code, authenticate if needed
> /mcp
# Select "Authenticate" for GitHub

# 3. Now you can ask Claude to work with GitHub
> "Review PR #456 and suggest improvements"
> "Create a new issue for the bug we just found"
> "Show me all open PRs assigned to me"
```

### Example 2: Query PostgreSQL Database

```bash
# 1. Add the database server with your connection string
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:pass@prod.db.com:5432/analytics"

# 2. Query your database naturally
> "What's our total revenue this month?"
> "Show me the schema for the orders table"
> "Find customers who haven't made a purchase in 90 days"
```

### Example 3: Monitor Errors with Sentry

```bash
# 1. Add the Sentry MCP server
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. In Claude Code, authenticate
> /mcp

# 3. Debug production issues
> "What are the most common errors in the last 24 hours?"
> "Show me the stack trace for error ID abc123"
> "Which deployment introduced these new errors?"
```

---

## Authentication with Remote MCP Servers

Many cloud-based MCP servers require OAuth 2.0 authentication.

```bash
# 1. Add the server that requires authentication
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. Use the /mcp command within Claude Code
> /mcp

# 3. Follow the steps in your browser to login
```

Authentication tokens are stored securely and refreshed automatically.

---

## Add MCP Servers from JSON

```bash
# Basic syntax
claude mcp add-json <name> '<json>'

# Example: Adding an HTTP server with JSON configuration
claude mcp add-json weather-api '{"type":"http","url":"https://api.weather.com/mcp","headers":{"Authorization":"Bearer token"}}'

# Example: Adding a stdio server with JSON configuration
claude mcp add-json local-weather '{"type":"stdio","command":"/path/to/weather-cli","args":["--api-key","abc123"],"env":{"CACHE_DIR":"/tmp"}}'
```

---

## Import MCP Servers from Claude Desktop

```bash
# Import servers from Claude Desktop
claude mcp add-from-claude-desktop
```

Then select which servers to import from the interactive dialog.

---

## Use Claude Code as an MCP Server

You can expose Claude Code's tools as an MCP server:

```bash
# Start Claude as a stdio MCP server
claude mcp serve
```

Add to Claude Desktop configuration:

```json
{
  "mcpServers": {
    "claude-code": {
      "type": "stdio",
      "command": "claude",
      "args": ["mcp", "serve"],
      "env": {}
    }
  }
}
```

---

## MCP Tool Search

When you have many MCP servers, tool definitions can consume significant context. MCP Tool Search dynamically loads tools on-demand.

### How It Works

1. MCP tools are deferred rather than preloaded into context
2. Claude uses a search tool to discover relevant MCP tools when needed
3. Only the tools Claude actually needs are loaded

### Configure Tool Search

```bash
# Use a custom 5% threshold
ENABLE_TOOL_SEARCH=auto:5 claude

# Disable tool search entirely
ENABLE_TOOL_SEARCH=false claude

# Always enable
ENABLE_TOOL_SEARCH=true claude
```

---

## MCP Resources

MCP servers can expose resources that you can reference using `@` mentions:

```bash
# Reference an MCP resource
> Can you analyze @github:issue://123 and suggest a fix?

> Please review the API documentation at @docs:file://api/authentication
```

---

## MCP Prompts as Commands

MCP servers can expose prompts that become available as commands:

```bash
# Execute MCP prompt without arguments
> /mcp__github__list_prs

# Execute MCP prompt with arguments
> /mcp__github__pr_review 456

> /mcp__jira__create_issue "Bug in login flow" high
```

---

## Environment Variable Expansion in `.mcp.json`

Claude Code supports environment variable expansion:

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

Syntax:
- `${VAR}` - Expands to environment variable `VAR`
- `${VAR:-default}` - Uses `default` if `VAR` is not set

---

## Popular MCP Servers

| Server | Description | Install Command |
|--------|-------------|-----------------|
| filesystem | Local file access | `npx -y @modelcontextprotocol/server-filesystem` |
| git | Git operations | `npx -y @modelcontextprotocol/server-git` |
| github | GitHub API integration | `claude mcp add github https://api.githubcopilot.com/mcp/` |
| memory | Knowledge graph storage | `npx -y @modelcontextprotocol/server-memory` |
| sentry | Error monitoring | `claude mcp add sentry https://mcp.sentry.dev/mcp` |
| postgres | PostgreSQL database | `npx -y @modelcontextprotocol/server-postgres` |

---

## Database & Analytics Integration

**[COMMUNITY] CLI-Based Data Analytics**

**Source**: Boris Cherny (Claude Code creator) team practices

The Claude Code team uses database CLI tools and MCP servers for analytics queries directly in Claude Code. This pattern works for any database with a CLI, MCP, or API.

**Pattern: BigQuery Skill**

The team has a BigQuery skill checked into the codebase:

```markdown
Ask Claude Code to use the "bq" CLI to pull and analyze metrics on the fly.
```

**Example Queries**:
```bash
> Show me user signups by week for the last 30 days using BigQuery

> What's the error rate trend for API endpoints? Use the bq skill.

> Pull revenue metrics for Q4 and visualize them.
```

**Why This Works**:
- Zero SQL writing required
- Claude handles query generation and optimization
- Results analyzed in context of your codebase
- Metrics drive decisions immediately

**Works With**:
| Database | Method | Command Example |
|:---------|:-------|:----------------|
| **BigQuery** | CLI (bq) | `bq query "SELECT ..."` |
| **PostgreSQL** | MCP server | `@postgres:query` |
| **MySQL** | CLI | `mysql -e "SELECT ..."` |
| **MongoDB** | MCP server | `@mongodb:aggregate` |
| **Snowflake** | CLI | `snowsql -q "SELECT ..."` |
| **Any REST API** | WebFetch | Standard API calls |

**Setup**:
1. Configure database CLI in your environment
2. Set up MCP server if available
3. Create a skill that wraps common queries
4. Reference database schemas in skill documentation

**Key**: The team reports not writing SQL manually for 6+ months. Claude handles query generation, optimization, and results interpretation.

---

## Security Considerations

<Warning>
Use third-party MCP servers at your own risk. Anthropic has not verified the correctness or security of all servers. Be especially careful with servers that could fetch untrusted content, as these can expose you to prompt injection risk.
</Warning>

### Best Practices

1. **Trust the source**: Only install MCP servers from trusted publishers
2. **Review permissions**: Check what tools and resources the server exposes
3. **Use scoped credentials**: Don't share sensitive API keys in project-scoped configs
4. **Monitor usage**: Watch for unusual activity from MCP tool calls

---

## Troubleshooting

### MCP Calls Fail

| Symptom | Solution |
|---------|----------|
| Tool not found | Verify skill references correct MCP tool names |
| Connection failed | Check MCP server is running and accessible |
| Authentication error | Re-authenticate with `/mcp` command |
| Timeout | Increase `MCP_TIMEOUT` environment variable |

### Common Issues

| Issue | Solution |
|-------|----------|
| "Connection closed" | On Windows, use `cmd /c` wrapper for npx servers |
| "Tool not found" | Verify MCP tool name format: `mcp__server__tool` |
| Output too large | Set `MAX_MCP_OUTPUT_TOKENS` environment variable |

---

## MCP Output Limits

When MCP tools produce large outputs:

- **Warning threshold**: 10,000 tokens
- **Default limit**: 25,000 tokens
- **Configurable**: Set `MAX_MCP_OUTPUT_TOKENS` environment variable

```bash
# Increase limit to 50,000 tokens
export MAX_MCP_OUTPUT_TOKENS=50000
claude
```

---

## Managed MCP Configuration

For organizations, two configuration options exist:

### Option 1: Exclusive Control with `managed-mcp.json`

Deploy a fixed set of MCP servers that users cannot modify.

**Locations:**
- macOS: `/Library/Application Support/ClaudeCode/managed-mcp.json`
- Linux: `/etc/claude-code/managed-mcp.json`
- Windows: `C:\Program Files\ClaudeCode\managed-mcp.json`

### Option 2: Policy-Based Control

Use `allowedMcpServers` and `deniedMcpServers` in settings:

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverUrl": "https://mcp.company.com/*" }
  ],
  "deniedMcpServers": [
    { "serverName": "dangerous-server" }
  ]
}
```

---

## Quick Reference

| Command | Description |
|---------|-------------|
| `claude mcp add <name> <url>` | Add HTTP server |
| `claude mcp add --transport stdio <name> -- <command>` | Add stdio server |
| `claude mcp list` | List all servers |
| `claude mcp remove <name>` | Remove a server |
| `claude mcp get <name>` | Get server details |
| `claude mcp add-json <name> '<json>'` | Add from JSON |
| `claude mcp add-from-claude-desktop` | Import from Claude Desktop |
| `claude mcp serve` | Serve as MCP server |
| `/mcp` | Authenticate with remote servers |

---

## Related Documentation

| Topic | File |
|-------|------|
| Skills | 01-skills-context.md |
| Commands | 01-skills-context.md |
| Agents | 02-subagents.md |
| Hooks | 03-hooks-specific.md (Part 1) |
| Quality Practices | 04-practices.md |
| Reference Tables | 05-reference.md |

---

---

## MCP TypeScript SDK v1.x (Server Development)

> **[OFFICIAL]** Source: [github.com/modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk) (v1.x production branch)

This section covers building MCP servers using the official TypeScript SDK v1.x.

---

### SDK Overview

The `@modelcontextprotocol/sdk` package provides the core classes and transports for building MCP servers in TypeScript.

#### Package and Installation

```bash
npm install @modelcontextprotocol/sdk zod
```

Zod is a required peer dependency for schema validation.

#### Core Imports

```typescript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import { StreamableHTTPServerTransport } from '@modelcontextprotocol/sdk/server/streamableHttp.js';
import * as z from 'zod/v4';
```

> **Note**: The SDK uses Zod v4. Projects using Zod v3.25+ can also import from `zod/v3` for backwards compatibility.

---

### Quick Start: Your First Server

Create a minimal MCP server with stdio transport:

```typescript
import { McpServer, StdioServerTransport } from '@modelcontextprotocol/sdk/server/mcp.js';
import * as z from 'zod/v4';

const server = new McpServer({
    name: 'my-server',
    version: '1.0.0'
});

server.registerTool(
    'greet',
    {
        description: 'Greet a user by name',
        inputSchema: {
            name: z.string().describe('Name to greet')
        }
    },
    async ({ name }) => {
        return {
            content: [{ type: 'text', text: `Hello, ${name}!` }]
        };
    }
);

async function main() {
    const transport = new StdioServerTransport();
    await server.connect(transport);
    console.error('Server running...');
}

main().catch(console.error);
```

Run with `npx tsx server.ts`.

---

### Transports

#### Stdio Transport

Best for local development and CLI tools:

```typescript
import { McpServer, StdioServerTransport } from '@modelcontextprotocol/sdk/server/mcp.js';

const server = new McpServer({ name: 'local-server', version: '1.0.0' });
const transport = new StdioServerTransport();
await server.connect(transport);
```

**Warning**: Never use `console.log()` in stdio mode - it corrupts JSON-RPC messages. Use `console.error()` for logging.

#### Streamable HTTP Transport (Recommended for Remote)

```typescript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { StreamableHTTPServerTransport } from '@modelcontextprotocol/sdk/server/streamableHttp.js';

const server = new McpServer({
    name: 'remote-server',
    version: '1.0.0'
});

const transport = new StreamableHTTPServerTransport({
    sessionIdGenerator: undefined // undefined = stateless
});

await server.connect(transport);
```

#### DNS Rebinding Protection

MCP servers running on localhost are vulnerable to DNS rebinding attacks. Use `createMcpExpressApp()` to create an Express app with DNS rebinding protection enabled by default:

```typescript
import express from 'express';
import { createMcpExpressApp } from '@modelcontextprotocol/sdk/server/express.js';

const app = createMcpExpressApp({
    host: '127.0.0.1'  // Enables DNS rebinding protection
});

app.use(express.json());

app.post('/mcp', async (req, res) => {
    await transport.handleRequest(req, res, req.body);
});

app.get('/mcp', async (req, res) => {
    await transport.handleRequest(req, res);
});

app.listen(3000);
```

**Protection levels:**

| Host | Protection |
|------|------------|
| `127.0.0.1` | Auto-enabled |
| `localhost` | Auto-enabled |
| `::1` | Auto-enabled |
| `0.0.0.0` | No auto-protection |

For custom host validation, use the middleware directly:

```typescript
import express from 'express';
import { hostHeaderValidation } from '@modelcontextprotocol/sdk/server/middleware/hostHeaderValidation.js';

const app = express();
app.use(express.json());
app.use(hostHeaderValidation(['localhost', '127.0.0.1', 'myhost.local']));
```

#### StreamableHTTPServerTransport Options

The transport supports many configuration options:

```typescript
import { StreamableHTTPServerTransport } from '@modelcontextprotocol/sdk/server/streamableHttp.js';
import { InMemoryEventStore } from '@modelcontextprotocol/sdk/examples/shared/inMemoryEventStore.js';

const transport = new StreamableHTTPServerTransport({
    sessionIdGenerator: () => crypto.randomUUID(),
    onsessioninitialized: (sessionId) => {
        console.log('Session initialized:', sessionId);
    },
    onsessionclosed: (sessionId) => {
        console.log('Session closed:', sessionId);
    },
    eventStore: new InMemoryEventStore(),
    enableJsonResponse: false,
    retryInterval: 5000
});
```

| Option | Description |
|--------|-------------|
| `sessionIdGenerator` | Function to generate session IDs. Omit for stateless mode. |
| `onsessioninitialized` | Callback when a session is created |
| `onsessionclosed` | Callback when a session ends |
| `eventStore` | Store for message resumability (optional) |
| `enableJsonResponse` | Use JSON responses instead of SSE (default: false) |
| `retryInterval` | SSE retry interval in ms for polling |

---

### Tools

Tools let LLMs execute actions on your server.

#### Basic Tool

```typescript
server.registerTool(
    'calculate-bmi',
    {
        title: 'Calculate Body Mass Index',
        description: 'Computes the Body Mass Index (BMI) from weight and height measurements.',
        inputSchema: {
            weightKg: z.number().positive().describe('Weight in kilograms'),
            heightM: z.number().positive().describe('Height in meters')
        },
        outputSchema: {
            bmi: z.number().describe('Calculated BMI value'),
            category: z.enum(['underweight', 'normal', 'overweight', 'obese']).describe('BMI category')
        }
    },
    async ({ weightKg, heightM }) => {
        const bmi = weightKg / (heightM * heightM);
        const category = bmi < 18.5 ? 'underweight' :
                        bmi < 25 ? 'normal' :
                        bmi < 30 ? 'overweight' : 'obese';
        return {
            content: [{ type: 'text', text: `BMI: ${bmi.toFixed(1)} (${category})` }],
            structuredContent: { bmi: Number(bmi.toFixed(1)), category }
        };
    }
);
```

#### Tool with Multiple Parameters

```typescript
server.registerTool(
    'create-user',
    {
        title: 'Create User Account',
        description: 'Creates a new user account in the system with the specified name, email, and role.',
        inputSchema: {
            name: z.string().min(1).max(100).describe('Full name of the user'),
            email: z.string().email().describe('Valid email address'),
            role: z.enum(['admin', 'user']).default('user').describe('Permission level')
        }
    },
    async ({ name, email, role }) => {
        const user = await createUser({ name, email, role });
        return {
            content: [{ type: 'text', text: `Created user: ${user.id}` }],
            structuredContent: user
        };
    }
);
```
        })
    },
    async ({ name, email, role }) => {
        const user = await createUser({ name, email, role });
        return {
            content: [{ type: 'text', text: `Created user: ${user.id}` }],
            structuredContent: user
        };
    }
);
```

#### Error Handling

```typescript
server.registerTool(
    'fetch-external-data',
    {
        title: 'Fetch External API Data',
        description: 'Retrieves JSON data from a specified HTTP URL endpoint.',
        inputSchema: {
            url: z.string().url().describe('Valid HTTP/HTTPS URL')
        }
    },
    async ({ url }) => {
        try {
            const response = await fetch(url);
            if (!response.ok) {
                return {
                    content: [{ type: 'text', text: `HTTP ${response.status}: ${response.statusText}` }],
                    isError: true
                };
            }
            const data = await response.json();
            return {
                content: [{ type: 'text', text: JSON.stringify(data, null, 2) }],
                structuredContent: data
            };
        } catch (error) {
            return {
                content: [{ type: 'text', text: `Failed to fetch: ${error instanceof Error ? error.message : 'Unknown error'}` }],
                isError: true
            };
        }
    }
);
```
            return {
                content: [{ type: 'text', text: `Failed to fetch: ${error instanceof Error ? error.message : 'Unknown error'}` }],
                isError: true
            };
        }
    }
);
```

#### Error Handling

```typescript
server.registerTool(
    'fetch-external-data',
    {
        title: 'Fetch External API Data',
        description: 'Retrieves JSON data from a specified HTTP URL endpoint.',
        inputSchema: {
            url: z.string().url().describe('Valid HTTP/HTTPS URL')
        }
    },
    async ({ url }) => {
        try {
            const response = await fetch(url);
            if (!response.ok) {
                return {
                    content: [{ type: 'text', text: `HTTP ${response.status}: ${response.statusText}` }],
                    isError: true
                };
            }
            const data = await response.json();
            return {
                content: [{ type: 'text', text: JSON.stringify(data, null, 2) }],
                structuredContent: data
            };
        } catch (error) {
            return {
                content: [{ type: 'text', text: `Failed to fetch: ${error instanceof Error ? error.message : 'Unknown error'}` }],
                isError: true
            };
        }
    }
);
```

---

### Resources

Resources expose read-only data to clients.

#### Fixed Resource

```typescript
server.registerResource(
    'app-configuration',
    'config://app',
    {
        title: 'Application Configuration',
        description: 'Returns the current application configuration settings.',
        mimeType: 'application/json'
    },
    async () => {
        const config = await loadConfig();
        return {
            contents: [{
                uri: 'config://app',
                mimeType: 'application/json',
                text: JSON.stringify(config, null, 2)
            }]
        };
    }
);
```

#### Resource Template

```typescript
import { ResourceTemplate } from '@modelcontextprotocol/sdk/server/mcp.js';

server.registerResource(
    'user-file',
    new ResourceTemplate('file://users/{userId}', { list: undefined }),
    {
        title: 'User File Resource',
        description: 'Retrieves files associated with a specific user account.',
        mimeType: 'text/plain'
    },
    async (uri, variables, extra) => {
        const content = await readUserFile(variables.userId);
        return {
            contents: [{
                uri: uri.href,
                mimeType: 'text/plain',
                text: content
            }]
        };
    }
);
```

**Resource Template Options:**

```typescript
new ResourceTemplate(uriPattern, {
    list: async (extra) => {
        // Return all matching resources
        return { resources: [...] };
    },
    complete: {
        userId: async (value, extra) => {
            // Autocomplete for userId variable
            return ['user1', 'user2', 'user3'];
        }
    }
});
```

---

### Prompts

Prompts are reusable templates for model interactions. Each prompt should have a clear purpose and well-documented parameters.

```typescript
server.registerPrompt(
    'review-code',
    {
        title: 'Code Review Assistant',
        description: 'Generates a structured code review prompt for analyzing code.',
        argsSchema: {
            code: z.string().describe('Source code to be reviewed'),
            language: z.string().optional().describe('Programming language')
        }
    },
    ({ code, language }) => {
        const langHint = language ? `\nProgramming Language: ${language}` : '';
        return {
            messages: [{
                role: 'user',
                content: {
                    type: 'text',
                    text: `Please review the following code and provide a comprehensive analysis:${langHint}

\`\`\`\n${code}\n\`\`\`

Focus on:
1. Security vulnerabilities
2. Performance issues
3. Code style improvements
4. Error handling
5. Best practices

Provide specific line numbers and concrete suggestions.`
                }
            }]
        };
    }
);
```

---

### Sampling (Server-Side LLM)

Request LLM completions from the client. Use sampling when your tool needs language model capabilities but you want to remain model-independent.

#### Critical Implementation Rules (v1.25.3+)

To avoid common runtime and type errors when using sampling:

1.  **Role Restrictions**: The `messages` array only supports roles `"user"` and `"assistant"`. **DO NOT use `"system"` role** in the messages array; instead, prepend system instructions to the first user message.
2.  **Required Parameters**: `maxTokens` is a **required** parameter in the `createMessage` call.
3.  **Type Safety**: Avoid `(server as any)` casts. Use the underlying `server.server.createMessage` property on the `McpServer` instance.
4.  **Response Handling**: The response `content` is a `ContentBlock`. Always check `response.content.type === 'text'` before accessing `response.content.text`.

```typescript
import { SamplingMessage } from '@modelcontextprotocol/sdk/types.js';

server.registerTool(
    'summarize-text',
    {
        title: 'Summarize Text Content',
        description: 'Generates a concise summary using the client\'s language model.',
        inputSchema: {
            text: z.string().min(50).describe('Text content to summarize'),
            maxLength: z.number().min(50).max(500).optional().describe('Maximum summary length in words')
        }
    },
    async ({ text, maxLength = 100 }) => {
        const systemPrompt = "You are a helpful assistant that summarizes text.";
        
        const response = await server.server.createMessage({
            messages: [{
                role: 'user',
                content: { 
                    type: 'text', 
                    text: `${systemPrompt}\n\nPlease summarize this text in ${maxLength} words or fewer:\n\n${text}` 
                }
            }] as SamplingMessage[],
            maxTokens: 500 // REQUIRED
        });

        const summary = response.content.type === 'text' ? response.content.text : 'Unable to generate summary';
        return {
            content: [{ type: 'text', text: summary }]
        };
    }
);
```

---

### Structured Output (2026)

For tools that require structured data (JSON), use the `generateText` function with `Output.object()` from the Vercel AI SDK. This is particularly useful for extraction tasks where the client needs to process the data programmatically.

```typescript
import { generateText, Output, jsonSchema } from 'ai';

server.registerTool(
    'extract-entities',
    {
        title: 'Extract Entities',
        description: 'Extracts named entities into a structured JSON format.',
        inputSchema: {
            text: z.string().describe('Text to analyze'),
            jsonSchema: z.string().describe('JSON schema for the output')
        }
    },
    async ({ text, jsonSchema: schemaString }) => {
        const { output } = await generateText({
            model: step35Flash,
            output: Output.object({ schema: jsonSchema(JSON.parse(schemaString)) }),
            system: "You are an entity extraction specialist.",
            prompt: `Extract entities from: ${text}`
        });

        return {
            content: [{ type: 'text', text: JSON.stringify(output, null, 2) }],
            structuredContent: output
        };
    }
);
```

> [!TIP]
> **Preferred Pattern (2026)**: For complex analysis tasks or when specific model performance is required, direct delegation via the **Vercel AI SDK** (e.g., using OpenRouter) is preferred over MCP Sampling. This ensures model consistency and removes dependency on client-side LLM availability.

---

### Error Handling in Tools

#### Undefined Variable Pitfalls

When catching errors in tool handlers, ensure you define your message variables before using them in conditional logic to avoid "variable is undefined" runtime crashes.

```typescript
} catch (err) {
    const message = err instanceof Error ? err.message : String(err);
    const code = (typeof err === 'object' && err !== null && 'code' in err) 
        ? (err as { code?: number }).code 
        : undefined;

    const isSamplingNotSupported = 
        code === -32601 || 
        message.includes("Method not found") || 
        message.includes("sampling/createMessage");

    const userMessage = isSamplingNotSupported
        ? "Sampling not supported by client."
        : message;

    return {
        isError: true,
        content: [{ type: 'text', text: `Error: ${userMessage}` }]
    };
}
```

---

### Resource URI Parsing

When extracting data from dynamic resource URIs, use anchored regex (e.g., `/^\/prefix\//`) instead of simple string replacement to avoid accidental matches elsewhere in the URL.

```typescript
server.registerResource(
    "webResource",
    "webfetch://dynamic/{url}",
    { title: "Dynamic Resource" },
    async (uri) => {
        // Use anchored regex to strip the prefix safely
        const url = decodeURIComponent(uri.pathname.replace(/^\/dynamic\//, ""));
        // ...
    }
);
```

---

### TypeScript Configuration & Schema Order

1.  **Schema Definition Order**: Always define your Zod schemas *before* using `z.infer` to create types. Referencing a schema before its declaration will cause TypeScript compilation errors.
2.  **Typechecking**: Add a `typecheck` script to your `package.json` to verify types without generating build artifacts.

```json
"scripts": {
    "typecheck": "tsc -p tsconfig.json --noEmit"
}
```

---

### Tool Hygiene

1.  **Tool Inventory**: Ensure your `README.md` and tool registration counts are synchronized.
2.  **Naming**: Prefer `registerTool` over the deprecated `.tool()` method for better type support and future-proofing.


### Elicitation

> **[COMMUNITY BEST PRACTICE]**: Elicitation should be avoided by default. It disrupts the autonomous flow of the agent and creates a blocking user interaction pattern.

Two modes are supported:
- **Form mode**: Collect non-sensitive structured data via a schema-driven form
- **URL mode**: Direct users to complete sensitive flows in a browser (OAuth, payments, API keys)

#### Form Elicitation

```typescript
server.registerTool(
    'register-user',
    {
        title: 'User Registration',
        description: 'Collects user registration information',
        inputSchema: {}
    },
    async () => {
        const result = await server.server.elicitInput({
            mode: 'form',
            message: 'Please provide your registration information:',
            requestedSchema: {
                type: 'object',
                properties: {
                    username: {
                        type: 'string',
                        title: 'Username',
                        description: 'Your desired username (3-20 characters)',
                        minLength: 3,
                        maxLength: 20
                    },
                    email: {
                        type: 'string',
                        title: 'Email',
                        description: 'Your email address',
                        format: 'email'
                    }
                },
                required: ['username', 'email']
            }
        });

        if (result.action === 'accept' && result.content) {
            const { username, email } = result.content as { username: string; email: string };
            return {
                content: [{ type: 'text', text: `User registered: ${username} (${email})` }]
            };
        }
        return {
            content: [{ type: 'text', text: 'Registration cancelled' }]
        };
    }
);
```

#### URL Elicitation

For sensitive data (API keys, OAuth, payments), use URL elicitation to open a browser:

```typescript
import { UrlElicitationRequiredError } from '@modelcontextprotocol/sdk/types.js';

server.registerTool(
    'payment-confirm',
    {
        title: 'Payment Confirmation',
        description: 'Confirms a payment with user interaction',
        inputSchema: {
            cartId: z.string().describe('The ID of the cart to confirm')
        }
    },
    async ({ cartId }, extra) => {
        const sessionId = extra.sessionId;
        if (!sessionId) {
            throw new Error('Expected a Session ID');
        }

        const elicitationId = generateElicitationId();
        throw new UrlElicitationRequiredError([
            {
                mode: 'url',
                message: 'Open the link to confirm payment!',
                url: `https://example.com/payment?session=${sessionId}&elicitation=${elicitationId}&cartId=${cartId}`,
                elicitationId
            }
        ]);
    }
);
```

**Key differences:**

| Aspect | Form Elicitation | URL Elicitation |
|--------|------------------|-----------------|
| Data type | Non-sensitive | Sensitive (secrets, payments) |
| User flow | In-client form | External browser |
| API | `server.server.elicitInput()` | Throw `UrlElicitationRequiredError` |
| Completion | Auto-returned in response | Requires tracking via callback |

---

### Tasks (Experimental)

Long-running operations with polling and resumption support. Tasks are useful for operations that exceed typical request timeouts, require background processing, or need to survive client disconnections.

```typescript
import { randomUUID } from 'node:crypto';
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { StreamableHTTPServerTransport } from '@modelcontextprotocol/sdk/server/streamableHttp.js';
import { CallToolResult, GetTaskResult } from '@modelcontextprotocol/sdk/types.js';
import {
    InMemoryTaskStore,
    InMemoryTaskMessageQueue
} from '@modelcontextprotocol/sdk/experimental/tasks/stores/in-memory.js';

const taskStore = new InMemoryTaskStore();
const taskQueue = new InMemoryTaskMessageQueue();

const server = new McpServer(
    { name: 'task-server', version: '1.0.0' },
    { capabilities: { tasks: { requests: { tools: { call: {} } } } } }
);

server.connect(
    new StreamableHTTPServerTransport({
        sessionIdGenerator: () => randomUUID(),
        eventStore: { storeEvent: async () => 'evt_' + randomUUID() }
    })
);

server.experimental.tasks.registerToolTask(
    'process-dataset',
    {
        title: 'Process Large Dataset',
        description: 'Processes a large dataset asynchronously.',
        inputSchema: {
            datasetId: z.string().describe('Identifier of the dataset'),
            outputFormat: z.enum(['json', 'csv', 'parquet']).default('json').describe('Output format')
        }
    },
    {
        async createTask(args, extra) {
            const task = await extra.taskStore.createTask(
                { ttl: 3600000, context: args },
                extra.requestId,
                extra.request,
                extra.sessionId
            );

            (async () => {
                try {
                    const result = await processDataset(args.datasetId, args.outputFormat);
                    await extra.taskStore.storeTaskResult(
                        task.taskId,
                        'completed',
                        { content: [{ type: 'text', text: `Processed ${args.datasetId}: ${result}` }] } as CallToolResult,
                        extra.sessionId
                    );
                } catch (error) {
                    await extra.taskStore.storeTaskResult(
                        task.taskId,
                        'failed',
                        { content: [{ type: 'text', text: `Failed: ${error.message}` }] } as CallToolResult,
                        extra.sessionId
                    );
                }
            })();

            return { task };
        },
        async getTask(_, extra) {
            return await extra.taskStore.getTask(extra.taskId, extra.sessionId);
        },
        async getTaskResult(_, extra) {
            return await extra.taskStore.getTaskResult(extra.taskId, extra.sessionId) as CallToolResult;
        }
    }
);
```

**Task Handler Context:**

The `extra` parameter in task handlers has these properties:

| Property | Type | Description |
|----------|------|-------------|
| `extra.taskStore` | `RequestTaskStore` | Store for task operations |
| `extra.taskId` | `string` | Current task ID (getTask, getTaskResult) |
| `extra.requestId` | `RequestId` | JSON-RPC request ID |
| `extra.request` | `Request` | Original request object |
| `extra.sessionId` | `string` | Session ID for stateful mode |
| `extra.requestContext` | `Record<string, unknown>` | Additional context |

**TaskStore Methods:**

```typescript
interface CreateTaskOptions {
    ttl?: number | null;
    pollInterval?: number;
    context?: Record<string, unknown>;
}

interface TaskStore {
    createTask(
        params: CreateTaskOptions,
        requestId: RequestId,
        request: Request,
        sessionId?: string
    ): Promise<Task>;

    getTask(taskId: string, sessionId?: string): Promise<Task | null>;

    storeTaskResult(
        taskId: string,
        status: 'completed' | 'failed',
        result: Result,
        sessionId?: string
    ): Promise<void>;

    getTaskResult(taskId: string, sessionId?: string): Promise<Result>;

    updateTaskStatus(
        taskId: string,
        status: TaskStatus,
        statusMessage?: string,
        sessionId?: string
    ): Promise<void>;

    listTasks(cursor?: string, sessionId?: string): Promise<{ tasks: Task[]; nextCursor?: string }>;
}

interface TaskMessageQueue {
    enqueue(taskId: string, message: QueuedMessage, sessionId?: string, maxSize?: number): Promise<void>;
    dequeue(taskId: string, sessionId?: string): Promise<QueuedMessage | undefined>;
    dequeueAll(taskId: string, sessionId?: string): Promise<QueuedMessage[]>;
}
```

**CreateTaskOptions Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `ttl` | `number \| null` | Time-to-live in ms. `null` = unlimited lifetime |
| `pollInterval` | `number` | ms between polling requests (optional) |
| `context` | `Record<string, unknown>` | Metadata passed to task handlers |

---

### Best Practices

#### Tool Description Guidelines

Write clear, comprehensive tool descriptions that help LLMs understand when and how to use your tools:

```typescript
server.registerTool(
    'web-search',
    {
        title: 'Web Search',
        description: `Performs web searches to find current information on the internet.

When to use:
    - Answering questions about current events
    - Finding official documentation
    - Researching products or companies

Returns a structured list of search results with titles, URLs, and descriptions.`,
        inputSchema: {
            query: z.string().min(3).describe('Search query (minimum 3 characters)')
        }
    },
    async ({ query }) => {
        const results = await performSearch(query);
        return {
            content: [{ type: 'text', text: JSON.stringify(results) }],
            structuredContent: { results }
        };
    }
);
```

#### Logging

```typescript
// Use console.error for logging (stdio mode)
console.error('Processing started');

// For HTTP transport, use any logging library
import winston from 'winston';
const logger = winston.createLogger({ level: 'info' });
```

#### Async Operations

```typescript
server.registerTool(
    'process-items',
    {
        description: 'Process multiple items',
        inputSchema: {
            items: z.array(z.string()).describe('Items to process')
        }
    },
    async ({ items }) => {
        const results = await Promise.all(
            items.map(async item => await processItem(item))
        );
        return {
            content: [{ type: 'text', text: `Processed ${results.length} items` }],
            structuredContent: { results }
        };
    }
);
```

#### Input Validation

Leverage Zod for comprehensive validation:

```typescript
server.registerTool(
    'validate-user',
    {
        description: 'Create a validated user',
        inputSchema: {
            name: z.string().min(2).max(50).describe('Name (2-50 chars)'),
            email: z.string().email().describe('Valid email'),
            age: z.number().min(0).max(120).optional().describe('Age'),
            tags: z.array(z.string()).max(5).optional().describe('Tags')
        }
    },
    async (input) => {
        return { content: [{ type: 'text', text: 'Valid' }] };
    }
);
```

---

### SDK Documentation Links

| Resource | URL | Description |
|----------|-----|-------------|
| SDK Repository | https://github.com/modelcontextprotocol/typescript-sdk | Main SDK repository (v2 in development) |
| v1.x Branch | https://github.com/modelcontextprotocol/typescript-sdk/tree/v1.x | Production-stable v1.x SDK |
| SDK API Docs | https://modelcontextprotocol.github.io/typescript-sdk/ | Generated API documentation |
| Server Guide | https://github.com/modelcontextprotocol/typescript-sdk/blob/v1.x/docs/server.md | Building MCP servers |
| Capabilities Guide | https://github.com/modelcontextprotocol/typescript-sdk/blob/v1.x/docs/capabilities.md | Sampling, elicitation, tasks |
| Protocol Spec | https://spec.modelcontextprotocol.io | Full MCP protocol specification |
| MCP Docs | https://modelcontextprotocol.io/docs | Official MCP documentation |
| Reference Servers | https://github.com/modelcontextprotocol/servers | Collection of example servers |

---

## Structured Tool Definitions Guide

> **[ORCHESTRATION PRINCIPLE]**: Tool descriptions are the primary mechanism for controlling what AI agents perceive and can invoke. Well-crafted descriptions with clear "When to use" guidance act as the orchestration layer between your server capabilities and LLM decision-making. The LLM sees ONLY what you write in the description—use this to guide behavior, prevent misuse, and set expectations.

### Why Detailed Descriptions Matter

When an LLM inspects an MCP server, it sees only:
1. Tool name
2. Title (if provided)
3. Description (entire text you provide)
4. Input schema (parameters)

**The description is your primary control point.** It determines whether the LLM will:
- Choose this tool over alternatives
- Understand when to invoke it
- Set proper user expectations
- Avoid costly or inappropriate invocations

### Multi-Parameter Tool Example: Text-to-Speech

```typescript
server.registerTool(
    'text_to_audio',
    {
        title: 'Text to Speech Converter',
        description: `Converts text to spoken audio using Minimax's TTS API and saves the output file.

WHEN TO USE:
    - When the user explicitly requests audio output of written content
    - For accessibility purposes (converting articles, documents, or messages to speech)
    - Creating voiceovers for videos or presentations
    - Generating podcast-style content from scripts
    - Producing audio versions of written materials

WHEN NOT TO USE:
    - For routine text processing without audio request
    - When the user wants written summaries (use summarize-text instead)
    - Before confirming user wants audio output

COST WARNING: This tool makes an API call to Minimax which may incur costs. Only invoke when explicitly requested by the user.

OUTPUT: Returns text content with the path to the generated audio file and the voice model used.`,
        inputSchema: z.object({
            text: z.string().min(1).describe('The text content to convert to speech'),
            voice_id: z.string().optional().describe('Voice model ID for speech synthesis. Options: "male-qn-qingse", "audiobook_female_1", "cute_boy", "Charming_Llady"... Defaults to system default voice'),
            model: z.string().optional().describe('TTS model to use. Defaults to optimal model for the selected voice'),
            speed: z.number().min(0.5).max(2.0).optional().default(1.0).describe('Speech speed multiplier. 0.5 = half speed, 2.0 = double speed. Default: 1.0'),
            pitch: z.number().min(-12).max(12).optional().default(0).describe('Voice pitch adjustment. Range: -12 (deeper) to +12 (higher). Default: 0'),
            volume: z.number().min(0).max(10).optional().default(1).describe('Audio volume level. Range: 0 (silent) to 10 (loudest). Default: 1'),
            emotion: z.enum(['happy', 'sad', 'angry', 'fearful', 'disgusted', 'surprised', 'neutral']).optional().default('happy').describe('Emotional tone of the speech. Default: happy'),
            sample_rate: z.number().int().optional().default(32000).describe('Audio sample rate in Hz. Options: 8000, 16000, 22050, 24000, 32000, 44100. Default: 32000'),
            bitrate: z.number().int().optional().default(128000).describe('Audio bitrate in bps. Options: 32000, 64000, 128000, 256000. Default: 128000'),
            channel: z.number().int().optional().default(1).describe('Audio channels. 1 = mono, 2 = stereo. Default: 1'),
            format: z.enum(['pcm', 'mp3', 'flac']).optional().default('mp3').describe('Output audio format. Default: mp3'),
            language_boost: z.string().optional().default('auto').describe('Language boost for pronunciation. Options: "Chinese", "English", "Japanese", "auto"... Default: auto'),
            output_directory: z.string().optional().describe('Directory to save the audio file. Defaults to user\'s Desktop if not specified')
        })
    },
    async ({ text, voice_id, model, speed, pitch, volume, emotion, sample_rate, bitrate, channel, format, language_boost, output_directory }) => {
        const result = await generateAudio({
            text,
            voice_id,
            model,
            speed,
            pitch,
            volume,
            emotion,
            sample_rate,
            bitrate,
            channel,
            format,
            language_boost,
            output_directory
        });
        return {
            content: [{ type: 'text', text: `Audio saved to: ${result.path}\nVoice: ${result.voice}` }],
            structuredContent: result
        };
    }
);
```

### Creative Generation Tool: Image Generation

```typescript
server.registerTool(
    'text_to_image',
    {
        title: 'AI Image Generator',
        description: `Generates images from text prompts using Minimax's image generation API.

WHEN TO USE:
    - When the user explicitly requests image creation or visual content
    - For illustrations, concept art, or visual aids requested by the user
    - Creating featured images for articles or presentations
    - Generating graphics based on specific visual descriptions

WHEN NOT TO USE:
    - For routine text processing
    - Without explicit user request for images
    - When the user needs written descriptions (use other tools instead)

COST WARNING: This tool makes API calls to Minimax which may incur costs per image generated.

OUTPUT: Returns text with the path to generated image file(s). Supports saving to specified directory.`,
        inputSchema: z.object({
            prompt: z.string().min(10).describe('Detailed text description of the image to generate. Be specific about style, subjects, colors, composition, and mood for best results'),
            model: z.enum(['image-01']).optional().default('image-01').describe('Image generation model. Currently supports: image-01'),
            aspect_ratio: z.enum(['1:1', '16:9', '4:3', '3:2', '2:3', '3:4', '9:16', '21:9']).optional().default('1:1').describe('Image aspect ratio. Common options: 1:1 (square), 16:9 (widescreen), 4:3 (standard), 9:16 (story/vertical)'),
            n: z.number().int().min(1).max(9).optional().default(1).describe('Number of images to generate per prompt. Range: 1-9. Default: 1'),
            prompt_optimizer: z.boolean().optional().default(true).describe('Whether to optimize the prompt for better generation results. Default: true (recommended)'),
            output_directory: z.string().optional().describe('Directory to save generated images. Defaults to working directory if not specified')
        })
    },
    async ({ prompt, model, aspect_ratio, n, prompt_optimizer, output_directory }) => {
        const images = await generateImage({
            prompt,
            model,
            aspect_ratio,
            n,
            prompt_optimizer,
            output_directory
        });
        return {
            content: [{ type: 'text', text: `Generated ${images.length} image(s):\n${images.map((img, i) => `${i + 1}. ${img.path}`).join('\n')}` }],
            structuredContent: { images }
        };
    }
);
```

### Creative Generation Tool: Video Generation

```typescript
server.registerTool(
    'generate_video',
    {
        title: 'AI Video Generator',
        description: `Generates videos from text prompts or images using Minimax's video generation API.

WHEN TO USE:
    - When the user explicitly requests video content creation
    - For short clips, animations, or motion content
    - Converting image concepts to video with Director models
    - Creating visual content for presentations or social media

WHEN NOT TO USE:
    - For routine content creation without video request
    - Without explicit user authorization (video generation is expensive)
    - For complex long-form video (this tool creates short clips)

COST WARNING: This tool makes API calls to Minimax with significant costs per video. Only invoke when explicitly requested.

OUTPUT: Returns text with the path to the generated video file.

MODEL SELECTION GUIDE:
    - T2V-01: Text-to-video from prompt
    - T2V-01-Director: Text-to-video with camera movement instructions
    - I2V-01: Image-to-video (animate a static image)
    - I2V-01-Director: Image-to-video with camera control`,
        inputSchema: z.object({
            model: z.enum(['T2V-01', 'T2V-01-Director', 'I2V-01', 'I2V-01-Director', 'I2V-01-live']).optional().default('T2V-01').describe('Video generation model. T2V = text-to-video, I2V = image-to-video. "Director" versions support camera movement instructions.'),
            prompt: z.string().min(10).describe('Text description of the video to generate. For Director models, supports Camera Movement Instructions: [Truck left/right], [Pan left/right], [Push in/Pull out], [Pedestal up/down], [Tilt up/down], [Zoom in/out], [Shake], [Tracking shot], [Static shot]'),
            first_frame_image: z.string().optional().describe('URL or path to image for I2V models. Required when model starts with "I2V"'),
            output_directory: z.string().optional().describe('Directory to save the video. Defaults to working directory')
        })
    },
    async ({ model, prompt, first_frame_image, output_directory }) => {
        const video = await generateVideo({
            model,
            prompt,
            first_frame_image,
            output_directory
        });
        return {
            content: [{ type: 'text', text: `Video generated: ${video.path}\nDuration: ${video.duration}s\nModel: ${model}` }],
            structuredContent: video
        };
    }
);
```

### Utility Tool: Voice Cloning

```typescript
server.registerTool(
    'voice_clone',
    {
        title: 'Voice Cloning Service',
        description: `Creates a cloned voice from provided audio samples for use with text-to-speech.

WHEN TO USE:
    - When the user wants to create a custom voice for TTS
    - For brand consistency across audio content
    - When specific voice characteristics are needed
    - After obtaining proper consent for voice cloning

REQUIREMENTS:
    - Must provide at least 10 seconds of clear audio
    - Audio should be free of background noise
    - Speaker should speak at normal pace

COST WARNING: Cloning is billed on first use. The cloned voice will consume credits when used for TTS generation.

OUTPUT: Returns the new voice ID for use with text_to_audio tool.`,
        inputSchema: z.object({
            voice_id: z.string().min(1).describe('Unique identifier for the new cloned voice'),
            audio_source: z.string().describe('Path to audio file OR URL of audio to clone'),
            reference_text: z.string().optional().describe('Transcript of the audio for verification and alignment'),
            is_url: z.boolean().optional().default(false).describe('Set true if audio_source is a URL, false if local file path')
        })
    },
    async ({ voice_id, audio_source, reference_text, is_url }) => {
        const result = await cloneVoice({
            voice_id,
            audio_source,
            reference_text,
            is_url
        });
        return {
            content: [{ type: 'text', text: `Voice cloned successfully!\nVoice ID: ${voice_id}\nUse with text_to_audio tool.` }],
            structuredContent: result
        };
    }
);
```

### Informational Tool: List Available Voices

```typescript
server.registerTool(
    'list_voices',
    {
        title: 'Voice Library Browser',
        description: `Lists all available TTS voice models with their characteristics.

WHEN TO USE:
    - When the user wants to browse available voice options
    - Before using text_to_audio to select an appropriate voice
    - For finding voices with specific attributes (gender, age, accent)

OUTPUT: Returns a formatted list of available voices with IDs, names, and characteristics.`,
        inputSchema: z.object({
            voice_type: z.enum(['all', 'system', 'voice_cloning']).optional().default('all').describe('Filter voices by type: "all" = all available voices, "system" = built-in voices, "voice_cloning" = user-cloned voices')
        })
    },
    async ({ voice_type }) => {
        const voices = await listVoices(voice_type);
        return {
            content: [{
                type: 'text',
                text: `Available Voices:\n${voices.map(v => `- ${v.id}: ${v.name} (${v.gender}, ${v.accent})`).join('\n')}`
            }],
            structuredContent: { voices }
        };
    }
);
```

### Tool Description Template

Use this structure for consistency:

```typescript
server.registerTool(
    'tool_name',
    {
        title: 'Human-Readable Title',
        description: `One-line summary of what the tool does.

WHEN TO USE:
    - Specific scenario 1
    - Specific scenario 2
    - Specific scenario 3

WHEN NOT TO USE:
    - Scenario where this tool is inappropriate
    - Situations requiring different tools

[COST/WARNING/SECURITY notices if applicable]

OUTPUT: Description of what the tool returns`,
        inputSchema: z.object({
            param1: z.type().describe('Description with constraints'),
            param2: z.type().optional().describe('Optional parameter with defaults'),
            // ... more parameters
        })
    },
    async ({ param1, param2 }) => {
        // Implementation
    }
);
```

---

## Orchestration & Tool Navigation Guide

> **[ORCHESTRATION PRINCIPLE]**: Design tools as a cohesive system where each tool's description explicitly guides the LLM to the next logical tool. Use DO/DON'T patterns, return value documentation, and clear workflow guidance to create navigable tool chains.

### Why Tool Navigation Matters

When building related tools (search → detail → enrichment), each tool must:
1. Tell the LLM what it returns (so it knows what to do next)
2. Guide the LLM to the next tool (workflow enforcement)
3. Warn against redundant calls (efficiency)
4. Define clear boundaries (when NOT to use)

### Example: Business Data Research Workflow

```typescript
server.registerTool(
    'autocomplete',
    {
        title: 'Filter Value Autocomplete',
        description: `Provides valid enum values for Explorium API filter fields. MUST be called before using filters in other tools to ensure valid parameter values.

WHEN TO USE:
    - Before any search/fetch tool that requires filter parameters
    - To discover available options for: country, country_code, region_country_code, google_category, naics_category, linkedin_category, company_tech_stack_tech, company_tech_stack_categories, job_title, company_size, company_revenue, number_of_locations, company_age, job_department, job_level, city_region_country, company_name

CALLING PATTERN: Call this tool in parallel (simultaneously) for all needed filters. Do NOT call sequentially. Each filter requires its own autocomplete call with the query parameter.

HINTS:
    - Looking for 'saas' categories? Search for 'software' instead
    - Need country codes? Search by country name first

OUTPUT: Returns valid enum values that can be used in subsequent filter parameters.`,
        inputSchema: z.object({
            query: z.string().min(1).describe('Search term for filter values'),
            field: z.enum([
                'country', 'country_code', 'region_country_code', 'google_category',
                'naics_category', 'linkedin_category', 'company_tech_stack_tech',
                'company_tech_stack_categories', 'job_title', 'company_size',
                'company_revenue', 'number_of_locations', 'company_age',
                'job_department', 'job_level', 'city_region_country', 'company_name'
            ]).describe('Filter field to get autocomplete values for')
        })
    },
    async ({ query, field }) => {
        const results = await getAutocompleteValues(query, field);
        return {
            content: [{ type: 'text', text: `Available values for ${field}:\n${results.join('\n')}` }],
            structuredContent: { field, values: results }
        };
    }
);

server.registerTool(
    'fetch_businesses',
    {
        title: 'Business Search & Discovery',
        description: `Fetches business/company records from Explorium API matching specified filters. Returns Business IDs essential for all enrichment operations.

WORKFLOW PREREQUISITES:
    1. Call autocomplete tool first to get valid filter values
    2. Use returned values in this tool's filter parameters

DO NOT call autocomplete after calling this tool—filter validation is complete.

OUTPUT: Returns Business IDs for matched companies. These IDs are REQUIRED inputs for all enrichment tools (enrich_businesses_*).

COMMON WORKFLOW:
    fetch_businesses → enrich_businesses_firmographics (or any enrich_* tool)

DO NOT USE when:
    - You already have Business IDs (use enrichment tools directly)
    - Looking for specific employees (use fetch_prospects instead)
    - Need company size/revenue/industry (call this first, then enrichment)

If a requested filter is unsupported, execution stops and user is notified.`,
        inputSchema: z.object({
            country: z.string().optional().describe('Country filter (must use autocomplete first)'),
            google_category: z.string().optional().describe('Google business category'),
            company_size: z.string().optional().describe('Employee count range'),
            company_revenue: z.string().optional().describe('Annual revenue range'),
            naics_category: z.string().optional().describe('NAICS industry code')
        })
    },
    async ({ country, google_category, company_size, company_revenue, naics_category }) => {
        const businesses = await searchBusinesses({
            country, google_category, company_size, company_revenue, naics_category
        });
        return {
            content: [{
                type: 'text',
                text: `Found ${businesses.length} businesses\nBusiness IDs: ${businesses.map(b => b.id).join(', ')}`
            }],
            structuredContent: { businesses }
        };
    }
);

server.registerTool(
    'match_businesses',
    {
        title: 'Business Lookup by Name/Domain',
        description: `Resolves business names or domains to Explorium Business IDs. Use this when you have company identifiers and need their internal IDs.

USE WHEN:
    - You have a list of company names and need Business IDs
    - You have company domains and need to look up IDs
    - You need to enrich a known company with additional data

OUTPUT: Returns Business IDs that can be used with all enrich_businesses_* tools.

DO NOT USE when:
    - You already have Business IDs (wasteful lookup)
    - Looking for employees (use fetch_prospects instead)
    - Need leadership/executive info (use enrich_businesses_financial_metrics instead)

COMMON WORKFLOW:
    1. match_businesses (get IDs)
    2. enrich_businesses_firmographics (get basic info)
    3. enrich_businesses_technographics (get tech stack)
    4. enrich_businesses_financial_metrics (get financial data)`,
        inputSchema: z.object({
            company_name: z.string().array().optional().describe('Array of company names to look up'),
            domain: z.string().array().optional().describe('Array of company domains (e.g., "example.com")')
        })
    },
    async ({ company_name, domain }) => {
        const results = await matchBusinesses({ company_name, domain });
        return {
            content: [{
                type: 'text',
                text: `Matched ${results.length} businesses\nIDs: ${results.map(r => r.id).join(', ')}`
            }],
            structuredContent: { matches: results }
        };
    }
);

server.registerTool(
    'enrich_businesses_firmographics',
    {
        title: 'Company Firmographics Enrichment',
        description: `Retrieves detailed company profile data including description, industry classification, size, and revenue.

REQUIRED INPUT: Valid Business IDs from fetch_businesses or match_businesses.

OUTPUT RETURNS:
    - Business ID and name
    - Detailed business description
    - Website URL and geographic info
    - NAICS/SIC industry codes and descriptions
    - Stock ticker (public companies)
    - Employee count range and annual revenue

DO NOT USE when:
    - You need employee details at a company (use fetch_prospects)
    - Looking for leadership/executive info (use enrich_businesses_financial_metrics)

WORKFLOW:
    fetch_businesses → enrich_businesses_firmographics
    match_businesses → enrich_businesses_firmographics`,
        inputSchema: z.object({
            business_ids: z.string().array().min(1).describe('Array of Business IDs from fetch_businesses or match_businesses')
        })
    },
    async ({ business_ids }) => {
        const data = await getFirmographics(business_ids);
        return {
            content: [{
                type: 'text',
                text: `Enriched ${data.length} companies\n${data.map(c => `- ${c.name}: ${c.industry}`).join('\n')}`
            }],
            structuredContent: { firms: data }
        };
    }
);

server.registerTool(
    'enrich_businesses_technographics',
    {
        title: 'Technology Stack Analysis',
        description: `Retrieves the complete technology stack used by companies including Sales, Marketing, DevOps tools and more.

REQUIRED INPUT: Valid Business IDs.

OUTPUT RETURNS:
    - Full technology stack categorized by function
    - Categories include: Testing tools, Sales software, Programming languages, Marketing tech, IT security, DevOps tools, CRM systems, Analytics, and 15+ more

WORKFLOW:
    fetch_businesses → enrich_businesses_technographics (compare tech stacks)
    match_businesses → enrich_businesses_technographics

USE CASE: Analyze what tools competitors use, find integration opportunities, or assess company tech maturity.`,
        inputSchema: z.object({
            business_ids: z.string().array().min(1).describe('Business IDs to get technographics for')
        })
    },
    async ({ business_ids }) => {
        const tech = await getTechnographics(business_ids);
        return {
            content: [{
                type: 'text',
                text: `Technology analysis for ${tech.length} companies`
            }],
            structuredContent: { technographics: tech }
        };
    }
);

server.registerTool(
    'fetch_prospects',
    {
        title: 'Employee/Person Search',
        description: `Finds individuals (prospects) at companies based on job title, department, seniority, and other professional attributes.

WORKFLOW PREREQUISITES:
    1. Call autocomplete first to get valid filter values
    2. Get Business IDs (use fetch_businesses or match_businesses)
    3. Call this tool with validated filters

OUTPUT: Returns Prospect IDs for individuals matching criteria.

DO NOT USE when:
    - Looking for companies (use fetch_businesses)
    - Need company firmographics (use enrich_businesses_firmographics)
    - Need leadership info at public companies (use enrich_businesses_financial_metrics)

WORKFLOW:
    fetch_businesses → fetch_prospects → enrich_prospects_profiles
    match_businesses → fetch_prospects → enrich_prospects_contacts_information`,
        inputSchema: z.object({
            business_id: z.string().describe('Business ID from fetch_businesses or match_businesses'),
            job_title: z.string().optional().describe('Job title filter (use autocomplete first)'),
            job_level: z.string().optional().describe('Seniority level (use autocomplete first)'),
            job_department: z.string().optional().describe('Department (use autocomplete first)')
        })
    },
    async ({ business_id, job_title, job_level, job_department }) => {
        const prospects = await searchProspects({ business_id, job_title, job_level, job_department });
        return {
            content: [{
                type: 'text',
                text: `Found ${prospects.length} prospects\nIDs: ${prospects.map(p => p.id).join(', ')}`
            }],
            structuredContent: { prospects }
        };
    }
);

server.registerTool(
    'enrich_prospects_profiles',
    {
        title: 'Detailed Prospect Profile Enrichment',
        description: `Retrieves comprehensive profile information for individuals including work history, education, and current role details.

REQUIRED INPUT: Prospect IDs from fetch_prospects.

OUTPUT RETURNS:
    - Full name, demographics, location
    - LinkedIn profile URL
    - Current role: company, title, department, seniority
    - Work experience history with dates
    - Education: schools, degrees, majors
    - Skills and interests when available

WORKFLOW:
    fetch_prospects → enrich_prospects_profiles

DO NOT USE when:
    - You need company-level data (use enrich_businesses_* tools)
    - Looking for contact info (use enrich_prospects_contacts_information)`,
        inputSchema: z.object({
            prospect_ids: z.string().array().min(1).describe('Array of Prospect IDs from fetch_prospects')
        })
    },
    async ({ prospect_ids }) => {
        const profiles = await getProspectProfiles(prospect_ids);
        return {
            content: [{
                type: 'text',
                text: `Enriched ${profiles.length} prospect profiles`
            }],
            structuredContent: { profiles }
        };
    }
);
```

### Navigation Pattern Summary

| Pattern | Example |
|---------|---------|
| **Sequential Required** | autocomplete → fetch_businesses (must validate filters first) |
| **Parallel Allowed** | Multiple autocomplete calls simultaneously |
| **ID-Based Chaining** | fetch_businesses → enrich_businesses_* (use returned IDs) |
| **Prohibit Redundant** | "DO NOT call match_businesses if you already have IDs" |
| **Clear Next Step** | "Returns IDs required for enrichment tools" |
| **Boundary Setting** | "DO NOT use for employees—use fetch_prospects instead" |

### Common Workflow Patterns

```
Company Research:
  match_businesses → enrich_businesses_firmographics → enrich_businesses_technographics

Employee Discovery:
  fetch_businesses → fetch_prospects → enrich_prospects_profiles → enrich_prospects_contacts_information

Competitor Analysis:
  fetch_businesses (multiple) → enrich_businesses_competitive_landscape
```

---

## External Resources

| Resource | URL | Description |
|----------|-----|-------------|
| MCP Official Documentation | https://modelcontextprotocol.io/docs | Official MCP documentation and guides |
| MCP Protocol Specification | https://spec.modelcontextprotocol.io | Full protocol specification |
| TypeScript SDK (GitHub) | https://github.com/modelcontextprotocol/typescript-sdk | Official TypeScript SDK repository |
| TypeScript SDK (v1.x branch) | https://github.com/modelcontextprotocol/typescript-sdk/tree/v1.x | Production-stable v1.x SDK |
| TypeScript SDK API Docs | https://modelcontextprotocol.github.io/typescript-sdk/ | Generated API documentation |
| Example Servers | https://github.com/modelcontextprotocol/servers | Collection of reference MCP servers |
| MCP Protocol Python SDK | https://github.com/modelcontextprotocol/python-sdk | Official Python SDK for MCP |
| MCP Protocol Go SDK | https://github.com/metoro-io/mcp-golang | Go SDK for MCP |
| MCP Registry | https://github.com/mcp-registry | Community MCP server registry |

---

*Documentation last updated: February 2026*
