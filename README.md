# RichBinder MCP — connect your documents to AI agents

RichBinder is a document binder app for Windows. Its built-in MCP server lets an AI agent search the documents you keep in RichBinder, read the matching passages and quote them. You decide in the app which binders an agent may read, and whether it may change anything at all.

This repository is the setup guide. It contains configuration examples and documentation only — no executable code.

## Before you start

1. Install RichBinder from the Microsoft Store on Windows 10 or 11: https://apps.microsoft.com/detail/9ndj4bjf44wg
2. Start RichBinder and let it build its search index.
3. Use an edition that includes AI search: the AI edition, or the 30-day trial of it. With the Free edition the server starts but its tools answer with an edition error.
4. Keep the RichBinder app running. The MCP server talks to the running app; if the app is off, the tools answer with an "app off" error instead of results.

The server is started by one command, `rbctl mcp` (the program file is `rbctl.exe`). `rbctl.exe` is installed together with the RichBinder app. It is not downloaded by this guide and it is not in this repository.

## Approve the connection

The first time an AI agent connects, the connection waits for you. Approve it in the app: **RichBinder Settings (the gear button at the top right of the RichBinder window) › AI apps**. There you also choose which binders the agent may read. An agent can never approve itself.

## What the agent can do

### Read (always read-only)

| Tool | What it does |
|---|---|
| `state` | Shows the edition, whether this connection is approved, the index status and the next step if something is blocked. Works before approval. |
| `guide` | Returns the RichBinder usage guide. |
| `search` | Finds matches in the binders this connection may read. The same hits are listed in the RichBinder search window, so you see what the agent sees. |
| `fetch` | Reads a passage of a matched document so the agent can quote it. |
| `check_decision` | Looks up records in your documents that relate to a decision text and returns them with citations. It does not judge; the agent reads the records and decides. |
| `needs_list`, `needs_get`, `continue` | Read open-question records and the notes left for continuing earlier work. |

### Show on screen

| Tool | What it does |
|---|---|
| `open` | Opens one search hit on your screen in the RichBinder app. It changes nothing and returns no document text. |

### Change (off unless you turn it on)

These tools work only on a connection where you turned the matching permission on in the app. Otherwise the app refuses the call.

| Tool | What it does |
|---|---|
| `needs_create`, `evidence_link`, `handoff_write`, `needs_close_request` | Add an open-question record, propose evidence for it, write a handoff note, or ask you to close it. A close is only a request; you confirm it in the app. |
| `files_create` | Creates one new text document in a binder. It never overwrites an existing file. You may be asked in the app first. |
| `files_edit` | Replaces the text of one existing text document. You are asked in the app, and nothing is written unless you allow it within two minutes. The app keeps the original before writing. |
| `files_trash` | Moves documents to the Windows Recycle Bin. If a file cannot go to the Recycle Bin, you are always asked in the app first. |
| `binders_add`, `binders_remove` | Add a folder as a binder, or remove a binder from the list. You are asked in the app every time. Removing a binder deletes no files. |

### Account

| Tool | What it does |
|---|---|
| `activate` | Starts the next account step when RichBinder blocks AI reading: signing in, starting the trial, opening the purchase page, refreshing the license, or opening the AI apps screen. Starting the trial shows the terms first and needs your confirmation. Sign-up, agreeing to terms and payment are always done by you. |

`activate` does not read or change documents. For its steps the RichBinder app communicates with the RozinSol account server and opens the sign-in or purchase page in your browser. Account information in these steps is handled under the RozinSol privacy policy: https://rozinsol.com/en/legal/privacy/

## Connect your AI agent

The server is a local program that speaks MCP over standard input and output (stdio). Every client below needs the same two values: the command `rbctl.exe` and the argument `mcp`.

### Claude Code

```
rbctl mcp install
```

This registers RichBinder in Claude Code for your user, so it is available in any folder. Restart Claude Code afterwards. `rbctl mcp install --print` only shows the command and the configuration without running anything, and `rbctl mcp status` checks the registration.

To register it yourself instead:

```
claude mcp add --scope user richbinder -- rbctl mcp
```

### Claude Desktop

Add this to `claude_desktop_config.json` (Settings → Developer → Edit Config), then restart Claude Desktop:

```json
{
  "mcpServers": {
    "richbinder": {
      "command": "rbctl.exe",
      "args": ["mcp"]
    }
  }
}
```

### Cursor

Add the same block to `.cursor/mcp.json` in your project, or to `mcp.json` in your user Cursor folder:

```json
{
  "mcpServers": {
    "richbinder": {
      "command": "rbctl.exe",
      "args": ["mcp"]
    }
  }
}
```

### Gemini CLI

Add the same block to `settings.json` in your user `.gemini` folder:

```json
{
  "mcpServers": {
    "richbinder": {
      "command": "rbctl.exe",
      "args": ["mcp"]
    }
  }
}
```

### Any other MCP client

Register a stdio server with the command `rbctl.exe` and the single argument `mcp`. No environment variables and no keys are needed in the configuration.

## Example prompts

- Find my meeting notes about the quarterly budget review in RichBinder and summarize the decisions that were made.
- Search my RichBinder documents for the warranty period in the office printer service contract, and quote the exact sentence.
- Using RichBinder, compare what my two apartment lease drafts say about the security deposit and list the differences.
- Is RichBinder ready on this PC? Call `state` and tell me what it says.

## Troubleshooting

| Symptom | Check |
|---|---|
| The agent says the server failed to start | Is RichBinder installed from the Microsoft Store? Open a terminal and run `rbctl mcp status`. If `rbctl` is not found, turn it on in Windows Settings › Apps › Advanced app settings › App execution aliases, or repair RichBinder. |
| The tools say the connection is not approved | Approve it in RichBinder Settings › AI apps, then ask again. |
| The tools answer with an "app off" error | Start the RichBinder app and keep it running, then ask again. |
| The tools answer with an edition error | AI search needs the AI edition or its 30-day trial. Check the edition in the app, or ask the agent to use `activate`. |
| The server answers but finds nothing | Has the app finished building its index, and is the binder allowed for this connection? Ask the agent to call `state` and read what it says. |
| A change tool is refused | Changes are off by default. Turn on the permission for this connection in RichBinder Settings › AI apps. |

## Privacy

Data handling follows the RichBinder privacy policy: https://richbinder.com/en/legal/privacy/

The account steps (sign-in, trial, license) are handled by the RozinSol account service, which has its own privacy policy: https://rozinsol.com/en/legal/privacy/

## Support

Contact: support@rozinsol.com

## License

The text and configuration examples in this repository are licensed under the MIT License (see [LICENSE](LICENSE)). The RichBinder app and `rbctl.exe` are not covered by this license; they are licensed under their own end-user license agreement.
