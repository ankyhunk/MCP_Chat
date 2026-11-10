# MCP Chat

MCP Chat is a command-line chat app that talks to Claude through the Anthropic API and uses the **Model Context Protocol (MCP)** to give Claude access to documents, tools and prompts provided by an MCP server.

You can mention documents with `@`, run prompt commands with `/`, and Claude can read and edit documents itself through MCP tools.

## How it works

```
You ──> CLI (prompt-toolkit) ──> Claude (Anthropic API)
                 │                      │ tool calls
                 └──── MCP client ──────┴──> MCP server (mcp_server.py)
                                              documents · tools · resources · prompts
```

The MCP server (`mcp_server.py`) holds a small set of in-memory documents and exposes:

| Type | Name | What it does |
|---|---|---|
| Tool | `read_doc_contents` | Returns a document's text |
| Tool | `edit_document` | Replaces text in a document |
| Resource | `docs://documents` | Lists all document IDs (JSON) |
| Resource | `docs://documents/{doc_id}` | Returns one document's text |
| Prompt | `format` | Asks Claude to rewrite a document in Markdown |

Sample documents: `deposition.md`, `report.pdf`, `financials.docx`, `outlook.pdf`, `plan.md`, `spec.txt`.

## Prerequisites

- Python 3.10+
- An Anthropic API key
- [uv](https://github.com/astral-sh/uv) (recommended)

## Setup

### 1. Create the `.env` file

Create a `.env` file in the project root (it's git-ignored, so your key stays private):

```
CLAUDE_MODEL="claude-sonnet-5-5"   # any Claude model ID
ANTHROPIC_API_KEY=""               # your Anthropic API key
USE_UV=1                           # 1 if you run the project with uv, 0 if not
```

### 2. Install and run

**With uv (recommended):**

```bash
uv venv
source .venv/bin/activate   # On Windows: .venv\Scripts\activate
uv pip install -e .
uv run main.py
```

**Without uv** (set `USE_UV=0` in `.env`):

```bash
python -m venv .venv
source .venv/bin/activate   # On Windows: .venv\Scripts\activate
pip install "anthropic<1" python-dotenv prompt-toolkit "mcp[cli]>=1.8.0,<2"
python main.py
```

To connect extra MCP servers, pass their scripts as arguments:

```bash
uv run main.py my_other_server.py
```

## Usage

**Chat:** type a message and press Enter.

**Mention a document** with `@` to include its contents in your question:

```
> Tell me about @deposition.md
```

**Run a prompt command** with `/` (press Tab to auto-complete):

```
> /format plan.md
```

Claude can also call the `read_doc_contents` and `edit_document` tools on its own when your question needs them.

## Project structure

```
main.py           # Entry point: loads .env, starts the MCP client(s) and the CLI
mcp_server.py     # MCP server: documents, tools, resources and prompts
mcp_client.py     # MCP client wrapper (run it directly to list the server's tools)
core/
  claude.py       # Anthropic API calls
  chat.py         # Chat loop and tool-call handling
  cli_chat.py     # @mentions and /commands
  cli.py          # Terminal UI with auto-complete
  tools.py        # Connects MCP tools to Claude
```

## Development

- **Add documents:** edit the `docs` dictionary in `mcp_server.py`.
- **Add a tool, resource or prompt:** add a function with `@mcp.tool`, `@mcp.resource` or `@mcp.prompt` in `mcp_server.py`.
- **Check the server:** `uv run mcp_client.py` connects to the server and prints its tools.
- **Not done yet:** a `summarize` prompt is planned but not written (see the TODO in `mcp_server.py`).
