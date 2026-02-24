# CLAUDE.md

This file provides guidance to Claude Code when working with n8n workflow templates in this repository.

## Agent Mode

Use **agent mode** (`/agent` or `claude --agent`) for all n8n workflow tasks. Agent mode enables autonomous multi-step execution with MCP tool access, which is required for building, validating, and deploying n8n workflows.

When operating in agent mode:
- Use MCP tools from `n8n-mcp` to search nodes, build workflows, validate, and deploy
- Follow the iterative pattern: search → configure → validate → fix → deploy
- Always validate workflows before marking them complete
- Think step-by-step through the workflow architecture before writing JSON

## MCP Server Configuration

This project uses the `n8n-mcp` MCP server for full n8n workflow automation. Configure it in your `.mcp.json`:

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["n8n-mcp"],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true",
        "N8N_API_URL": "https://your-n8n-instance.com",
        "N8N_API_KEY": "your-api-key"
      }
    }
  }
}
```

> Without `N8N_API_URL` and `N8N_API_KEY`, only documentation/node-lookup tools are available. With them, workflow management tools (create, update, validate, deploy, execute) are unlocked.

## Required Skills

Install the following n8n skills from [skills.sh](https://skills.sh/czlonkowski/n8n-skills):

```bash
npx skills add czlonkowski/n8n-skills/n8n-workflow-patterns
npx skills add czlonkowski/n8n-skills/n8n-mcp-tools-expert
npx skills add czlonkowski/n8n-skills/n8n-validation-expert
npx skills add czlonkowski/n8n-skills/n8n-expression-syntax
```

### Skill Descriptions

| Skill | Purpose | Activates When |
|-------|---------|----------------|
| **n8n-workflow-patterns** | 5 proven architectural patterns (webhook, HTTP API, database, AI agent, scheduled) with examples from 2,653+ templates | Building new workflows, choosing architecture |
| **n8n-mcp-tools-expert** | Expert guide for using n8n-mcp MCP tools — tool selection, parameter formats, common patterns | Searching nodes, validating configs, managing workflows |
| **n8n-validation-expert** | Interprets validation errors, handles false positives, auto-fixes with `n8n_autofix_workflow` | Validation failures, debugging workflow issues |
| **n8n-expression-syntax** | Correct `{{ }}` expression patterns, `$json`/`$node` variables, webhook data access (`$json.body`) | Writing expressions, fixing syntax errors |

## Agents

### 1. Workflow Builder Agent

**Role:** Create n8n workflows as valid JSON files.

**Process:**
1. Understand the automation goal and identify required triggers, actions, and data flow
2. Use `search_nodes` to find appropriate n8n nodes for each step
3. Use `get_node` (detail level: `standard` or `full`) to get configuration schemas
4. Select the correct workflow pattern from `n8n-workflow-patterns` skill
5. Build the workflow JSON with proper node configurations, connections, and expressions
6. Use `validate_workflow` or `validate_node` to verify correctness
7. Fix any validation errors using guidance from `n8n-validation-expert` skill
8. Save the final JSON to the appropriate template directory

**Workflow JSON Structure:**
```json
{
  "name": "Workflow Name",
  "nodes": [
    {
      "parameters": {},
      "type": "n8n-nodes-base.nodeType",
      "typeVersion": 1,
      "position": [x, y],
      "id": "uuid",
      "name": "Node Display Name",
      "credentials": {}
    }
  ],
  "connections": {
    "Source Node": {
      "main": [[{ "node": "Target Node", "type": "main", "index": 0 }]]
    }
  },
  "pinData": {},
  "settings": { "executionOrder": "v1" },
  "active": false,
  "tags": [],
  "meta": {}
}
```

**Rules:**
- Every workflow must have exactly one trigger node
- Node `type` must use the full qualified name (e.g., `n8n-nodes-base.httpRequest`, `@n8n/n8n-nodes-langchain.agent`)
- Connections must reference nodes by their `name` field
- Position nodes on a grid with ~250px horizontal spacing between steps
- Set `"active": false` by default — activation is a separate step
- Use `n8n-expression-syntax` skill for all `{{ }}` expressions
- Webhook data is accessed via `$json.body`, not `$json` directly

### 2. Workflow Debugger Agent

**Role:** Diagnose and fix broken or failing n8n workflows.

**Process:**
1. Load the workflow JSON from file or retrieve via `n8n_executions` for execution history
2. Run `validate_workflow` or `n8n_validate_workflow` (by ID) to get structured error output
3. Categorize errors: node configuration, missing connections, expression syntax, credential references
4. Apply fixes using guidance from `n8n-validation-expert` and `n8n-expression-syntax` skills
5. For common issues, use `n8n_autofix_workflow` for automated repair
6. Re-validate after each fix to confirm resolution
7. If connected to n8n instance, use `n8n_update_partial_workflow` for incremental updates
8. Test the fixed workflow with `n8n_test_workflow` if instance access is available

**Common Issues & Fixes:**
- **Missing required fields:** Use `get_node` with `full` detail to check all required properties for the selected operation
- **Expression errors:** Verify `{{ }}` syntax, check variable paths, confirm `$json.body` for webhook data
- **Connection gaps:** Ensure every non-trigger node has at least one input connection
- **Type mismatches:** Check `typeVersion` matches the property schema version
- **Validation false positives:** Consult `n8n-validation-expert` before making unnecessary changes

**Iteration Pattern:**
```
validate → identify errors → fix → re-validate → repeat until clean
```
Average: 23s analysis, 58s fixing per validation cycle.

### 3. Template Deployer Agent

**Role:** Deploy workflow templates to a live n8n instance.

**Process:**
1. Read the workflow JSON from the repository
2. Use `n8n_create_workflow` to push the workflow to the n8n instance
3. Run `n8n_validate_workflow` against the deployed workflow
4. If validation passes, optionally activate with `n8n_update_partial_workflow` (operation: `activateWorkflow`)
5. Confirm deployment with `n8n_executions` to verify the workflow is registered

## Key MCP Tools Reference

| Tool | Purpose |
|------|---------|
| `search_nodes` | Find nodes by keyword |
| `get_node` | Get node info (detail levels: minimal, standard, full) |
| `validate_node` | Validate a single node config |
| `validate_workflow` | Validate entire workflow JSON |
| `n8n_create_workflow` | Create workflow on n8n instance |
| `n8n_update_partial_workflow` | Incremental updates (17 operation types) |
| `n8n_validate_workflow` | Validate workflow by ID on instance |
| `n8n_autofix_workflow` | Auto-fix common issues |
| `n8n_deploy_template` | Deploy a template to n8n |
| `n8n_workflow_versions` | Version history and rollback |
| `n8n_test_workflow` | Execute workflow for testing |
| `n8n_executions` | View execution history |
| `search_templates` | Search 2,709 templates (keyword, by_nodes, by_task, by_metadata) |
| `get_template` | Get full template details |
| `tools_documentation` | Meta-docs for all MCP tools |
| `ai_agents_guide` | AI agent workflow patterns |

## Repository Structure

```
n8n-templates/
├── claude.md                  # This file
├── readme.md                  # Project overview
├── ai-overview-analyzer/      # AI Overview analysis workflow
├── ai-overview-optimizer/     # AI Overview optimization workflow
├── ai-powered-seo-team/       # Multi-agent SEO workflow
├── get-google-search-console-data/
├── google-index-checker/
├── gsc-ai-seo-writer/
├── keyword-rank-tracker/
├── mailing-list-analysis/
├── report-generator/
├── seo-data-analyst/
├── serp-analysis/
├── tracked-keyword-performance-report-generator/
├── traffic-performance-analysis/
└── website-seo-audit/
```

Each template directory contains the workflow JSON and a readme describing its purpose, setup, and required credentials.

## Workflow Development Guidelines

1. **Always search before building** — Use `search_templates` to check if a similar workflow already exists among the 2,709 templates
2. **Start with patterns** — Pick the right architectural pattern from `n8n-workflow-patterns` before writing any JSON
3. **Validate continuously** — Run validation after every significant change, not just at the end
4. **Never edit production directly** — Copy workflows before modifying; test in development first
5. **Use incremental updates** — Prefer `n8n_update_partial_workflow` over full workflow replacement
6. **Name nodes descriptively** — Node names appear in logs and error messages; make them meaningful
7. **Document credentials** — List required credentials in the template's readme, never store secrets in JSON

## Cross-Skill Integration Flow

```
n8n-workflow-patterns  →  Identify architecture
         ↓
n8n-mcp-tools-expert   →  Find and configure nodes
         ↓
n8n-expression-syntax  →  Write data mappings
         ↓
n8n-validation-expert  →  Validate and fix
```

Skills activate automatically based on context. They are designed to work together — the MCP tools expert finds nodes, the node configuration skill sets them up, the expression syntax skill handles data mapping, and the validation expert catches errors.
