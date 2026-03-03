<!-- mcp-name: SQL Server MCP -->
<!-- mcp-description: MCP server for SQL Server — schema inspection, safe read-only queries, and query plan analysis -->
<!-- mcp-tags: sql-server, mcp, database, dotnet-tool -->

# SqlServerMcp

A [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server for **SQL Server**. Exposes schema inspection, safe read-only queries, and query plan analysis as MCP tools.

## Install

```bash
dotnet tool install -g SqlServerMcp
```

## Configure

After installing, edit the `appsettings.json` in the tool's install directory to add your connection strings:

```jsonc
{
  "ConnectionStrings": {
    "MyDb": "Server=localhost;Database=MyDatabase;Trusted_Connection=True;TrustServerCertificate=True;"
  },
  "Query": {
    "DefaultTop": 1000,
    "MaxTop": 500000
  }
}
```

> **Tip:** Run `dotnet tool list -g` to find the install path, then navigate to the tool folder to find `appsettings.json`.

## Connect from an MCP Client

Add the following to your MCP client configuration (e.g. Claude Desktop `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "sqlserver": {
      "command": "sqlservermcp",
      "args": []
    }
  }
}
```

## Available Tools

| Tool | Description |
|------|-------------|
| `ListDatabases` | List available database names configured in this server |
| `ListTables` | List all user tables in a database |
| `DescribeTable` | Describe columns of a specific table (schema.table format) |
| `GetDatabaseSchema` | Return full database schema: tables, columns, PKs, FKs |
| `SearchSchema` | Search tables and columns by keyword |
| `RunQuery` | Execute a safe read-only SELECT query with auto row-limiting |
| `ExplainQuery` | Explain a SELECT query using estimated execution plan |

## Safety

- Only `SELECT` queries are allowed — all write operations are blocked
- Queries are automatically row-limited to prevent runaway results
- Connection strings are configured server-side, never exposed to the AI client

## License

MIT
