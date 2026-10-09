> **Note:** This fork is maintained and updated exclusively for use in my personal homelab. It is not a community project — contributions are welcome as issues or PRs, but development follows my own needs.

<div align="center">
  <img src="banner.svg" alt="hemmelig" />
</div>

<h1 align="center">Hemmelig - Encrypted Secret Sharing</h1>

<p align="center">
  Share sensitive information securely with client-side encryption and self-destructing messages.
</p>

<p align="center">
  <a href="#quick-start">Quick Start</a> •
  <a href="docs/docker.md">Docker Guide</a> •
  <a href="docs/env.md">Configuration</a>
</p>

<p align="center">
  <a href="https://github.com/danielsreichenbach/Hemmelig.app/pkgs/container/hemmelig"><img src="https://img.shields.io/badge/GHCR-available-blue" alt="GHCR image" /></a>
</p>

## How It Works

1. Create a secret using the web app, CLI, or API of your self-hosted instance
2. Set expiration time, view limits, and optional password
3. Share the generated link with your recipient
4. The secret is automatically deleted after being viewed or expired

**Zero-knowledge architecture:** All encryption happens in your browser. The server only stores encrypted data and never sees your secrets or encryption keys.

## Features

- **Client-side AES-256-GCM encryption** - Your data is encrypted before leaving your browser
- **Self-destructing secrets** - Configurable expiration and view limits
- **Password protection** - Optional additional security layer
- **IP restrictions** - Limit access to specific IP ranges
- **File uploads** - Share encrypted files (authenticated users)
- **Rich text editor** - Format your secrets with styling
- **QR codes** - Easy mobile sharing
- **Multi-language support** - Available in multiple languages
- **Webhook notifications** - Get notified when secrets are viewed or burned ([docs](docs/webhook.md))

## Quick Start

### Docker (Recommended)

```bash
docker run -d \
  --name hemmelig \
  -p 3000:3000 \
  -v hemmelig-data:/app/database \
  -v hemmelig-uploads:/app/uploads \
  -e DATABASE_URL="file:/app/database/hemmelig.db" \
  -e BETTER_AUTH_SECRET="$(openssl rand -base64 32)" \
  -e BETTER_AUTH_URL="https://your-domain.com" \
  hemmelig/hemmelig:v7
```

### Docker Compose

```bash
git clone https://github.com/danielsreichenbach/Hemmelig.app.git
cd Hemmelig.app

# Edit docker-compose.yml with your settings
docker compose up -d
```

See [Docker Guide](docs/docker.md) for detailed deployment instructions.

### CLI

The `hemmelig` CLI does all that the web app does, with the access of your account. It encrypts and decrypts on your machine.

```bash
# Download the binary for Linux on amd64. Other platforms are in docs/cli.md.
curl -L https://github.com/danielsreichenbach/Hemmelig.app/releases/download/cli-v1.1.0/hemmelig-linux-amd64 -o hemmelig
chmod +x hemmelig

# Create a secret and print the link.
./hemmelig "API key: sk-1234" -t "Production API Key" -e 7d -v 3

# Read a secret, then work with your own instance.
./hemmelig secrets get "https://secrets.example.com/s/<id>#<key>"
./hemmelig login --url https://secrets.example.com
./hemmelig admin users list --json
```

The CLI also runs a local MCP server, so AI assistants can use Hemmelig with an API key:

```bash
HEMMELIG_API_KEY=hemmelig_xxxxxxxxxxxxxxxx ./hemmelig mcp
```

- [CLI documentation](docs/cli.md): install, authentication, every command, JSON output and exit codes.
- [MCP server](docs/mcp.md): Claude Desktop and Claude Code setup, the tools and the security model.

## Documentation

- [Docker Deployment](docs/docker.md) - Complete Docker setup guide
- [Helm Chart](docs/helm.md) - Kubernetes deployment with Helm
- [Environment Variables](docs/env.md) - All configuration options
- [Managed Mode](docs/managed.md) - Configure instance settings via environment variables
- [CLI](docs/cli.md) - Command-line interface for automation and CI/CD
- [MCP Server](docs/mcp.md) - Let AI agents create and read secrets with local encryption
- [API Keys](docs/api-keys.md) - Use the API with a key, and the actions that need a session
- [Encryption](docs/encryption.md) - How client-side encryption works
- [Social Login](docs/social-login.md) - OAuth provider setup (GitHub, Google, etc.)
- [Secret Requests](docs/secret-request.md) - Request secrets from others securely
- [Webhooks](docs/webhook.md) - Webhook notifications for secret events
- [Health Checks](docs/health.md) - Liveness and readiness probes for container orchestration
- [Prometheus Metrics](docs/metrics.md) - Monitor your instance with Prometheus
- [API Documentation](docs/api.md) - REST API reference and OpenAPI spec
- [SDK Generation](docs/sdk.md) - Generate client SDKs from OpenAPI spec
- [E2E Testing](docs/e2e.md) - End-to-end testing with Playwright
- [Upgrading from v6](docs/upgrade.md) - Migration guide for v6 to v7

## Development

```bash
npm install
npm run dev
npm run dev:api
```

## License

O'Saasy License Agreement - Copyright © 2025, Bjarne Øverli.

This project is licensed under a modified MIT license that prohibits using the software to compete with the original licensor as a hosted SaaS product. See [LICENSE](LICENSE) for details.
This fork retains the same license terms.
