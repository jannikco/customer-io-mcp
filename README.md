# Customer.io MCP server

Inspect existing segments and explicitly create an empty manual segment for a planned audience.

A direct-provider MCP server with useful local execution. No PipMCP account is required for local use. Provider API access and any provider charges are your responsibility.

## Quick start

Requires Node.js22+. Clone this repository, then run:

```sh
npm ci --ignore-scripts
npm test
npm start
```

The default command inspects the tools offline. Configure your own credentials in your environment:

- `CUSTOMER_IO_APP_API_KEY`



Credential help: https://docs.customer.io/integrations/api/app/tag/segments/listsegments/

## Provider setup

Use your own scoped App API Key (sk_), not Track API Site ID/key. This is the US App API; EU workspaces require a separately reviewed api-eu.customer.io manifest. An empty segment can still interact with existing workspace automation; review it.

## Execute a real read

Replace example record IDs with records you are authorized to access.

```sh
npm start -- --call customerio_list_segments '{}'
```

## Use with an MCP client

Launch `node /absolute/path/customer-io-mcp/server.mjs` using stdio transport; pass credentials through the client environment. Stdout is reserved for JSON-RPC.

## Tools

| Tool | Effect | Purpose |
| --- | --- | --- |
| customerio_list_segments | Read | Read segment definitions from the US-region App API. |
| customerio_create_manual_segment | Explicit approval required | Create an empty manual segment; no people are added and no message is sent by this request. |

## Concrete workflow

Inspect existing segments and explicitly create an empty manual segment for a planned audience.

1. customerio_list_segments
2. customerio_create_manual_segment

## Optional writes

Writes are disabled by default. Set CONNECTOR_ALLOW_WRITES=true and supply `_approval: {approved:true,idempotencyKey:"unique-request-id"}` on the specific call. Do not reuse a key for a different request. Uncertain outcomes are never automatically replayed; inspect the provider before another attempt.

## Capability evidence

Local: **Tested — HTTP fixtures and real stdio tools/call**. Managed: **Documented**. Provider authentication and live operations have not been established by these fixture tests. Evidence applies per operation and execution mode; setup discovery does not imply provider execution.

## Managed production

[Activate with PipMCP](https://pipmcp.com/github/start?app=customer-io&integration=customer-io-mcp&utm_source=github&utm_campaign=github-customer-io-mcp&utm_content=readme) when you want the hosted service.  Local credentials are not uploaded by this code. Review provider permissions in the managed onboarding. [Directory and related projects](https://jannikco.github.io/pipmcp-connectors/integrations/customer-io/).

## Provenance and maintenance

- https://docs.customer.io/integrations/api/app/tag/segments/listsegments/
- https://docs.customer.io/integrations/api/app/tag/segments/createmansegment/

Maintainer:jannikco. MIT license for this code. See connector.json for selected operations and integration.json for evidence. Runtime dependencies retain their own licenses. Report sensitive issues through https://pipmcp.com/contact; do not post credentials or customer data in public issues.
