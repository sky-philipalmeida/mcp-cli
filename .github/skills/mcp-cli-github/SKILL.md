---
name: mcp-cli-github
description: Interface for MCP (Model Context Protocol) servers via CLI. Use when you need to interact with external tools, APIs, or data sources through MCP servers.
---

# MCP-CLI

Access MCP servers through the command line. MCP enables interaction with external systems like GitHub, filesystems, databases, and APIs.

## Commands

| Command | Output |
|---------|--------|
| `mcp-cli call <server> <tool>` | Call tool (reads JSON from stdin if no args) |
| `mcp-cli call <server> <tool> '<json>'` | Call tool with arguments |

**Both formats work:** `<server> <tool>` or `<server>/<tool>`

### Executing call instructions

**CRITICAL**: Always execute as a **single-line command**. Do not use multi-line shell input, as it causes terminal parsing issues.

**Correct Format:**
```bash
mcp-cli call github update_pull_request '{
  "owner": "org-name",
  "repo": "repo-name",
  "pullNumber": 123,
  "body": "## Context\n\nDescription here\n\n## Purpose\n\nMore text"
}'
```

**Best Practices:**
- **Single-line execution**: Pass the entire JSON as one command argument (can span visual lines but must be one shell command)
- Use `\n` for newlines in body text (not actual line breaks)
- Single quotes `'` around the entire JSON object
- Double quotes `"` for all JSON property names and values
- Escape special characters that might break shell parsing
- Exit code `0` indicates success (even if output appears garbled)
- Verify success with: `echo $?` (should return `0`)

**Common Mistakes to Avoid:**
- ❌ Breaking command into multi-line shell input (causes parsing errors)
- ❌ Using double quotes around the JSON object
- ❌ Forgetting to escape newlines with `\n`
- ❌ Using actual newlines instead of `\n` in the body text

## Exit Codes

- `0`: Success
- `1`: Client error (bad args, missing config)
- `2`: Server error (tool failed)
- `3`: Network error

## Available Servers & Tools


### Server: github

#### Pull Tools

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

#### Update Tools

**update_pull_request** - Update an existing pull request in a GitHub repository.
```json
{
  "base": "string (optional) - New base branch name",
  "body": "string (optional) - New description",
  "draft": "boolean (optional) - Mark pull request as draft (true) or ready for review (false)",
  "maintainer_can_modify": "boolean (optional) - Allow maintainer edits",
  "owner": "string (required) - Repository owner",
  "pullNumber": "number (required) - Pull request number to update",
  "repo": "string (required) - Repository name",
  "reviewers": "array (optional) - GitHub usernames to request reviews from",
  "state": "string (optional) - New state",
  "title": "string (optional) - New title"
}
```
