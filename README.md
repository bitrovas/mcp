# Bitrovas MCP Server

[Deutsche Version (README.de.md)](./README.de.md)

Model Context Protocol (MCP) server for the Bitrovas L402 Lightning-paid Bitcoin and data utility services on [`https://api.bitrovas.ch`](https://api.bitrovas.ch/docs).

No accounts, no API keys, no cookies, no customer data stored. Paid endpoints use the HTTP **L402** protocol (BOLT11 Lightning invoice + macaroon).

## How L402 Payment Works in MCP

1. Call any paid tool (for example `bitrovas_psbtcheck`) **without** `l402_macaroon` / `l402_preimage`.
2. Invalid inputs are rejected free of charge (`HTTP 400`) before an invoice is issued.
3. For valid inputs, the tool returns `status: "payment_required"` containing:
   - `amount_sats`
   - `invoice` (BOLT11 Lightning invoice)
   - `macaroon` (L402 token)
   - `payment_hash` and `expires_at`
4. Pay the BOLT11 invoice with any Lightning wallet (or an MCP wallet tool) to receive the 64-character hex `preimage`.
5. Call the **exact same tool again** with the **exact same input arguments** plus:
   - `l402_macaroon`: the macaroon from step 3
   - `l402_preimage`: the 64-hex Lightning preimage from step 4

The server sends `Authorization: L402 <macaroon>:<preimage>` to `api.bitrovas.ch` and returns the service result.

## Tools

### Free Tools (No Invoice)

| Tool | Endpoint | Purpose |
| --- | --- | --- |
| `bitrovas_services_status` | `GET /ready`, `GET /.well-known/l402-services` | Check API readiness and list all services, prices, and schemas |
| `bitrovas_psbtcheck_quote` | `POST /v1/psbtcheck/quote` | Free 21-sat price check + optional PSBT validity pre-check |
| `bitrovas_formatcheck_quote` | `POST /v1/formatcheck/quote` | Free byte/KB price quote + optional syntax/schema pre-check |
| `bitrovas_markdown_quote` | `POST /v1/markdown/quote` | Free document type/size/price check (100 sats) |
| `bitrovas_timestamp_status` | `GET /v1/timestamp/<id>` | Poll OpenTimestamps status (`pending`/`confirmed`), `.ots` proof, and block height |
| `bitrovas_adresswaechter_info` | `GET /v1/adresswaechter/info` | Service info, limits, and L402 flow for Adress-Waechter |
| `bitrovas_adresswaechter_status` | `GET /v1/adresswaechter/<id>` or `/status` | Check subscription status by ID, or recover an order via L402 credential |
| `bitrovas_poisoncheck_ai_status` | `GET /v1/poisoncheck/ai/<ticket>` | Poll supplementary AI explanation once if `ai_explanation.poll` was returned |

### Paid L402 Tools

| Tool | Endpoint | Price | Purpose |
| --- | --- | --- | --- |
| `bitrovas_psbtcheck` | `POST /v1/psbtcheck` | 21 sats | Pre-sign analysis of an unsigned Bitcoin PSBT (`ok\|warn\|danger`) |
| `bitrovas_poisoncheck` | `POST /v1/poisoncheck` | 21 sats (batch: 21 + 5/addr, max 50) | Address-poisoning lookalike check (Bitcoin, EVM, Tron) |
| `bitrovas_formatcheck` | `POST /v1/formatcheck` | 10 sats + 1 sat/KB (max 1 MiB) | Validate/convert CSV, YAML, XML, TOML, JSON + JSON Schema 2020-12 |
| `bitrovas_jsonrepair` | `POST /v1/jsonrepair` | 10 sats | Repair & clean JSON arrays (max 1000 rows) + JSON Schema check |
| `bitrovas_qrbill` | `POST /v1/qrbill` | 10 sats | Technical check of a Swiss QR-bill SPC payload or CH/LI IBAN + reference |
| `bitrovas_timestamp` | `POST /v1/timestamp` | 21 sats | Bitcoin OpenTimestamps proof for a SHA-256 hash or Nostr event ID |
| `bitrovas_adresswaechter_subscribe` | `POST /v1/adresswaechter` | 2100 sats / 90 days | Watch-only Bitcoin mainnet address/xpub monitoring via NIP-17 Nostr DM |
| `bitrovas_walletaddr` | `POST /v1/walletaddr` | 21 sats | Watch-only address derivation & ownership check from xpub/descriptor |
| `bitrovas_markdown` | `POST /v1/markdown` | 100 sats | Convert PDF, DOCX, XLSX, PPTX, HTML, or CSV (max 5 MB) to Markdown |

## Usage

Connect any MCP client that supports Streamable HTTP to:

```text
https://api.bitrovas.ch/mcp
```

Example client configuration (Cursor / Claude):

```json
{
  "mcpServers": {
    "bitrovas": {
      "type": "streamable-http",
      "url": "https://api.bitrovas.ch/mcp"
    }
  }
}
```

## MCP Registry

The server is listed in the official [MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=bitrovas) as `io.github.bitrovas/mcp`.
