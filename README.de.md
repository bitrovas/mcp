# Bitrovas MCP-Server

[English version (README.md)](./README.md)

Model-Context-Protocol-(MCP-)Server für die Bitrovas-Dienste auf [`https://api.bitrovas.ch`](https://api.bitrovas.ch/docs) mit Lightning-Bezahlung über **L402**.

Kein Konto, kein API-Schlüssel, keine Cookies, keine Speicherung von Kundendaten. Kostenpflichtige Endpunkte nutzen das offene HTTP-**L402**-Protokoll (BOLT11-Lightning-Rechnung + Macaroon).

## So funktioniert L402 im MCP

1. Ein kostenpflichtiges Tool (z. B. `bitrovas_psbtcheck`) **ohne** `l402_macaroon` / `l402_preimage` aufrufen.
2. Ungültige Eingaben werden kostenlos (`HTTP 400`) abgelehnt, bevor eine Rechnung entsteht.
3. Bei gültiger Eingabe liefert das Tool `status: "payment_required"` mit:
   - `amount_sats`
   - `invoice` (BOLT11-Lightning-Rechnung)
   - `macaroon` (L402-Token)
   - `payment_hash` und `expires_at`
4. Die BOLT11-Rechnung mit einer beliebigen Lightning-Wallet (oder einem Wallet-Tool) bezahlen und das 64-stellige Hex-`preimage` auslesen.
5. Dasselbe Tool **erneut mit exakt denselben Eingabefeldern** aufrufen und zusätzlich angeben:
   - `l402_macaroon`: das Macaroon aus Schritt 3
   - `l402_preimage`: das 64-stellige Hex-Preimage aus Schritt 4

Der MCP-Server sendet `Authorization: L402 <macaroon>:<preimage>` an `api.bitrovas.ch` und gibt das Ergebnis zurück.

## Verfügbare Tools

### Kostenlose Tools (ohne Rechnung)

| Tool | Endpunkt | Zweck |
| --- | --- | --- |
| `bitrovas_services_status` | `GET /ready`, `GET /.well-known/l402-services` | Status der API und Live-Liste aller Dienste, Preise und Schemas |
| `bitrovas_psbtcheck_quote` | `POST /v1/psbtcheck/quote` | Gratis-Preisauskunft (21 Sats) + optionale PSBT-Vorprüfung |
| `bitrovas_formatcheck_quote` | `POST /v1/formatcheck/quote` | Gratis-Preisberechnung nach Bytes/KB + optionale Syntax-/Schema-Vorprüfung |
| `bitrovas_markdown_quote` | `POST /v1/markdown/quote` | Gratis-Prüfung von Dateityp, Grösse und Preis (100 Sats) |
| `bitrovas_utxoplan_preview` | `POST /v1/utxoplan/preview` | Gratis-Vorschau des UTXO-Planers: Anzahl UTXOs, Bestand, Gebührenszenarien, Kosten des Zusammenlegens (xpub, Descriptor, Adressen oder UTXO-Liste, ohne Outpoints) |
| `bitrovas_timestamp_status` | `GET /v1/timestamp/<id>` | Status eines OpenTimestamps-Belegs (`pending`/`confirmed`) samt Blockhöhe abrufen |
| `bitrovas_adresswaechter_info` | `GET /v1/adresswaechter/info` | Preis, Limits und Ablauf des Adress-Wächters |
| `bitrovas_adresswaechter_status` | `GET /v1/adresswaechter/<id>` oder `/status` | Abo-Status per ID oder Wiederherstellung per L402-Nachweis |
| `bitrovas_poisoncheck_ai_status` | `GET /v1/poisoncheck/ai/<ticket>` | Einmaliges Abholen einer optionalen KI-Zusatzerklärung (falls `ai_explanation.poll` geliefert wurde) |

### Kostenpflichtige L402-Tools

| Tool | Endpunkt | Preis | Zweck |
| --- | --- | --- | --- |
| `bitrovas_psbtcheck` | `POST /v1/psbtcheck` | 21 Sats | Vorab-Prüfung einer unsignierten Bitcoin-PSBT (`ok\|warn\|danger`) |
| `bitrovas_poisoncheck` | `POST /v1/poisoncheck` | 21 Sats (Batch: 21 + 5/Adresse, max. 50) | Prüfung auf Address-Poisoning-Lookalikes (Bitcoin, EVM, Tron) |
| `bitrovas_formatcheck` | `POST /v1/formatcheck` | 10 Sats + 1 Sat/KB (max. 1 MiB) | Prüfung/Konvertierung von CSV, YAML, XML, TOML, JSON inkl. JSON-Schema 2020-12 |
| `bitrovas_jsonrepair` | `POST /v1/jsonrepair` | 10 Sats | Reparatur & Bereinigung von JSON-Datensätzen (max. 1000 Zeilen) |
| `bitrovas_qrbill` | `POST /v1/qrbill` | 10 Sats | Technische Prüfung einer Schweizer QR-Rechnung (SPC) oder CH/LI-IBAN + Referenz |
| `bitrovas_timestamp` | `POST /v1/timestamp` | 21 Sats | Bitcoin-Zeitstempel (OpenTimestamps) für SHA-256-Hash oder Nostr-Event-ID |
| `bitrovas_adresswaechter_subscribe` | `POST /v1/adresswaechter` | 2100 Sats / 90 Tage | Watch-only-Überwachung einer Bitcoin-Mainnet-Adresse oder xpub/ypub/zpub per NIP-17-DM |
| `bitrovas_walletaddr` | `POST /v1/walletaddr` | 21 Sats | Watch-only-Adressableitung & Zugehörigkeitsprüfung aus xpub/Descriptor |
| `bitrovas_markdown` | `POST /v1/markdown` | 100 Sats | Umwandlung von PDF, DOCX, XLSX, PPTX, HTML oder CSV (max. 5 MB) in Markdown |
| `bitrovas_utxoplan` | `POST /v1/utxoplan` | 1200 Sats | Watch-only UTXO-Plan: zusammenlegen, einfrieren, getrennt lassen, Outpoints für Coin Control, Break-even, Plan als Markdown |

## Einrichtung

### Variante 1: Remote Streamable HTTP (empfohlen)

MCP-Clients mit Streamable-HTTP-Unterstützung verbinden sich direkt mit:

```text
https://api.bitrovas.ch/mcp
```

Beispielkonfiguration (Cursor / Claude):

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

### Variante 2: Lokal (`stdio`) aus diesem Ordner

Voraussetzung: Node.js >= 22. Es gibt kein veröffentlichtes npm-Paket; der Server wird aus diesem Ordner gestartet:

```bash
npm ci
npm run build
node dist/src/cli.js --stdio
```

### Variante 3: Eigener Streamable-HTTP-Server (Self-Hosting)

```bash
npm ci
npm run build
node dist/src/cli.js --http --host 127.0.0.1 --port 8101 --path /mcp
```

Umgebungsvariablen:

- `BITROVAS_API_URL` (Standard: `https://api.bitrovas.ch`)
- `BITROVAS_API_TIMEOUT_MS` (Standard: `45000`)
- `MCP_TRANSPORT` (`stdio` oder `http`, Standard: `stdio`)
- `MCP_HOST` (Standard: `127.0.0.1`)
- `MCP_PORT` (Standard: `8101`)
- `MCP_PATH` (Standard: `/mcp`)
- `MCP_CORS_ORIGINS` (optional, kommagetrennte Origin-Allowlist)

Vorlagen für systemd und Caddy liegen unter [`deploy/bitrovas-mcp.service`](./deploy/bitrovas-mcp.service) und [`deploy/Caddyfile.snippet`](./deploy/Caddyfile.snippet).

## Entwicklung & Tests

```bash
npm ci
npm run typecheck
npm test
```

## MCP-Registry

`server.json` entspricht dem offiziellen MCP-Registry-Schema (`2025-12-11`) und beschreibt nur den Remote-Endpunkt (`streamable-http` unter `https://api.bitrovas.ch/mcp`). Es gibt kein veröffentlichtes npm-Paket.
