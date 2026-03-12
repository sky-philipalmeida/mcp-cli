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

## Exit Codes

- `0`: Success
- `1`: Client error (bad args, missing config)
- `2`: Server error (tool failed)
- `3`: Network error

## Common Workflows

### Update PR Description

When updating a PR description, always use single quotes around the JSON argument and properly escape newlines with `\n`:

```bash
mcp-cli call github update_pull_request '{
  "owner": "org-name",
  "repo": "repo-name",
  "pullNumber": 123,
  "body": "## Overview\n\nYour description here\n\n## Changes\n- Item 1\n- Item 2"
}'
```

**Best Practices:**
- Use `\n` for newlines in the body text
- Single quotes around the entire JSON object
- Double quotes for JSON property names and string values
- For long descriptions, keep them properly formatted with sections
- Exit code 0 indicates success, even if terminal output appears garbled

### Read PR Information

To get PR details before updating:

```bash
# Get full PR info
mcp-cli call github pull_request_read '{
  "owner": "org-name",
  "repo": "repo-name",
  "pullNumber": 123,
  "method": "get"
}'

# Get PR diff
mcp-cli call github pull_request_read '{
  "owner": "org-name",
  "repo": "repo-name",
  "pullNumber": 123,
  "method": "get_diff"
}'

# Get files changed
mcp-cli call github pull_request_read '{
  "owner": "org-name",
  "repo": "repo-name",
  "pullNumber": 123,
  "method": "get_files"
}'
```

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
