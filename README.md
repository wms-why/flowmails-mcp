# Flowmails MCP

The hosted [Model Context Protocol](https://modelcontextprotocol.io) server for
[Flowmails](https://flowmails.net) — a custom-domain email platform built for AI agents.

Point any MCP client at **`https://sdk.flowmails.net/mcp`** with a Flowmails bearer token,
and the agent gets a mailbox on your own domain: send mail, read the inbox, search history,
and **wait** for an inbound message instead of polling.

- **Transport:** MCP Streamable HTTP, JSON-RPC 2.0, protocol revision `2025-06-18`
- **Auth:** `Authorization: Bearer <flowmails-api-key>` (an SDK key `fm_…` or an agent credential `agc_…`)
- **Tools:** 12 (ten of them can change mailbox state; every call is scope-checked first)
- **Run it:** hosted — nothing to install, no server to operate
- **Pricing:** the platform's — Base is free, Pro $9.9/mo, Enterprise $39.9/mo. Your email
  provider bills you directly; Flowmails charges no per-email markup.

---

## Why it exists

Most "email MCP servers" wrap an inbox API. This one is built around the hard part of agent
workflows: **waiting**.

`wait_for_email` opens a durable wait on an address and a subject. The platform owns that
state, so the wait survives the edge, and the agent is handed the message when it lands — it
does not spin on an inbox polling loop. That is the difference between an agent that can run a
sign-up flow unattended and one that has to be babysat.

Sending is the same contract as the REST API and the official SDK: one key, bound to one
domain, with an idempotency key and an audit trail behind every send.

## Install

### Claude Code / Claude Desktop

```bash
claude mcp add --transport http flowmails https://sdk.flowmails.net/mcp \
  --header "Authorization: Bearer fm_..."
```

### Cursor / VS Code / Codex / any MCP client

```json
{
  "mcpServers": {
    "flowmails": {
      "url": "https://sdk.flowmails.net/mcp",
      "headers": {
        "Authorization": "Bearer fm_..."
      }
    }
  }
}
```

### Any client, raw JSON-RPC

```bash
curl -X POST https://sdk.flowmails.net/mcp \
  -H "Authorization: Bearer fm_..." \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## The tools

| Tool | Scope | What it does |
|---|---|---|
| `list_domains` | `addresses:read` | List the domains the credential can touch |
| `list_addresses` | `addresses:read` | List the inbound addresses of a domain |
| `create_address` | `addresses:write` | Provision a new inbound address |
| `search_emails` | `messages:search` | Search message history |
| `get_email` | `messages:read` | Read one message (body, attachments, headers) |
| `send_email` | `messages:send` | Send mail from a domain-bound address |
| `get_send_status` | `send_status:read` | Delivery status of a sent message |
| `wait_for_email` | `waits:write` | Open a durable wait on an address + subject |
| `get_wait_status` | `waits:read` | Check a wait (pending / resolved / expired) |
| `list_events` | `events:read` | Read the event log (cursor-based) |
| `claim_message` | `messages:claim` | Claim a message so a second agent won't take it |
| `complete_message` | `messages:claim` | Mark a claimed message as handled |

The scope check runs **before** the tool does, so a read-only credential cannot send, and a
credential bound to one domain cannot touch another.

## A typical agent flow

```
list_domains → create_address → wait_for_email → get_email → send_email → get_send_status
```

No dashboard visit in between. The MCP server is the protocol-level door onto the same
contract the REST API, the SDK and the CLI use.

## Links

- Landing page: <https://flowmails.net/mcp>
- Documentation: <https://flowmails.net/docs>
- For AI coding agents: <https://flowmails.net/for/ai-coding-agents>
- OpenAPI 3.1: <https://flowmails.net/openapi.json>
- SDK: [`@flowmails/flowmails-sdk`](https://www.npmjs.com/package/@flowmails/flowmails-sdk) ·
  CLI: [`@flowmails/cli`](https://www.npmjs.com/package/@flowmails/cli)

## Registry metadata

This repository is the source of truth for the server's entry in the official MCP Registry:
see [`server.json`](./server.json). The name is `io.github.wms-why/flowmails-mcp`.

## License

MIT — see [LICENSE](./LICENSE).

## Listed on

- [IndieHunt](https://indiehunt.io/project/flowmails) — free launch queued
