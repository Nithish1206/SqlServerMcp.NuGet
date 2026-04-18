<!-- mcp-name: SQL Server MCP -->
<!-- mcp-description: MCP server for SQL Server — schema inspection, safe read-only queries, and query plan analysis -->
<!-- mcp-tags: sql-server, mcp, database, dotnet-tool -->

# SqlServerMcp

A [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server for **SQL Server**. Exposes schema inspection, safe read-only queries, and query plan analysis as MCP tools.

> **How it works:** This server does not accept natural-language questions. Instead, the connected **AI tool (the MCP client) decides and generates the SQL query** based on the user's intent and the schema it inspects via the tools below. The server only executes the SQL the AI produces, under the safety rules described in [Safety](#safety).

## Install

```bash
dotnet tool install -g SqlServerMcp
```

## Configure

Configuration is done entirely through **environment variables** in your MCP client config. No need to edit any files — just add your connection strings and query settings directly in the `env` block.

Add the following to your MCP client configuration (e.g. Claude Desktop `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "sqlserver": {
      "command": "sqlservermcp",
      "env": {
        "ConnectionStrings__MyDb": "Server=localhost;Database=MyDatabase;Trusted_Connection=True;TrustServerCertificate=True;",
        "Query__DefaultTop": "1000",
        "Query__MaxTop": "500000"
      }
    }
  }
}
```

### Environment Variable Format

| Variable | Description |
|----------|-------------|
| `ConnectionStrings__<name>` | SQL Server connection string. Add multiple with different names. |
| `Query__DefaultTop` | Default row limit for queries (default: `1000`) |
| `Query__MaxTop` | Maximum allowed row limit (default: `500000`) |

> **Note:** Use double underscores (`__`) as the separator — this is the .NET configuration convention for environment variables.

### Multiple Databases

You can configure multiple databases by adding multiple connection string variables:

```json
{
  "env": {
    "ConnectionStrings__Production": "Server=prod-server;Database=ProdDb;...",
    "ConnectionStrings__Staging": "Server=staging-server;Database=StagingDb;..."
  }
}
```

Then use the `ListDatabases` tool to see available names, and pass the database name to any query tool.

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

## Example

**Prompt:**

> *"Using the `MyDb` database, show me the top 5 customers by total order amount in 2025."*

**The AI generates and executes the query** by calling `SearchSchema` / `DescribeTable` to discover the tables, then passing this SQL to `RunQuery`:

```sql
SELECT TOP 5 c.CustomerId, c.Name, SUM(o.TotalAmount) AS TotalSpent
FROM dbo.Customers c
JOIN dbo.Orders o ON o.CustomerId = c.CustomerId
WHERE YEAR(o.OrderDate) = 2025
GROUP BY c.CustomerId, c.Name
ORDER BY TotalSpent DESC;
```

**Expected output:**

| CustomerId | Name        | TotalSpent |
|-----------:|-------------|-----------:|
| 1042       | Acme Corp   | 184500.00  |
| 778        | Globex Ltd  | 162300.50  |
| 2310       | Initech     | 149875.25  |
| 915        | Soylent Inc | 138420.00  |
| 604        | Umbrella Co | 121990.75  |

> **Disclaimer:** The accuracy of the generated SQL and the final output depends on the AI model you select in your MCP client. More capable models produce more reliable queries; weaker models may misinterpret the schema or the intent of the prompt.

## Safety

- Only `SELECT` queries are allowed — all write operations are blocked
- Queries are automatically row-limited to prevent runaway results
- Connection strings are configured server-side, never exposed to the AI client

## License

MIT
