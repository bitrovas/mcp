# Bitrovas MCP-Server

[English version (README.md)](./README.md)

Model-Context-Protocol-Server (MCP) für die L402-Dienste von Bitrovas: Bitcoin- und Daten-Werkzeuge gegen Lightning-Zahlung, unter [`https://api.bitrovas.ch`](https://api.bitrovas.ch/docs).

Keine Konten, keine API-Schlüssel, keine Cookies, keine gespeicherten Kundendaten. Bezahlte Endpunkte nutzen das HTTP-Protokoll **L402** (BOLT11-Lightning-Rechnung plus Macaroon).

## So funktioniert die L402-Zahlung im MCP

1. Rufe ein kostenpflichtiges Tool (zum Beispiel `bitrovas_psbtcheck`) **ohne** `l402_macaroon` / `l402_preimage` auf.
2. Ungültige Eingaben werden kostenlos abgelehnt (`HTTP 400`), noch bevor eine Rechnung ausgestellt wird.
3. Bei gültigen Eingaben liefert das Tool `status: "payment_required"` mit:
   - `amount_sats`
   - `invoice` (BOLT11-Lightning-Rechnung)
   - `macaroon` (L402-Token)
   - `payment_hash` und `expires_at`
4. Bezahle die BOLT11-Rechnung mit einer beliebigen Lightning-Wallet (oder einem MCP-Wallet-Tool) und erhalte den 64-stelligen hexadezimalen `preimage`.
5. Rufe **dasselbe Tool erneut** mit **denselben Eingaben** auf und ergänze:
   - `l402_macaroon`: den Macaroon aus Schritt 3
   - `l402_preimage`: den 64-stelligen hexadezimalen Lightning-Preimage aus Schritt 4

Der Server sendet `Authorization: L402 <macaroon>:<preimage>` an `api.bitrovas.ch` und gibt das Ergebnis zurück.

## Tools

### Kostenlose Tools (ohne Rechnung)

| Tool | Endpunkt | Zweck |
| --- | --- | --- |
| `bitrovas_services_status` | `GET /ready`, `GET /.well-known/l402-services` | API-Bereitschaft prüfen und alle Dienste, Preise und Schemas auflisten |
| `bitrovas_psbtcheck_quote` | `POST /v1/psbtcheck/quote` | Kostenlose Preisprüfung (21 Sats) plus optionale PSBT-Gültigkeitsprüfung |
| `bitrovas_formatcheck_quote` | `POST /v1/formatcheck/quote` | Kostenlose Preisangabe nach Byte/KB plus optionale Syntax- und Schema-Prüfung |
| `bitrovas_markdown_quote` | `POST /v1/markdown/quote` | Kostenlose Prüfung von Dokumenttyp, Grösse und Preis (100 Sats) |
| `bitrovas_timestamp_status` | `GET /v1/timestamp/<id>` | OpenTimestamps-Status abfragen (`pending`/`confirmed`), `.ots`-Nachweis und Blockhöhe |
| `bitrovas_adresswaechter_info` | `GET /v1/adresswaechter/info` | Dienstinfo, Grenzen und L402-Ablauf für den Adress-Wächter |
| `bitrovas_adresswaechter_status` | `GET /v1/adresswaechter/<id>` oder `/status` | Abo-Status per ID prüfen oder eine Bestellung über den L402-Nachweis wiederherstellen |
| `bitrovas_poisoncheck_ai_status` | `GET /v1/poisoncheck/ai/<ticket>` | Ergänzende KI-Erklärung einmal abfragen, wenn `ai_explanation.poll` zurückgegeben wurde |

### Kostenpflichtige L402-Tools

| Tool | Endpunkt | Preis | Zweck |
| --- | --- | --- | --- |
| `bitrovas_psbtcheck` | `POST /v1/psbtcheck` | 21 Sats | Analyse einer unsignierten Bitcoin-PSBT vor dem Signieren (`ok\|warn\|danger`) |
| `bitrovas_poisoncheck` | `POST /v1/poisoncheck` | 21 Sats (Stapel: 21 + 5/Adresse, max. 50) | Prüfung auf ähnlich aussehende Adressen (Bitcoin, EVM, Tron) |
| `bitrovas_formatcheck` | `POST /v1/formatcheck` | 10 Sats + 1 Sat/KB (max. 1 MiB) | Prüfen und umwandeln von CSV, YAML, XML, TOML, JSON plus JSON Schema 2020-12 |
| `bitrovas_jsonrepair` | `POST /v1/jsonrepair` | 10 Sats | JSON-Arrays reparieren und bereinigen (max. 1000 Zeilen) plus JSON-Schema-Prüfung |
| `bitrovas_qrbill` | `POST /v1/qrbill` | 10 Sats | Technische Prüfung eines Swiss-QR-Bill-SPC-Payloads oder einer CH/LI-IBAN plus Referenz |
| `bitrovas_timestamp` | `POST /v1/timestamp` | 21 Sats | Bitcoin-OpenTimestamps-Nachweis für einen SHA-256-Hash oder eine Nostr-Event-ID |
| `bitrovas_adresswaechter_subscribe` | `POST /v1/adresswaechter` | 2'100 Sats / 90 Tage | Watch-only-Überwachung einer Bitcoin-Mainnet-Adresse oder xpub per NIP-17-Nostr-DM |
| `bitrovas_walletaddr` | `POST /v1/walletaddr` | 21 Sats | Watch-only-Adressableitung und Eigentumsprüfung aus xpub oder Descriptor |
| `bitrovas_markdown` | `POST /v1/markdown` | 100 Sats | PDF, DOCX, XLSX, PPTX, HTML oder CSV (max. 5 MB) nach Markdown umwandeln |

## Nutzung

Verbinde einen MCP-Client mit Unterstützung für Streamable HTTP mit:

```text
https://api.bitrovas.ch/mcp
```

Beispiel-Konfiguration (Cursor / Claude):

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

## MCP-Verzeichnis

Der Server ist im offiziellen [MCP-Verzeichnis](https://registry.modelcontextprotocol.io/v0/servers?search=bitrovas) unter dem Namen `io.github.bitrovas/mcp` eingetragen.
