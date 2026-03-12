---
name: mcp-cli
description: Interface for MCP (Model Context Protocol) servers via CLI. Use when you need to interact with external tools, APIs, or data sources through MCP servers.
---

# MCP-CLI

Access MCP servers through the command line. MCP enables interaction with external systems like GitHub, filesystems, databases, and APIs.

## Commands

| Command | Output |
|---------|--------|
| `mcp-cli` | List all servers and tools |
| `mcp-cli info <server>` | Show server tools and parameters |
| `mcp-cli info <server> <tool>` | Get tool JSON schema |
| `mcp-cli grep "<pattern>"` | Search tools by name |
| `mcp-cli call <server> <tool>` | Call tool (reads JSON from stdin if no args) |
| `mcp-cli call <server> <tool> '<json>'` | Call tool with arguments |

**Both formats work:** `<server> <tool>` or `<server>/<tool>`

## Workflow

1. **Discover**: `mcp-cli` → see available servers
2. **Explore**: `mcp-cli info <server>` → see tools with parameters
3. **Inspect**: `mcp-cli info <server> <tool>` → get full JSON schema
4. **Execute**: `mcp-cli call <server> <tool> '<json>'` → run with arguments

## Examples

```bash
# List all servers
mcp-cli

# With descriptions  
mcp-cli -d

# See server tools
mcp-cli info filesystem

# Get tool schema (both formats work)
mcp-cli info filesystem read_file
mcp-cli info filesystem/read_file

# Call tool
mcp-cli call filesystem read_file '{"path": "./README.md"}'

# Pipe from stdin (no '-' needed!)
cat args.json | mcp-cli call filesystem read_file

# Search for tools
mcp-cli grep "*file*"

# Output is raw text (pipe-friendly)
mcp-cli call filesystem read_file '{"path": "./file"}' | head -10
```

## Advanced Chaining

```bash
# Chain: search files → read first match
mcp-cli call filesystem search_files '{"path": ".", "pattern": "*.md"}' \
  | head -1 \
  | xargs -I {} mcp-cli call filesystem read_file '{"path": "{}"}'

# Loop: process multiple files
mcp-cli call filesystem list_directory '{"path": "./src"}' \
  | while read f; do mcp-cli call filesystem read_file "{\"path\": \"$f\"}"; done

# Conditional: check before reading
mcp-cli call filesystem list_directory '{"path": "."}' \
  | grep -q "README" \
  && mcp-cli call filesystem read_file '{"path": "./README.md"}'

# Multi-server aggregation
{
  mcp-cli call github search_repositories '{"query": "mcp", "per_page": 3}'
  mcp-cli call filesystem list_directory '{"path": "."}'
}

# Save to file
mcp-cli call github get_file_contents '{"owner": "x", "repo": "y", "path": "z"}' > output.txt
```

**Note:** `call` outputs raw text content directly (no jq needed for text extraction)

## Options

| Flag | Purpose |
|------|---------|
| `-d` | Include descriptions |
| `-c <path>` | Specify config file |

## Common Errors

| Wrong Command | Error | Fix |
|---------------|-------|-----|
| `mcp-cli server tool` | AMBIGUOUS_COMMAND | Use `call server tool` or `info server tool` |
| `mcp-cli run server tool` | UNKNOWN_SUBCOMMAND | Use `call` instead of `run` |
| `mcp-cli list` | UNKNOWN_SUBCOMMAND | Use `info` instead of `list` |
| `mcp-cli call server` | MISSING_ARGUMENT | Add tool name |
| `mcp-cli call server tool {bad}` | INVALID_JSON | Use valid JSON with quotes |

## Exit Codes

- `0`: Success
- `1`: Client error (bad args, missing config)
- `2`: Server error (tool failed)
- `3`: Network error


## Available Servers & Operations


### github

#### Pull Operations

**pull_request_read** - Get information on a specific pull request in GitHub repository.
```json
{
  "method": "string (required) - Action to specify what pull request data needs to be retrieved from GitHub. \nPossible options: \n 1. get - Get details of a specific pull request.\n 2. get_diff - Get the diff of a pull request.\n 3. get_status - Get combined commit status of a head commit in a pull request.\n 4. get_files - Get the list of files changed in a pull request. Use with pagination parameters to control the number of results returned.\n 5. get_review_comments - Get review threads on a pull request. Each thread contains logically grouped review comments made on the same code location during pull request reviews. Returns threads with metadata (isResolved, isOutdated, isCollapsed) and their associated comments. Use cursor-based pagination (perPage, after) to control results.\n 6. get_reviews - Get the reviews on a pull request. When asked for review comments, use get_review_comments method.\n 7. get_comments - Get comments on a pull request. Use this if user doesn't specifically want review comments. Use with pagination parameters to control the number of results returned.\n 8. get_check_runs - Get check runs for the head commit of a pull request. Check runs are the individual CI/CD jobs and checks that run on the PR.\n",
  "owner": "string (required) - Repository owner",
  "page": "number (optional) - Page number for pagination (min 1)",
  "perPage": "number (optional) - Results per page for pagination (min 1, max 100)",
  "pullNumber": "number (required) - Pull request number",
  "repo": "string (required) - Repository name"
}
```

**pull_request_review_write** - Create and/or submit, delete review of a pull request.
```json
{
  "body": "string (optional) - Review comment text",
  "commitID": "string (optional) - SHA of commit to review",
  "event": "string (optional) - Review action to perform.",
  "method": "string (required) - The write operation to perform on pull request review.",
  "owner": "string (required) - Repository owner",
  "pullNumber": "number (required) - Pull request number",
  "repo": "string (required) - Repository name"
}
```


### - delete_pending: Delete an existing pending review of a pull request. This requires that a pending review exists for the current user on the specified pull request.


### - create: Create a new review of a pull request. If "event" parameter is provided, the review is submitted. If "event" is omitted, a pending review is created.


### - submit_pending: Submit an existing pending review of a pull request. This requires that a pending review exists for the current user on the specified pull request. The "body" and "event" parameters are used when submitting the review.


### Available methods:

