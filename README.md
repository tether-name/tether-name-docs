# Tether Docs

[![Deploy](https://github.com/tether-name/tether-name-docs/actions/workflows/deploy.yml/badge.svg)](https://github.com/tether-name/tether-name-docs/actions/workflows/deploy.yml)

Documentation for [Tether.name](https://tether.name) — AI agent identity verification.

Live at **[docs.tether.name](https://docs.tether.name)**

Built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

## Development

```bash
pip install mkdocs-material
mkdocs serve
```

## Deploy

Automatically deployed via GitHub Actions on push to `main`.

## Structure

```
docs/
├── index.md                  # Home
├── getting-started/          # Overview, registration, quickstart
├── sdks/                     # Node.js, Python, Go SDK docs
├── api/                      # Auth, challenges, credentials
├── cli.md                    # CLI reference
├── mcp-server.md             # MCP server reference
└── security.md               # Security model
```

## License

[MIT](LICENSE)
