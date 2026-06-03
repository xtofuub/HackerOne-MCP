# HackerOne MCP Server

> **Disclaimer:** This is an unofficial, community-built project. It is not affiliated with, endorsed by, or maintained by HackerOne. "HackerOne" is a trademark of HackerOne, Inc. This project simply integrates with their publicly documented [Hacker API](https://api.hackerone.com/hacker-resources/).

MCP server that gives Claude Code (or any MCP client) access to your HackerOne programs, scope, reports, earnings, and the public hacktivity feed via the HackerOne Hacker API — plus submitting new reports.

Every tool in this server is verified against the live API. See [API coverage](#api-coverage) for what the Hacker API does and does not expose.

## Setup

### 1. Get your HackerOne API token

Generate one at **[hackerone.com/settings/api_token/edit](https://hackerone.com/settings/api_token/edit)**. Your `H1_USERNAME` is your HackerOne username; `H1_API_TOKEN` is the token value.

Quick sanity check (should return HTTP 200):

```bash
curl -s -o /dev/null -w "%{http_code}\n" \
  "https://api.hackerone.com/v1/hackers/programs" \
  -u "YOUR_USERNAME:YOUR_TOKEN" -H "Accept: application/json"
```

### 2. Install and build

```bash
git clone https://github.com/xtofuub/Hackerone-MCP.git
cd Hackerone-MCP
npm install
npm run build
```

### 3. Provide credentials

The server reads `H1_USERNAME` and `H1_API_TOKEN` from the environment, and falls back to a `.env` file in the project root if they are not already set. Pick whichever is convenient:

**Option A — `.env` file (simplest):**

```bash
cp .env.example .env
# edit .env and fill in your username + token
```

**Option B — pass env directly when registering with Claude Code:**

```bash
claude mcp add hackerone \
  -e H1_USERNAME=your-username \
  -e H1_API_TOKEN=your-api-token \
  -s user \
  -- node /absolute/path/to/Hackerone-MCP/dist/index.js
```

Or add manually to `~/.claude.json`:

```json
{
  "mcpServers": {
    "hackerone": {
      "command": "node",
      "args": ["/absolute/path/to/Hackerone-MCP/dist/index.js"],
      "env": {
        "H1_USERNAME": "your-username",
        "H1_API_TOKEN": "your-api-token"
      }
    }
  }
}
```

> **Security:** `.env` and `run.bat` are gitignored so your token never gets committed. Never paste a real token into a tracked file.

### 4. Verify

```bash
claude
> /mcp
# You should see "hackerone" listed with 13 tools
```

## Tools

### Read

| Tool | Description |
|------|-------------|
| `search_reports` | Search and filter your reports by keyword, program, severity, or state |
| `get_report` | Get full report details including CVSS vector, bounty amounts, and attachments |
| `get_report_with_conversation` | Get a report with its triage conversation thread |
| `get_report_activities` | Get activity timeline (comments, state changes, bounties) |
| `list_programs` | List all bug bounty programs you have access to (auto-paginates) |
| `get_program_details` | Get single program info: policy, response times, metrics |
| `get_program_scope` | Get all in-scope assets for a program (auto-paginates) |
| `get_program_weaknesses` | Get accepted CWE/weakness types for a program (auto-paginates) |
| `get_earnings` | Get your bounty earnings history (amounts, dates, programs) |
| `get_balance` | Get your current unpaid bounty balance |
| `analyze_report_patterns` | Analyze your hunting patterns (severity distribution, top programs, weakness types) |
| `search_disclosed_reports` | Search publicly disclosed reports on hacktivity — great for recon and learning |

### Write

| Tool | Description |
|------|-------------|
| `submit_report` | Submit a new vulnerability report to a program |

## API coverage

The HackerOne **Hacker API** (`/hackers/...`) is read-heavy. This server only ships tools backed by endpoints that actually work with a hacker token:

- ✅ Programs, scope, weaknesses, your reports, single report + activities, earnings, balance, hacktivity, report submission.
- ❌ **No self-profile endpoint.** `GET /hackers/me` returns 401 — reputation/signal/rank are not exposed via the Hacker API, so there is no `get_hacker_profile` tool.
- ❌ **No comment/close-via-API.** `POST /hackers/reports/{id}/activities` returns 401, so adding comments or withdrawing reports is not supported by the Hacker API. Do those in the web UI.

If HackerOne adds these endpoints, the tools are easy to re-add — the client layer is in `src/h1client.ts`.

## Usage Examples

**Check program details and scope before hunting:**
```
Show me the security program details and list its in-scope assets.
```

**Draft a report matching your style:**
```
Find my resolved reports and use the same structure to draft a new report for this finding.
```

**Research what gets paid:**
```
Search disclosed reports for SSRF — what programs paid and how much?
```

**Submit a report:**
```
Submit this finding to the <program> program with high severity. Here's my writeup: [paste]
```

**Analyze patterns:**
```
Analyze my report patterns — what severity gets resolved most?
```

**Track earnings:**
```
Show my recent bounty earnings and current balance.
```

## How It Works

- Connects to the [HackerOne Hacker API v1](https://api.hackerone.com/hacker-resources/) using your personal API token over HTTP Basic Auth.
- Runs locally over stdio — your credentials never leave your machine.
- Auto-paginates programs, scope, and weakness endpoints so nothing gets silently truncated.
- Uses server-side API filters where available (program, severity, state) for faster searches.
- Built-in retry with exponential backoff for rate-limit (429) handling.
- 60-second response cache to reduce redundant API calls.

## License

MIT
