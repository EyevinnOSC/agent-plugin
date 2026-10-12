# OSC Plugin for Claude Code

Deploy your own apps and manage live streaming and open intercom infrastructure on
[Open Source Cloud](https://www.osaas.io) directly from your Claude Code session.

The plugin connects Claude Code to OSC's operator MCP surface -- curated tools that cover service
discovery, provisioning, and platform architecture guidance:

- `osc_search_tools` -- search the OSC service catalog by keyword
- `osc_call_tool` -- read-only calls: describe instances, check status, read logs
- `osc_write_tool` -- calls that create, change or delete something
- `ask-osc-architect` -- natural-language OSC architecture and service selection guidance (see Disclosure below)

Three guided skills are included:

- **Deploy My Repo** -- put a git repository live as a private OSC My App and get its URL
- **Live Streaming** -- RTMP-to-HLS streaming stack using `eyevinn-live-encoding`
- **Open Intercom** -- self-hosted WebRTC intercom using `eyevinn-intercom-manager`, a WebRTC SFU, and CouchDB

## Install

In Claude Code:

```
/plugin marketplace add EyevinnOSC/agent-plugin
/plugin install osc@agent-plugin
```

Then run `/mcp` and approve the OSC sign-in when prompted.

## Use the skills

After installation, invoke a skill by name in chat:

- "Deploy this repo on OSC" -- triggers `deploy-my-repo`
- "Set up the live streaming stack" -- triggers `live-streaming`
- "Set up Open Intercom" -- triggers `open-intercom`

You can also call the operator tools directly. For example:

```
Search for streaming services on OSC
```

## Disclosure

`ask-osc-architect` sends your prompt to a third-party AI model provider. See
[www.osaas.io/subprocessors](https://www.osaas.io/subprocessors) for the full list of subprocessors and
retention terms.

## Troubleshooting

**OAuth prompt does not appear or complete.** Run `/mcp`, select `osc`, and choose to authenticate again. The
sign-in opens in your browser at app.osaas.io. If the browser does not open, copy the URL that Claude Code
prints and open it yourself.

**Tools missing after install.** Run `/reload-plugins`, or restart Claude Code, and check that `osc` shows as
connected in `/mcp`.

## License

MIT
