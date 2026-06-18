# GSC MCP Server

[![MCP](https://img.shields.io/badge/MCP-1.0-blue)](https://modelcontextprotocol.io)
[![Python](https://img.shields.io/badge/Python-3.10%2B-green)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-yellow)](LICENSE)

A Model Context Protocol (MCP) server that connects Claude Desktop (and other MCP clients) to the Google Search Console API — query search analytics, inspect URLs, manage sitemaps, and compare performance across time periods.

Based on [MCP-GSC](https://github.com/nicholasgcoles/mcp-gsc) by Amin Foroutan, with enhancements for lazy imports, data state control, multi-filter support, and additional tools.

## Features

### 20 Search Console Tools

**Search Analytics:**
| Tool | Description |
|------|-------------|
| `list_properties` | List all GSC properties accessible by the authenticated account |
| `get_search_analytics` | Search analytics with dimensions (query, page, device, country, date) |
| `get_advanced_search_analytics` | Advanced analytics with sorting, filtering, pagination (up to 25K rows) |
| `get_performance_overview` | Performance summary with daily trend data |
| `get_search_by_page_query` | Queries driving traffic to a specific page |
| `compare_search_periods` | Compare two time periods side-by-side with percentage changes |

**URL Inspection:**
| Tool | Description |
|------|-------------|
| `inspect_url_enhanced` | Full URL inspection — indexing status, rich results, canonical info |
| `batch_url_inspection` | Inspect up to 10 URLs in one call |
| `check_indexing_issues` | Categorized indexing issue report across multiple URLs |

**Sitemap Management:**
| Tool | Description |
|------|-------------|
| `get_sitemaps` | List sitemaps with status and URL counts |
| `list_sitemaps_enhanced` | Detailed sitemap listing with submission dates and warnings |
| `get_sitemap_details` | Full details for a specific sitemap |
| `submit_sitemap` | Submit or resubmit a sitemap |
| `delete_sitemap` | Remove a sitemap from GSC |
| `manage_sitemaps` | All-in-one sitemap management (list, details, submit, delete) |

**Property Management:**
| Tool | Description |
|------|-------------|
| `add_site` | Add a site to Search Console |
| `delete_site` | Remove a site from Search Console |
| `get_site_details` | Property details including verification and ownership info |
| `reauthenticate` | Switch Google accounts via OAuth re-login |

### Built-in Reliability

- **Dual auth** — OAuth (interactive) and service account (headless) with automatic fallback
- **Lazy imports** — defers heavy Google client libraries for fast MCP handshake
- **Data state control** — choose between fresh data (`all`, matches GSC dashboard) or confirmed data (`final`)
- **Multi-filter support** — AND logic across dimensions via JSON filter arrays

## Quick Start

### Prerequisites

- Python 3.10+
- A Google Cloud project with the Search Console API enabled
- Claude Desktop (or any MCP-compatible client)

### Authentication Options

#### Option A: Service Account (Headless — recommended for servers)

1. Create a [Google Cloud service account](https://console.cloud.google.com/iam-admin/serviceaccounts)
2. Enable the **Search Console API** in your project
3. Download the service account JSON key file
4. In GSC: go to **Settings > Users and permissions** and add the service account email as a user

#### Option B: OAuth (Interactive — recommended for local use)

1. Create OAuth credentials in [Google Cloud Console](https://console.cloud.google.com/apis/credentials)
2. Download the client secrets JSON as `client_secrets.json`
3. Place it in the `gsc-mcp/` directory
4. On first run, a browser window opens for Google login

### Installation

```bash
git clone https://github.com/luminarylane/gsc-mcp.git
cd gsc-mcp
pip install -r requirements.txt
```

### Configuration

**Claude Desktop (`claude_desktop_config.json`):**
```json
{
  "mcpServers": {
    "gsc": {
      "command": "python",
      "args": ["/path/to/gsc-mcp/gsc_server.py"],
      "env": {
        "GSC_CREDENTIALS_PATH": "/path/to/service_account_credentials.json"
      }
    }
  }
}
```

**Environment variables:**

| Variable | Required | Description |
|----------|----------|-------------|
| `GSC_CREDENTIALS_PATH` | No* | Path to service account JSON key file |
| `GSC_OAUTH_CLIENT_SECRETS_FILE` | No* | Path to OAuth client secrets JSON |
| `GSC_SKIP_OAUTH` | No | Set to `true` to skip OAuth and use service account only |
| `GSC_DATA_STATE` | No | `all` (default, matches GSC dashboard) or `final` (confirmed data, 2-3 day lag) |

*At least one authentication method must be configured.

Alternatively, place `service_account_credentials.json` or `client_secrets.json` in the `gsc-mcp/` directory.

## Usage Examples

Once configured, ask Claude to:

- "List my Search Console properties"
- "What are the top queries for my site this month?"
- "Show me the top pages by clicks for the last 90 days"
- "Is this URL indexed? Check https://example.com/my-page"
- "Compare search performance: last 28 days vs the 28 days before"
- "Check indexing issues for these URLs" (paste a list)
- "Submit my sitemap at https://example.com/sitemap.xml"
- "Show traffic sources by country and device"

## Data Freshness

By default, the server uses `dataState: "all"` which includes fresh/unconfirmed data and matches the GSC dashboard. Set `GSC_DATA_STATE=final` for confirmed-only data (lags 2-3 days). You can also override per-call via the `data_state` parameter in `get_advanced_search_analytics`.

## Troubleshooting

### Property not found (404)

The `site_url` must exactly match what GSC shows. Run `list_properties` first. Domain properties use the format `sc-domain:example.com`, not a full URL.

### Permission denied (403)

The authenticated account needs access to the property. For service accounts, add the email in GSC **Settings > Users and permissions**.

### Authentication failed

- **Service account**: Check `GSC_CREDENTIALS_PATH` points to a valid JSON key file
- **OAuth**: Delete `token.json` and re-run to trigger a fresh login, or use the `reauthenticate` tool

## Contributing

1. Fork the repo
2. Create a feature branch (`git checkout -b feat/my-feature`)
3. Make changes and test locally
4. Submit a pull request

## License

MIT License — see [LICENSE](LICENSE) for details.

## Acknowledgments

- [Amin Foroutan](https://aminforoutan.com/) for the original [MCP-GSC](https://github.com/nicholasgcoles/mcp-gsc) tool
- [Anthropic](https://anthropic.com) for the MCP specification
- [Google Search Console API](https://developers.google.com/webmaster-tools) for the underlying API
