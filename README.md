# Customer.io audience segment workflows for AI assistants

Inspect existing segments and explicitly create an empty manual segment for a planned audience.

[Start locally](#get-started) · [Connect your assistant](#connect-your-assistant) · [Available tools](#available-tools) · [Hosted PipMCP setup](#use-pipmcp-for-production)

## Built for useful workflows

- **Build a Customer.io brief.** Read segment definitions from the US-region App API. Make that provider response available to your assistant before it proposes a next action.
- **Use Customer.io from your existing tools.** Run `customerio_list_segments` from the example CLI or expose it to a stdio MCP client using the same account configuration.
- **Add a reviewed Customer.io action.** Create an empty manual segment; no people are added and no message is sent by this request. Review the input first; execution requires operator opt-in and per-call approval.

## Why use this package?

Get a focused Customer.io MCP integration without assembling request mappings and a stdio transport yourself. The package provides documented input schemas, provider authentication handling and an example CLI, with the same tools available to your assistant.

Reads are available independently of writes. Approved changes use operator opt-in and a per-call idempotency key; uncertain writes are not silently replayed.

## Get started

**Requirements:** Node.js 22 or newer, npm, and access to the provider account you intend to use.

```sh
git clone https://github.com/jannikco/customer-io-mcp.git
cd customer-io-mcp
npm ci --ignore-scripts
npm test
npm start
```

`npm start` shows the tool inventory and workflow offline. It does not authenticate or call the provider. The tests use provider fixtures and stdio transport; they do not modify your account.

### Configure your provider account

The current release connects directly to the provider with **your own credentials**. It does not require a PipMCP key for local execution. Configure the environment of the process or MCP client that runs the server; `.env` files are not loaded automatically.

| Environment variable | What to supply |
| --- | --- |
| `CUSTOMER_IO_APP_API_KEY` | Your own provider API key or access token. |

Use your own scoped App API Key (sk_), not Track API Site ID/key. This is the US App API; EU workspaces require a separately reviewed api-eu.customer.io manifest. An empty segment can still interact with existing workspace automation; review it.

[Provider credential/reference documentation](https://docs.customer.io/integrations/api/app/tag/segments/listsegments/). Keep credentials out of commits, issue reports and shared client configuration.

### Run your first provider read

After configuring the environment, run `customerio_list_segments`:

```sh
npm start -- --call customerio_list_segments '{}'
```

Replace sample account, record, file and project IDs with values from your authorized account. This command makes a real provider request and returns its result for you to inspect.

## Available tools

| Capability | MCP tool | Effect | Required inputs |
| --- | --- | --- | --- |
| Read segment definitions from the US-region App API. | `customerio_list_segments` | Read | None |
| Create an empty manual segment; no people are added and no message is sent by this request. | `customerio_create_manual_segment` | Approved write | `name`, `description` |

Tool schemas and bindings are in [connector.json](./connector.json). These are the selected operations this package ships.

### A practical tool sequence

1. Use `customerio_list_segments` — Read segment definitions from the US-region App API.
2. Use `customerio_create_manual_segment` — Create an empty manual segment; no people are added and no message is sent by this request.

## Connect your assistant

Add this stdio server to an MCP-compatible client. Replace the absolute path and credential placeholders. Use your client’s secret/environment controls instead of committing this configuration.

```json
{
  "mcpServers": {
    "customer-io-mcp": {
      "command": "node",
      "args": [
        "/absolute/path/customer-io-mcp/server.mjs"
      ],
      "env": {
        "CUSTOMER_IO_APP_API_KEY": "YOUR_CUSTOMER_IO_APP_API_KEY"
      }
    }
  }
}
```

`node server.mjs` starts the MCP server; `npm start` runs the example CLI. The server reserves stdout for MCP messages. If the client does not inherit your shell environment, pass the required variables explicitly.

## Review changes before running writes

The write tool is disabled by default. Enable `CONNECTOR_ALLOW_WRITES=true` in the server process and include this approval object in the **specific** write call after reviewing its arguments:

```json
{
  "_approval": {
    "approved": true,
    "idempotencyKey": "unique-reviewed-request-id"
  }
}
```

This object is added alongside the tool’s own required fields; it is not a complete write request. Use a new idempotency key for a different action. If an outcome is uncertain, inspect the provider before trying again. Creating a record may trigger existing provider automations even when this package does not send a message itself.

## Use PipMCP for production

The local package is useful when you want to run these selected Customer.io operations in your own process. PipMCP provides a hosted place to configure provider connections, publish tools, issue endpoint access tokens, apply server-side policy and approvals, and inspect execution/usage records. Provider credentials and hosted execution are configured in that service.

**[Set up Customer.io in PipMCP](https://pipmcp.com/github/start?app=customer-io&integration=customer-io-mcp&utm_source=github&utm_campaign=github-customer-io-mcp&utm_content=readme)**

A managed catalog template is listed, but that is not evidence that its tools match this repository or that your account is connected. Complete provider setup and inspect the endpoint’s actual tool inventory and permissions before use.

Local credentials are not uploaded by this repository. A hosted endpoint uses its own PipMCP endpoint access token and published tool inventory; the local and hosted credentials are separate.

## Scope and validation

- The installed package has HTTP request fixtures and MCP stdio `tools/call` tests.
- Fixture success checks client behavior; it does not establish provider authentication, paid-plan access or live account execution.
- No live provider execution is claimed for this package by these tests.
- Hosted tool names, schemas, permissions and availability can differ from this local package. A queued or accepted job/message is not proof of completion or delivery. Provider API plans, account rules and limits apply.
- See [integration.json](./integration.json) for operation-scoped evidence and the provider-specific limits above.

## FAQ

**Do I need a Customer.io account?**

You need the provider account, token and permissions required by these operations. Provider charges and API limits remain with that account.

**Does this version require a PipMCP API key?**

This package uses your provider credentials for local execution. To use the hosted service, configure the provider connection in PipMCP and use the endpoint access token issued there.

**Why is a tool missing or a request rejected?**

This repository exposes the selected tools above. Check the required environment variables, provider scopes, account/region requirements and input IDs first. Hosted endpoints have their own published inventory and may exclude tools by role or policy. Do not retry a failed write without checking its outcome.

**Is this an official provider SDK or a hosted-service guarantee?**

This is an independently maintained MCP integration. Its fixtures and source references do not imply provider endorsement, unrestricted API access, or identical hosted behavior.

## Guides and related workflows

- [Customer.io integration guide](https://jannikco.github.io/pipmcp-connectors/integrations/customer-io/)
- [Customer.io workflow and tool sequence](https://jannikco.github.io/pipmcp-connectors/workflows/customer-io/)
- [More integration workflows](https://jannikco.github.io/pipmcp-connectors/workflows/)
- [Typed LeadAdapter API client](https://github.com/jannikco/linkedin-typescript-sdk)
- [LeadAdapter Python API client](https://github.com/jannikco/linkedin-python-sdk)
- [LinkedIn API quickstart](https://github.com/jannikco/linkedin-api)
- [LinkedIn people and post search](https://github.com/jannikco/linkedin-search-api)
- [LinkedIn post engagers](https://github.com/jannikco/linkedin-post-engagers)
- [LeadAdapter LinkedIn MCP server](https://github.com/jannikco/linkedin-mcp)
- [OpenAPI to MCP import starter](https://github.com/jannikco/openapi-to-mcp)
- [Swagger to MCP import starter](https://github.com/jannikco/swagger-to-mcp)
- [Customer.io API and credential reference](https://docs.customer.io/integrations/api/app/tag/segments/listsegments/)
- [Additional official source 2](https://docs.customer.io/integrations/api/app/tag/segments/createmansegment/)

Maintained by [jannikco](https://github.com/jannikco). MIT for this repository’s code; dependencies retain their licenses. [Report a reproducible issue](https://github.com/jannikco/customer-io-mcp/issues) with redacted input and error details. For credentials or security issues, use [private contact](https://pipmcp.com/contact).
