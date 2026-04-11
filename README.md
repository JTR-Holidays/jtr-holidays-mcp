# JTR Holidays MCP Server

> Connect your AI assistant to JTR Holidays' live catalogue of attraction tickets, tours, and multi-day holiday packages via the [Model Context Protocol](https://modelcontextprotocol.io).

[![MCP Protocol](https://img.shields.io/badge/MCP-2024--11--05-blue)](https://modelcontextprotocol.io)
[![JSON-RPC](https://img.shields.io/badge/JSON--RPC-2.0-green)](https://www.jsonrpc.org/specification)
[![Destinations](https://img.shields.io/badge/Destinations-24%20Countries-orange)](https://www.jtrholidays.com)

JTR Holidays is a Dubai-based global travel platform rated **5 stars on Trustpilot** (734+ reviews). This MCP server gives AI assistants real-time access to search activities, check availability, and browse holiday packages across 24 countries — all via a single JSON-RPC 2.0 endpoint.

---

## Table of Contents

- [Quick Start](#quick-start)
- [Authentication](#authentication)
- [Available Tools](#available-tools)
- [Examples](#examples)
- [Supported Destinations](#supported-destinations)
- [Contact & Support](#contact--support)

---

## Quick Start

### Endpoint

```
POST https://api.jtrholidays.com/mcp/v1
Content-Type: application/json
x-jtr-aihub-secret: <your-secret-key>
```

### Auto-discovery (no auth required)

```
GET https://api.jtrholidays.com/.well-known/mcp.json
```

### MCP Handshake

```json
{
  "jsonrpc": "2.0",
  "method": "initialize",
  "id": 1,
  "params": {
    "protocolVersion": "2024-11-05",
    "clientInfo": { "name": "your-client", "version": "1.0" }
  }
}
```

### List available tools

```json
{
  "jsonrpc": "2.0",
  "method": "tools/list",
  "id": 2
}
```

### Call a tool

All tool calls use the `tools/call` method:

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": 3,
  "params": {
    "name": "<tool_name>",
    "arguments": { }
  }
}
```

---

## Authentication

Every request to `POST /mcp/v1` must include the secret header:

| Header | Value |
|--------|-------|
| `x-jtr-aihub-secret` | Your secret key |
| `Content-Type` | `application/json` |

To request API access, contact us at [it@jtrholidays.com](mailto:it@jtrholidays.com) or visit [jtrholidays.com/contact-us](https://www.jtrholidays.com/contact-us).

> The discovery endpoint (`GET /.well-known/mcp.json`) is public and requires no authentication.

---

## Available Tools

### `search_activities`

Search JTR Holidays' activity and tour catalogue by destination, category, and budget.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `destination` | string | No | City or country name (e.g. `"Dubai"`, `"Singapore"`) |
| `category` | string | No | Activity category (e.g. `"Desert Safari"`, `"Sightseeing"`) |
| `budget_max` | number | No | Maximum price per person in the activity's local currency |
| `limit` | integer | No | Max results to return (default: `20`, max: `50`) |

---

### `get_activity_details`

Get full details for a single activity including inclusions, exclusions, cancellation policy, and reviews.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `activity_id` | string | **Yes** | Activity UUID from `search_activities` results |

---

### `check_availability`

Check if an activity is available on a specific date and return live pricing options.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `activity_id` | string | **Yes** | Activity UUID |
| `date` | string | **Yes** | Date in `Y-m-d` format (e.g. `"2025-12-25"`) |
| `pax` | integer | No | Number of people (default: `1`) |

---

### `get_destinations`

List all destinations (countries) where JTR Holidays offers tours, with activity counts and popular picks. Takes no parameters.

---

### `get_holiday_packages`

Browse multi-day holiday packages with itineraries, departure dates, and pricing.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `destination` | string | No | Filter by country or region |
| `duration_days` | integer | No | Filter by exact number of days |
| `limit` | integer | No | Max results to return (default: `20`, max: `50`) |

---

## Examples

### 1. Search activities in Dubai

**Request**
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": 1,
  "params": {
    "name": "search_activities",
    "arguments": {
      "destination": "Dubai",
      "category": "Desert Safari",
      "budget_max": 200,
      "limit": 2
    }
  }
}
```

**Response**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "{\"activities\":[{\"activity_id\":\"a1b2c3d4-0001-0001-0001-000000000001\",\"title\":\"Dubai Evening Desert Safari with BBQ Dinner\",\"description\":\"Experience the magic of the Dubai desert with dune bashing, camel riding, and a live BBQ dinner under the stars.\",\"price_from\":55,\"currency\":\"AED\",\"city\":\"Dubai\",\"country\":\"UAE\",\"rating\":4.8,\"booking_url\":\"https://jtrholidays.com/activity/dubai-evening-desert-safari\"},{\"activity_id\":\"a1b2c3d4-0001-0001-0001-000000000002\",\"title\":\"Dubai Morning Desert Safari with Camel Ride\",\"description\":\"A sunrise desert adventure with sandboarding, camel ride, and a traditional Bedouin breakfast.\",\"price_from\":45,\"currency\":\"AED\",\"city\":\"Dubai\",\"country\":\"UAE\",\"rating\":4.7,\"booking_url\":\"https://jtrholidays.com/activity/dubai-morning-desert-safari\"}],\"total\":2}"
      }
    ]
  }
}
```

---

### 2. Check availability for an activity

**Request**
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": 2,
  "params": {
    "name": "check_availability",
    "arguments": {
      "activity_id": "a1b2c3d4-0001-0001-0001-000000000001",
      "date": "2025-12-25",
      "pax": 2
    }
  }
}
```

**Response**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "{\"available\":true,\"date\":\"2025-12-25\",\"options\":[{\"option\":\"Standard Package\",\"available\":true,\"pax_types\":[{\"type\":\"Adult\",\"note\":\"12+ years\",\"price\":55,\"discount_price\":49,\"currency\":\"AED\"},{\"type\":\"Child\",\"note\":\"3-11 years\",\"price\":40,\"discount_price\":null,\"currency\":\"AED\"},{\"type\":\"Infant\",\"note\":\"Under 3\",\"price\":0,\"discount_price\":null,\"currency\":\"AED\"}]},{\"option\":\"VIP Package\",\"available\":true,\"pax_types\":[{\"type\":\"Adult\",\"note\":\"12+ years\",\"price\":120,\"discount_price\":null,\"currency\":\"AED\"}]}],\"booking_url\":\"https://jtrholidays.com/activity/dubai-evening-desert-safari\"}"
      }
    ]
  }
}
```

---

### 3. List all destinations

**Request**
```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "id": 3,
  "params": {
    "name": "get_destinations",
    "arguments": {}
  }
}
```

**Response**
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "{\"destinations\":[{\"destination\":\"United Arab Emirates\",\"country_code\":\"AE\",\"activity_count\":148,\"popular_activities\":[{\"activity_id\":\"a1b2c3d4-0001-0001-0001-000000000001\",\"title\":\"Dubai Evening Desert Safari with BBQ Dinner\",\"price_from\":55,\"currency\":\"AED\",\"booking_url\":\"https://jtrholidays.com/activity/dubai-evening-desert-safari\"},{\"activity_id\":\"a1b2c3d4-0001-0001-0001-000000000010\",\"title\":\"Burj Khalifa At the Top Tickets\",\"price_from\":149,\"currency\":\"AED\",\"booking_url\":\"https://jtrholidays.com/activity/burj-khalifa-at-the-top\"}]},{\"destination\":\"Maldives\",\"country_code\":\"MV\",\"activity_count\":32,\"popular_activities\":[{\"activity_id\":\"a1b2c3d4-0001-0001-0002-000000000001\",\"title\":\"Maldives Sunset Dolphin Cruise\",\"price_from\":65,\"currency\":\"USD\",\"booking_url\":\"https://jtrholidays.com/activity/maldives-sunset-dolphin-cruise\"}]},{\"destination\":\"Singapore\",\"country_code\":\"SG\",\"activity_count\":27,\"popular_activities\":[{\"activity_id\":\"a1b2c3d4-0001-0001-0003-000000000001\",\"title\":\"Singapore Zoo Day Ticket\",\"price_from\":48,\"currency\":\"SGD\",\"booking_url\":\"https://jtrholidays.com/activity/singapore-zoo-ticket\"}]}]}"
      }
    ]
  }
}
```

---

## Supported Destinations

JTR Holidays offers tours and activities across **24 countries**:

| Region | Countries |
|--------|-----------|
| **Middle East** | 🇦🇪 United Arab Emirates · 🇴🇲 Oman · 🇯🇴 Jordan |
| **South & Southeast Asia** | 🇸🇬 Singapore · 🇻🇳 Vietnam · 🇹🇭 Thailand · 🇲🇾 Malaysia · 🇮🇩 Indonesia · 🇲🇻 Maldives |
| **East Asia** | 🇯🇵 Japan · 🇰🇷 South Korea · 🇭🇰 Hong Kong |
| **Europe** | 🇫🇷 France · 🇮🇹 Italy · 🇪🇸 Spain · 🇳🇱 Netherlands · 🇨🇭 Switzerland · 🇬🇷 Greece · 🇦🇹 Austria · 🇹🇷 Turkey |
| **Oceania** | 🇦🇺 Australia · 🇳🇿 New Zealand |
| **Americas & UK** | 🇬🇧 United Kingdom · 🇺🇸 United States |

Use `get_destinations` to retrieve live activity counts and popular picks for each country.

---

## Error Codes

| Code | HTTP | Meaning |
|------|------|---------|
| `-32600` | 400 | Invalid JSON-RPC request |
| `-32601` | 404 | Method not found |
| `-32602` | 422 | Invalid parameters or resource not found |
| `-32603` | 500 | Internal server error |

---

## Contact & Support

- **Website**: [jtrholidays.com](https://www.jtrholidays.com)
- **MCP Docs**: [jtrholidays.com/mcp-docs](https://www.jtrholidays.com/mcp-docs)
- **Email**: [it@jtrholidays.com](mailto:it@jtrholidays.com)
- **Phone**: +971 4 385 2466
- **Contact form**: [jtrholidays.com/contact-us](https://www.jtrholidays.com/contact-us)

To request API access or report an issue, open a [GitHub Issue](https://github.com/JTR-Holidays/jtr-holidays-mcp/issues) or email us directly.

---

*Powered by [JTR Holidays](https://www.jtrholidays.com) ·*
