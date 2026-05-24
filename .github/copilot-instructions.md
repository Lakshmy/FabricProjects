# Copilot Instructions for FabricProjects

This repository contains Microsoft Fabric skills, agents, and reusable AI assistant instructions sourced from [microsoft/skills-for-fabric](https://github.com/microsoft/skills-for-fabric) (v0.3.1).

## Architecture

The codebase follows a three-layer model: **Agents → Skills → Common**.

- **`skills-for-fabric/agents/`** — Cross-workload orchestration agents (FabricDataEngineer, FabricAdmin, FabricAppDev, FabricMigrationEngineer). Start here for multi-workload tasks like medallion architecture or migration.
- **`skills-for-fabric/skills/`** — Workload-specific skills organized by `{workload}-{role}-cli` naming:
  - `-authoring-cli`: Creating/managing artifacts (REST APIs, DDL, ingestion)
  - `-consumption-cli`: Read-only exploration and queries
  - `-operations-cli`: Performance diagnostics and health monitoring
- **`skills-for-fabric/common/`** — Shared reference docs (`*-CORE.md`) included by multiple skills. Edit here to change behavior across workloads.
- **`skills-for-fabric/plugins/`** — Bundle definitions for Copilot CLI plugin marketplace (fabric-skills, fabric-authoring, fabric-consumption, fabric-operations).

## Fabric Workloads Covered

| Workload | Skills prefix | Primary query language |
|---|---|---|
| Data Warehouse | `sqldw-*` | T-SQL (limited surface area) |
| Lakehouse / Spark | `spark-*` | PySpark, Delta Lake |
| Eventhouse / KQL DB | `eventhouse-*` | KQL |
| Eventstreams | `eventstream-*` | REST API (graph topology) |
| Dataflows Gen2 | `dataflows-*` | Power Query M via REST |
| Activator / Reflex | `activator-*` | REST API |
| Power BI Semantic Models | `powerbi-*` | DAX, XMLA |
| Catalog Search | `search-*` | REST API (`POST /v1/catalog/search`) |
| Migration | `databricks-migration`, `synapse-migration`, `hdinsight-migration` | Varies |

## Authentication

All Fabric operations require Azure AD tokens:

```bash
az login
az account get-access-token --resource https://api.fabric.microsoft.com
# For SQL endpoints:
az account get-access-token --resource https://database.windows.net
# For KQL/Kusto:
az account get-access-token --resource https://kusto.kusto.windows.net/.default
```

## Key Conventions

### SKILL.md Structure
Every skill has a `SKILL.md` with:
1. YAML frontmatter (`name`, `description`)
2. Update-check notice block at the top
3. Critical notes for workspace/item lookup via JMESPath filtering
4. Must/Prefer/Avoid sections
5. Examples with code or prompt/response pairs

### Naming Conventions
- End-to-end: `e2e-{task-name}`
- Developer: `{workload}-authoring-{access_method}`
- Consumer: `{workload}-consumption-{access_method}`
- Operations: `{workload}-operations-{access_method}`

### Workspace and Item Discovery
Always use this two-step pattern:
1. List all workspaces, then filter by name with JMESPath to get the workspace ID
2. List all items of a type in that workspace, then filter by name with JMESPath

### Critical Rules
- Use Delta Lake format for all Lakehouse tables
- Always include time filters in KQL queries (`where Timestamp > ago(...)`)
- Use `has` over `contains` for indexed KQL string search
- Use idempotent KQL commands (`.create-merge table`, `.create-or-alter function`)
- Use `TOP`/`LIMIT` on exploratory T-SQL queries
- Eventstream node names: alphanumeric PascalCase, 3-63 characters
- Never hardcode credentials — use Key Vault or environment variables
- Prefer medallion architecture (Bronze → Silver → Gold)
- Parameterize notebooks and pipelines for reusability

## Build and Test

```bash
# Run all tests
pytest

# Run semantic tests only
pytest -m semantic

# Run integration tests only
pytest -m integration

# Run a single test file
pytest tests/test_specific.py

# Coverage gap report
python tests/coverage_gap_report.py
```

## MCP Server

The repo includes a Power BI Query MCP server configuration (`.mcp.json`):
```json
{
  "mcpServers": {
    "PowerBIQuery": {
      "type": "http",
      "url": "https://api.fabric.microsoft.com/v1/mcp/powerbi",
      "tools": ["ExecuteQuery"]
    }
  }
}
```

Additional MCP servers can be registered via `skills-for-fabric/mcp-setup/register-fabric-mcp.ps1`.

## Reference Links

- [Fabric REST APIs](https://learn.microsoft.com/en-us/rest/api/fabric/articles/)
- [T-SQL surface area](https://learn.microsoft.com/en-us/fabric/data-warehouse/tsql-surface-area)
- [MCP setup guide](skills-for-fabric/mcp-setup/README.md)
- [Prompt examples](skills-for-fabric/prompt_examples/)
