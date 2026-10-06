# OSC Plugin for Claude Code

Deploy and manage live streaming and open intercom infrastructure on
[Open Source Cloud](https://www.osaas.io) directly from your Claude Code session.

The plugin connects Claude Code to OSC's operator MCP surface -- three curated tools that cover service
discovery, provisioning, and platform architecture guidance:

- `osc_search_tools` -- search the OSC service catalog by keyword
- `osc_call_tool` -- create, describe, and delete service instances
- `ask-osc-architect` -- natural-language OSC architecture and service selection guidance (see Disclosure below)

Two guided skills are included:

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

**OAuth prompt does not appear or complete.** If your client cannot complete the OAuth flow, add a local
override entry to your own `.mcp.json` (not the plugin's committed file):

```json
{
  "mcpServers": {
    "osc": {
      "type": "http",
      "url": "https://mcp.osaas.io/mcp",
      "headers": {
        "Authorization": "Bearer ${OSC_PAT}"
      }
    }
  }
}
```

Set `OSC_PAT` in your environment to a Personal Access Token from
[app.osaas.io](https://app.osaas.io) (Settings > Access Tokens). Do not commit tokens to any file.

## License

MIT
