# Overleaf MCP Server

An MCP (Model Context Protocol) server that provides access to Overleaf projects via Git integration. This allows Claude and other MCP clients to read LaTeX files, analyze document structure, extract content, and write files from and to Overleaf projects.

## Features

- 📄 **File Management**: List, read, and write files from and to Overleaf projects
- 📋 **Document Structure**: Parse LaTeX sections and subsections
- 🔍 **Content Extraction**: Extract specific sections by title
- 📊 **Project Summary**: Get overview of project status and structure
- 🏗️ **Multi-Project Support**: Manage multiple Overleaf projects

## Quick Start (recommended)

No clone, no `npm install`. Add this block to your Claude Desktop config and restart Claude Desktop.

**Config file location**

| OS      | Path |
|---------|------|
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| macOS   | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Linux   | `~/.config/claude/claude_desktop_config.json` |

**macOS / Linux**

```json
{
  "mcpServers": {
    "overleaf": {
      "command": "npx",
      "args": ["-y", "@mjyoo2/overleaf-mcp"],
      "env": {
        "OVERLEAF_PROJECT_ID": "YOUR_OVERLEAF_PROJECT_ID",
        "OVERLEAF_GIT_TOKEN": "YOUR_OVERLEAF_GIT_TOKEN"
      }
    }
  }
}
```

**Windows** — Claude Desktop on Windows needs `cmd /c` to find `npx`:

```json
{
  "mcpServers": {
    "overleaf": {
      "command": "cmd",
      "args": ["/c", "npx", "-y", "@mjyoo2/overleaf-mcp"],
      "env": {
        "OVERLEAF_PROJECT_ID": "YOUR_OVERLEAF_PROJECT_ID",
        "OVERLEAF_GIT_TOKEN": "YOUR_OVERLEAF_GIT_TOKEN"
      }
    }
  }
}
```

Restart Claude Desktop. The `overleaf` tools should appear in the 🔧 menu.

## Getting Overleaf Credentials

1. **Project ID** — open your Overleaf project; the ID is in the URL: `https://www.overleaf.com/project/[PROJECT_ID]`
2. **Git Token** — Overleaf → Account Settings → Git Integration → "Create Token"

## Configuration Reference

The server picks the **first** matching configuration source:

1. **Env vars (single project)** — `OVERLEAF_PROJECT_ID` + `OVERLEAF_GIT_TOKEN`. Optional: `OVERLEAF_PROJECT_NAME` for the display name.
2. **Token from a file** — set `OVERLEAF_PROJECT_ID` together with `OVERLEAF_GIT_TOKEN_FILE=/path/to/token.txt` (instead of `OVERLEAF_GIT_TOKEN`). Useful when you don't want the token in the Claude Desktop JSON. The file is read once at startup and any trailing whitespace/newline is trimmed.
3. **Multi-project file** — `OVERLEAF_PROJECTS_CONFIG=/absolute/path/projects.json`.
4. **User config dir** — `projects.json` in:
   - Windows: `%APPDATA%\overleaf-mcp\projects.json`
   - macOS / Linux: `$XDG_CONFIG_HOME/overleaf-mcp/projects.json` (defaults to `~/.config/overleaf-mcp/projects.json`)
5. **Working directory** — `./projects.json`
6. **Package directory** — `projects.json` next to the server script (legacy, for clone-based installs).

When env vars are set and a file is also present, env vars win and a notice is logged to stderr so the shadowing is visible.

### `projects.json` schema (multi-project)

```json
{
  "projects": {
    "default": {
      "name": "Main Paper",
      "projectId": "...",
      "gitToken": "olp_..."
    },
    "paper2": {
      "name": "Second Paper",
      "projectId": "...",
      "gitToken": "olp_..."
    }
  }
}
```

Then specify the project in tool calls: `projectName: "paper2"`.

## Local Development

If you want to hack on the server itself:

```bash
git clone https://github.com/mjyoo2/OverleafMCP.git
cd OverleafMCP
npm install
cp projects.example.json projects.json   # then edit it
node overleaf-mcp-server.js               # or wire it into Claude Desktop with an absolute path
```

## Available Tools

### `list_projects`
List all configured projects.

### `list_files`
List files in a project (default: .tex files).
- `extension`: File extension filter (optional)
- `projectName`: Project identifier (optional, defaults to "default")

### `read_file`
Read a specific file from the project.
- `filePath`: Path to the file (required)
- `projectName`: Project identifier (optional)

### `get_sections`
Get all sections from a LaTeX file.
- `filePath`: Path to the LaTeX file (required)
- `projectName`: Project identifier (optional)

### `get_section_content`
Get content of a specific section.
- `filePath`: Path to the LaTeX file (required)
- `sectionTitle`: Title of the section (required)
- `projectName`: Project identifier (optional)

### `status_summary`
Get a comprehensive project status summary.
- `projectName`: Project identifier (optional)

### `write_file`
Write the full content of a file to the project.
- `filePath`: Path to the file (required)
- `content`: Content to write to the file (required)
- `commitMessage`: Commit message (required)
- `projectName`: Project identifier (optional)

### `write_section`
Write the content of a specific section to the project.
- `filePath`: Path to the file (required)
- `sectionTitle`: Title of the section (required)
- `newContent`: Replacement content for the section, including the section heading (required)
- `commitMessage`: Commit message (required)
- `projectName`: Project identifier (optional)

## Usage Examples

```
# List all projects
Use the list_projects tool

# Get project overview
Use status_summary tool

# Read main.tex file
Use read_file with filePath: "main.tex"

# Get Introduction section
Use get_section_content with filePath: "main.tex" and sectionTitle: "Introduction"

# List all sections in a file
Use get_sections with filePath: "main.tex"

# Write the full content of a file to the project
Use write_file with filePath: "main.tex", content: "...", commitMessage: "..."

# Write the content of a specific section to the project
Use write_section with filePath: "main.tex", sectionTitle: "Introduction", newContent: "\\section{Introduction}\n...", commitMessage: "..."
```

## Security Notes

- The Overleaf Git token grants full read/write access to your project — treat it like a password.
- Prefer `OVERLEAF_GIT_TOKEN_FILE` over inlining the token in the Claude Desktop JSON if your config file is backed up or synced.
- `projects.json` is `.gitignore`d in this repo. Never commit real project IDs or Git tokens.
- File paths supplied through MCP tool calls are restricted to the cloned project directory; `..` traversal and absolute paths are rejected.

## License

MIT License
