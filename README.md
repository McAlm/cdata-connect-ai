# CData Connect AI — Camunda Element Templates

This repository contains [Camunda 8](https://camunda.com/) element templates for connecting BPMN processes to [CData Connect AI](https://www.cdata.com/connect/ai/) over the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/).

## What's in this repo

| File | Description |
|---|---|
| [`element-templates/CData - Connect AI.json`](element-templates/CData%20-%20Connect%20AI.json) | Configures a Camunda task to talk to CData's hosted MCP server (`https://mcp.cloud.cdata.com/mcp`), either as a standalone task or as a tool inside an AI Agent. |

This template doesn't ship its own connector runtime code — it's a properties-panel configuration for Camunda's built-in [Agentic AI MCP Remote Client Connector](https://docs.camunda.io/docs/components/connectors/out-of-the-box-connectors/agentic-ai-mcp-remote-client-connector/), pre-filled with CData's endpoints. It requires **Camunda 8.9 or later**.

## Prerequisites

- A Camunda 8.9+ cluster (SaaS or Self-Managed) with the Agentic AI connectors enabled.
- A [CData Connect AI](https://www.cdata.com/connect/ai/) account with access to the MCP server.
- A CData **ServiceAccount** (client ID + client secret) with the permissions your process needs, if you're using OAuth 2.0 authentication (the default).

## 1. Install the element template

Pick the option that matches your setup:

### Desktop Modeler

- **Per-project:** copy the JSON file into a `.camunda/element-templates/` folder next to your `.bpmn` file(s).
- **Globally (all diagrams):** copy it into the modeler's user data directory:
  - macOS: `~/Library/Application Support/camunda-modeler/resources/element-templates/`
  - Windows: `%APPDATA%\camunda-modeler\resources\element-templates\`
  - Linux: `~/.config/camunda-modeler/resources/element-templates/`

  Then reload templates with `Cmd+R` / `Ctrl+R`.

### Web Modeler

1. Open your project.
2. Use **Upload files** to upload the JSON file as a new element template (or, if updating an existing one, open it in the template editor and use **Replace via upload**).

### Self-Managed cluster runtime

The connector runtime itself (`io.camunda.agenticai:mcpremoteclient:1`) ships with Camunda 8.9+ Self-Managed by default — you only need to install the element template above so it shows up in the Modeler UI. See [Manage connector templates](https://docs.camunda.io/docs/components/connectors/out-of-the-box-connectors/agentic-ai-mcp-remote-client-connector/) for details on keeping the template in sync with your Camunda version.

## 2. Apply the template to a task

1. Add a **Task** (service task) to your BPMN diagram — either standalone or inside an AI Agent's ad-hoc sub-process.
2. In the properties panel, apply the **CData Connect AI** template.
3. Configure the sections below.

## 3. Configure transport

Under **Transport**, choose:

- **Streamable HTTP** (default, recommended) — URL defaults to `https://mcp.cloud.cdata.com/mcp`.
- **SSE** (legacy) — for servers that only support Server-Sent Events; URL typically ends in `/sse`.

Optionally set custom **HTTP headers** (as a FEEL map) and a request **timeout** (ISO-8601 duration, e.g. `PT30S`).

## 4. Configure authentication

Under **Authentication**, choose one:

| Type | What you need |
|---|---|
| None | Nothing — only use this if your MCP endpoint requires no auth. |
| Basic | Username and password. |
| Bearer token | A pre-issued bearer token. |
| **OAuth 2.0 (default)** | Client ID and client secret from a CData **ServiceAccount**, plus optional audience/scopes. |

The template pre-fills the OAuth 2.0 token endpoint (`https://cloud-login.cdata.com/oauth/token`) and defaults to sending credentials as a Basic Auth header (`BASIC_AUTH_HEADER`) — switch to `CREDENTIALS_BODY` if your setup requires client credentials in the request body instead.

**Create the ServiceAccount in CData first** and grant it the permissions your process needs before filling in the Client ID / Client secret fields. Treat these as secrets — reference them via Camunda secrets (e.g. `secrets.CDATA_CLIENT_SECRET`) rather than pasting raw values.

## 5. Choose a connector mode

Under **Connector mode**, pick how the task is used:

### Standalone

Use this when the task itself should perform one specific MCP call. Under **Operation**, pick:

- `tools/list` — list the tools CData exposes.
- `tools/call` — call a named tool (set **Tool name** and **Tool arguments** as a FEEL map).
- `resources/list`, `resources/templates/list`, `resources/read` — browse and read resources (set **Resource URI** for reads).
- `prompts/list`, `prompts/get` — browse and fetch prompts (set **Prompt name** and **Prompt arguments** for gets).

Use the **Filters** section (included/excluded tools, resources, or prompts) to restrict what a `list`/`call` operation can see or invoke.

### AI Agent tool

Use this when the task sits inside an [AI Agent ad-hoc sub-process](https://docs.camunda.io/docs/components/connectors/out-of-the-box-connectors/agentic-ai-mcp-remote-client-connector/) and should let the agent discover and call CData's tools on its own. In this mode the method and parameters come from the agent at runtime (`=toolCall.method` / `=toolCall.params`) — you don't set an operation manually. Use **Filters** to limit which tools the agent is allowed to see and call.

## 6. Output and error handling

- **Result variable** (default `toolCallResult`) — process variable the raw MCP response is stored in.
- **Result expression** — FEEL expression to map the response into your own process variables.
- **Error expression** — FEEL expression to handle failures.
- **Retries** (default `3`) and **retry backoff** (default `PT30S`) control retry behavior on failure; **job timeout** is optional.

See [Connectors: result variable and expression](https://docs.camunda.io/docs/components/connectors/use-connectors/#result-variable) for the general mapping syntax.

## Limitations

Each task activation opens a new HTTP connection/SSE subscription to the MCP server — there's no persistent connection reuse across process instances (you can opt into caching a single client instance across invocations within the same connector runtime via **Enable client caching** under **Options**, but only if your configuration and credentials won't change between calls). Per Camunda's documentation, this connector is best suited to prototyping and testing rather than high-throughput production workloads.

## Useful links

- [Agentic AI MCP Remote Client Connector docs](https://docs.camunda.io/docs/components/connectors/out-of-the-box-connectors/agentic-ai-mcp-remote-client-connector/)
- [Model Context Protocol specification](https://modelcontextprotocol.io/)
- [CData Connect AI](https://www.cdata.com/connect/ai/)
