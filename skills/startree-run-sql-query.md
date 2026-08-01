---
name: Run a SQL query on StarTree Cloud
description: >-
  Execute a read-only SQL query against a StarTree Cloud (Apache Pinot) cluster
  and read back the result table, using Bearer JWT auth and the correct
  multi-tenant workspace header.
api: openapi/startree-query-openapi.json
operations:
  - executeQuery
---

# Run a SQL query on StarTree Cloud

Use this skill to run real-time analytics SQL against a StarTree Cloud Broker.

## Prerequisites

- A Bearer **JWT** token generated from the **API Access** section of the
  StarTree Cloud Console.
- Your Broker base URL (e.g. `https://broker.pinot.<cluster>.cp.s7e.startree.cloud`).
- For multi-tenant / Free Tier clusters: your **workspace id** (`ws_...`).

## Steps

1. **Authenticate.** Send the token on every request:
   `Authorization: Bearer <JWT>`. See `authentication/startree-authentication.yml`.
2. **Set the tenant header (multi-tenant only).** Pass the workspace id in the
   `database` header (e.g. `database: ws_2kc8e2dnzzb0`). Dedicated clusters skip this.
   See `conventions/startree-conventions.yml`.
3. **Execute the query** — call `executeQuery` (`POST /query/sql`) with a JSON body:
   `{ "sql": "SELECT * FROM my_table LIMIT 10" }`. Keep queries **read-only**
   (`SELECT` / `WITH ... SELECT`); control result size with SQL `LIMIT`.
4. **Read the result.** On `200`, parse `resultTable.dataSchema.columnNames` +
   `columnDataTypes` for the schema and `resultTable.rows` for the rows.
5. **Handle failures** by HTTP status — a `401/403` means the token or workspace
   header is wrong; retries are safe because the query is idempotent (read-only).

## Notes

- This is the same operation the official `mcp-pinot` MCP server exposes as its
  `query` tool (`mcp/startree-mcp.yml`).
