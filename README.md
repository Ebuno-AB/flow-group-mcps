# Flow Group MCPs

Flow Group's plugins for Claude Code. Each one connects your coding assistant to one of our services
through its MCP server and adds skills for the work around it.

```bash
claude plugin marketplace add Ebuno-AB/flow-group-mcps
claude plugin install <plugin>@flow-group
```

| Plugin | What it does |
| --- | --- |
| `flow-affiliate` | Integrates Flow Affiliate into an Expo or React Native app, its backend or a website, tests it against Flow's test environment and offers an App Clip for iPhone. |

## flow-affiliate

Skills:

- `integrate-flow-affiliate`: finds the app, backend or website in your project, installs and wires the
  SDK, adds the sign-up confirmation to your backend, sends a test sign-up, checks what arrived and ends
  with what is left in the Flow dashboard.
- `add-app-clip`: offers an iOS App Clip for Flow invites and walks through adding one to an Expo app,
  or checks the one you have.

1. In Flow Affiliate, open your program, step 1 "Install the SDK", choose "Let your AI assistant do it"
   and create an AI key. It works only for that program, only in the test environment.
2. Then:

```bash
export FLOW_AI_KEY=fg_ai_...
claude plugin marketplace add Ebuno-AB/flow-group-mcps
claude plugin install flow-affiliate@flow-group
```

3. In your project: "Integrate Flow Affiliate into this app", or `/flow-affiliate:integrate-flow-affiliate`.

`FLOW_MCP_URL` points the plugin at another Flow environment (default `https://api.affiliate.yflow.io/v1/mcp`).

Without the plugin, only the tools:

```bash
claude mcp add --transport http flow-affiliate https://api.affiliate.yflow.io/v1/mcp --header "Authorization: Bearer $FLOW_AI_KEY"
```

Cursor, VS Code and Claude Desktop: see https://affiliate.yflow.io/docs/guides/ai-assisted-setup.

## Adding a plugin

Put it in `plugins/<name>/` (`.claude-plugin/plugin.json`, `.mcp.json`, `skills/`), list it in
`.claude-plugin/marketplace.json` and in the table above, and check it with
`claude plugin validate --strict .`.

The `flow-affiliate` plugin is developed in the Flow Affiliate monorepo (`integrations/claude-plugin`)
and published here.
