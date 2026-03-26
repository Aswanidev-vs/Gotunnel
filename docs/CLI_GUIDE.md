# GoTunnel CLI User Guide

## Table of Contents

1. [Installation](#installation)
2. [Quick Start](#quick-start)
3. [Global Options](#global-options)
4. [Commands Reference](#commands-reference)
5. [Configuration](#configuration)
6. [Authentication](#authentication)
7. [Tunnel Management](#tunnel-management)
8. [Certificate Management](#certificate-management)
9. [DNS Management](#dns-management)
10. [Security Policies](#security-policies)
11. [Collaborative Sessions](#collaborative-sessions)
12. [Plugins](#plugins)
13. [Profiles](#profiles)
14. [CI/CD Integration](#cicd-integration)
15. [Containerization](#containerization)
16. [Shell Completion](#shell-completion)
17. [Troubleshooting](#troubleshooting)

---

## Installation

### From Source

```bash
# Clone the repository
git clone https://github.com/portless/gotunnel.git
cd GoTunnel

# Build
go build -o gotunnel ./cmd/gotunnel

# Install to $GOPATH/bin
go install ./cmd/gotunnel
```

### Using Docker

```bash
docker pull gotunnel/gotunnel:latest
docker run -d --name gotunnel -p 8080:8080 gotunnel/gotunnel:latest
```

### Using Package Managers

```bash
# Homebrew (macOS)
brew install gotunnel

# Snap (Linux)
sudo snap install gotunnel

# Chocolatey (Windows)
choco install gotunnel
```

---

## Quick Start

### 1. Authenticate

```bash
# Login with token
gotunnel login --token YOUR_TOKEN

# Or set environment variable
export GOTUNNEL_TOKEN=YOUR_TOKEN
```

### 2. Start a Tunnel

```bash
# Basic HTTP tunnel
gotunnel tunnel start --name myapp --local-url http://localhost:3000

# With custom subdomain
gotunnel tunnel start --name myapp --subdomain myapp --local-url http://localhost:3000

# TCP tunnel
gotunnel tunnel start --name db --protocol tcp --local-url localhost:5432
```

### 3. List Tunnels

```bash
gotunnel tunnel list
```

---

## Global Options

Global options can be used with any command:

```bash
gotunnel [global-options] <command> [command-options]
```

| Option | Short | Description | Default |
|--------|-------|-------------|---------|
| `--config` | `-c` | Path to configuration file | `gotunnel.yaml` |
| `--log-level` | `-l` | Log level (trace, debug, info, warn, error, fatal) | `info` |
| `--log-format` | | Log format (text, json) | `text` |
| `--json` | | Output in JSON format | `false` |
| `--dry-run` | | Show what would be done without executing | `false` |
| `--verbose` | `-v` | Enable verbose output | `false` |
| `--quiet` | `-q` | Suppress non-error output | `false` |
| `--no-color` | | Disable colored output | `false` |
| `--token` | `-t` | Authentication token | |
| `--profile` | `-p` | Configuration profile (dev, staging, prod) | |
| `--timeout` | | Command timeout (e.g., 30s, 5m) | |
| `--help` | `-h` | Show help | |

### Examples

```bash
# Use a specific config file
gotunnel --config /path/to/config.yaml tunnel start

# Enable debug logging
gotunnel --log-level debug tunnel start

# Output in JSON format
gotunnel --json tunnel list

# Dry run mode
gotunnel --dry-run tunnel start --name myapp

# Verbose output
gotunnel --verbose tunnel start --name myapp

# Quiet mode (errors only)
gotunnel --quiet tunnel list

# Use a specific profile
gotunnel --profile prod tunnel start

# Set timeout
gotunnel --timeout 5m tunnel start
```

---

## Commands Reference

### Login

Authenticate with GoTunnel by storing an access token.

```bash
gotunnel login --token <token>
```

**Options:**
- `--token`: Access token to store (required)

**Example:**
```bash
gotunnel login --token abc123xyz
```

---

### Broker

Start the broker server for tunnel coordination.

```bash
gotunnel broker [options]
```

**Options:**
- `--listen`: Address to listen on (default: `:8090`)
- `--cert-file`: TLS certificate file
- `--key-file`: TLS key file

**Example:**
```bash
gotunnel broker --listen :8090
gotunnel broker --listen :8090 --cert-file cert.pem --key-file key.pem
```

---

### Relay

Start the relay server for tunnel traffic forwarding.

```bash
gotunnel relay [options]
```

**Options:**
- `--listen`: Address to listen on (default: `:8091`)
- `--broker-url`: Broker URL (default: `http://127.0.0.1:8090`)
- `--region`: Relay region
- `--cert-file`: TLS certificate file
- `--key-file`: TLS key file

**Example:**
```bash
gotunnel relay --listen :8091
gotunnel relay --listen :8091 --broker-url https://broker.example.com --region us-east-1
```

---

### Config

Manage configuration files.

```bash
gotunnel config <subcommand> [options]
```

**Subcommands:**
- `validate`: Validate a configuration file
- `init`: Initialize a new configuration file
- `show`: Show current configuration

**Examples:**
```bash
# Validate configuration
gotunnel config validate --file gotunnel.yaml

# Initialize new config
gotunnel config init

# Show current configuration
gotunnel config show
```

---

### Tunnel

Manage tunnel lifecycle.

```bash
gotunnel tunnel <subcommand> [options]
```

**Subcommands:**
- `start`: Start a new tunnel
- `stop`: Stop a running tunnel
- `list`: List all tunnels
- `inspect`: Inspect tunnel traffic
- `share`: Create collaborative debugging session

#### Start a Tunnel

```bash
gotunnel tunnel start [options]
```

**Options:**
- `--name`: Tunnel name (default: `app`)
- `--protocol`: Protocol: http, tcp, udp (default: `http`)
- `--local-url`: Local service URL (default: `http://127.0.0.1:3000`)
- `--subdomain`: Requested subdomain
- `--https`: HTTPS mode: auto, manual, disabled (default: `auto`)
- `--inspect`: Enable inspection (default: `true`)
- `--production`: Mark as production (default: `false`)
- `--allow-sharing`: Allow collaborative sharing (default: `true`)

**Examples:**
```bash
# Basic HTTP tunnel
gotunnel tunnel start --name myapp --local-url http://localhost:3000

# Custom subdomain
gotunnel tunnel start --name myapp --subdomain myapp --local-url http://localhost:3000

# TCP tunnel
gotunnel tunnel start --name db --protocol tcp --local-url localhost:5432

# Production tunnel with HTTPS
gotunnel tunnel start --name prod --local-url http://localhost:8080 --production --https auto
```

#### List Tunnels

```bash
gotunnel tunnel list
gotunnel tunnel list --json
```

#### Stop a Tunnel

```bash
gotunnel tunnel stop --name myapp
```

#### Inspect Traffic

```bash
gotunnel tunnel inspect --name myapp
```

#### Share a Tunnel

```bash
gotunnel tunnel share --name myapp
gotunnel tunnel share --name myapp --invite user@example.com
```

---

### Certificate

Manage TLS certificates.

```bash
gotunnel cert <subcommand> [options]
```

**Subcommands:**
- `list`: List all certificates
- `renew`: Renew a certificate
- `revoke`: Revoke a certificate
- `info`: Show certificate information

**Examples:**
```bash
# List certificates
gotunnel cert list

# Renew certificate
gotunnel cert renew --domain example.com

# Revoke certificate
gotunnel cert revoke --domain example.com

# Show certificate info
gotunnel cert info --domain example.com
```

---

### DNS

Manage DNS providers and records.

```bash
gotunnel dns <subcommand> [options]
```

**Subcommands:**
- `providers`: List DNS providers
- `records`: List DNS records
- `test`: Test DNS propagation

**Examples:**
```bash
# List DNS providers
gotunnel dns providers

# List DNS records
gotunnel dns records --zone example.com

# Test DNS propagation
gotunnel dns test --domain app.example.com
```

---

### Security

Manage security policies.

```bash
gotunnel security <subcommand> [options]
```

**Subcommands:**
- `policies`: List security policies
- `network`: Show network security
- `tls`: Show TLS configuration
- `rate-limit`: Show rate limit status
- `audit`: Show audit log

**Examples:**
```bash
# List security policies
gotunnel security policies

# Show network security
gotunnel security network

# Show TLS configuration
gotunnel security tls

# Show rate limit status
gotunnel security rate-limit

# Show audit log
gotunnel security audit
```

---

### Session

Manage collaborative debugging sessions.

```bash
gotunnel session <subcommand> [options]
```

**Subcommands:**
- `list`: List active sessions
- `join`: Join a collaborative session
- `annotate`: Add annotation to a request
- `replay`: Replay a request

**Examples:**
```bash
# List sessions
gotunnel session list

# Join a session
gotunnel session join --token SESSION_TOKEN

# Annotate a request
gotunnel session annotate --request-id REQ_ID --message "This is failing"

# Replay a request
gotunnel session replay --request-id REQ_ID
```

---

### Plugin

Manage CLI plugins.

```bash
gotunnel plugin <subcommand> [options]
```

**Subcommands:**
- `list`: List installed plugins
- `load`: Load a plugin from file
- `unload`: Unload a plugin

**Examples:**
```bash
# List plugins
gotunnel plugin list

# Load a plugin
gotunnel plugin load /path/to/plugin.so

# Unload a plugin
gotunnel plugin unload plugin-name
```

---

### Profile

Manage environment profiles.

```bash
gotunnel profile <subcommand> [options]
```

**Subcommands:**
- `list`: List available profiles
- `show`: Show profile details
- `create`: Create a new profile
- `delete`: Delete a profile
- `apply`: Apply a profile

**Examples:**
```bash
# List profiles
gotunnel profile list

# Show profile
gotunnel profile show dev

# Create profile
gotunnel profile create --name dev --env dev --description "Development profile"

# Delete profile
gotunnel profile delete dev

# Apply profile
gotunnel profile apply dev
```

---

### CICD

CI/CD integration.

```bash
gotunnel cicd <subcommand> [options]
```

**Subcommands:**
- `detect`: Detect CI/CD environment
- `generate`: Generate CI/CD configuration
- `secrets`: Manage secrets

**Examples:**
```bash
# Detect CI environment
gotunnel cicd detect

# Generate GitHub Actions config
gotunnel cicd generate --provider github

# Generate GitLab CI config
gotunnel cicd generate --provider gitlab

# Generate Jenkinsfile
gotunnel cicd generate --provider jenkins

# Generate CircleCI config
gotunnel cicd generate --provider circleci
```

---

### Container

Containerization support.

```bash
gotunnel container <subcommand> [options]
```

**Subcommands:**
- `generate`: Generate container configs

**Examples:**
```bash
# Generate Dockerfile
gotunnel container generate --type dockerfile

# Generate docker-compose.yaml
gotunnel container generate --type docker-compose

# Generate Kubernetes manifests
gotunnel container generate --type kubernetes

# Generate Helm chart
gotunnel container generate --type helm
```

---

### Completion

Generate shell completion scripts.

```bash
gotunnel completion <shell>
```

**Arguments:**
- `shell`: Shell type (bash, zsh, fish)

**Examples:**
```bash
# Generate bash completion
gotunnel completion bash

# Generate zsh completion
gotunnel completion zsh

# Generate fish completion
gotunnel completion fish
```

**Installation:**

```bash
# Bash - add to ~/.bashrc
eval "$(gotunnel completion bash)"

# Zsh - add to ~/.zshrc
eval "$(gotunnel completion zsh)"

# Fish - save to completions directory
gotunnel completion fish > ~/.config/fish/completions/gotunnel.fish
```

---

### Version

Show version information.

```bash
gotunnel version
```

---

### Help

Show help for a command.

```bash
gotunnel help [command]
```

**Examples:**
```bash
# Show general help
gotunnel help

# Show help for tunnel command
gotunnel help tunnel

# Show help for tunnel start subcommand
gotunnel help tunnel start
```

---

## Configuration

### Configuration File

GoTunnel uses YAML configuration files. The default location is `gotunnel.yaml`.

```yaml
version: 1

auth:
  token: ${GOTUNNEL_TOKEN}
  session_timeout: 24h
  max_concurrent_sessions: 5
  require_mfa: false

relay:
  broker_url: https://relay.example.com
  region: us-east-1

tunnels:
  - name: web
    protocol: http
    local_url: http://localhost:3000
    subdomain: myapp
    https: auto
    inspect: true
    production: false
    allow_sharing: true

acme:
  email: admin@example.com
  directory_url: https://acme-v02.api.letsencrypt.org/directory
  renewal_window: 720h
  max_retries: 5

dns:
  providers:
    - name: cloudflare
      type: cloudflare
      credentials:
        api_token: ${CLOUDFLARE_API_TOKEN}
      zones:
        - example.com

security:
  network:
    allowed_cidrs:
      - 10.0.0.0/8
    blocked_cidrs: []
  tls:
    min_version: "1.3"
  rate_limit:
    enabled: true
    requests_per_second: 100
    burst_size: 200
```

### Environment Variables

Environment variables can be used in configuration files with the `env:` prefix:

```yaml
auth:
  token: env:GOTUNNEL_TOKEN
dns:
  providers:
    - name: cloudflare
      credentials:
        api_token: env:CLOUDFLARE_API_TOKEN
```

### Validate Configuration

```bash
gotunnel config validate --file gotunnel.yaml
```

---

## Authentication

### Token-Based Authentication

```bash
# Login with token
gotunnel login --token YOUR_TOKEN

# Or set environment variable
export GOTUNNEL_TOKEN=YOUR_TOKEN

# Or pass token directly
gotunnel --token YOUR_TOKEN tunnel start
```

### MFA (Multi-Factor Authentication)

When MFA is enabled, you'll need to provide a TOTP code:

```bash
gotunnel tunnel start --name myapp
# Enter MFA code: 123456
```

### SSO (Single Sign-On)

For SSO authentication:

```bash
gotunnel login --sso-provider okta
# This will open a browser for authentication
```

---

## Tunnel Management

### Creating Tunnels

```bash
# HTTP tunnel
gotunnel tunnel start \
  --name web \
  --protocol http \
  --local-url http://localhost:3000 \
  --subdomain myapp \
  --https auto

# TCP tunnel
gotunnel tunnel start \
  --name postgres \
  --protocol tcp \
  --local-url localhost:5432

# UDP tunnel
gotunnel tunnel start \
  --name game \
  --protocol udp \
  --local-url localhost:7777
```

### Using Configuration File

```bash
# Start tunnel from config
gotunnel tunnel start --config gotunnel.yaml

# Start specific tunnel from config
gotunnel tunnel start --config gotunnel.yaml --name web
```

### Tunnel Status

```bash
# List all tunnels
gotunnel tunnel list

# List in JSON format
gotunnel tunnel list --json

# Inspect specific tunnel
gotunnel tunnel inspect --name web
```

---

## Certificate Management

### Automatic HTTPS

GoTunnel automatically manages TLS certificates for HTTP tunnels:

```yaml
tunnels:
  - name: web
    protocol: http
    local_url: http://localhost:3000
    https: auto  # Enable automatic HTTPS
```

### Manual Certificate Management

```bash
# List certificates
gotunnel cert list

# Get certificate info
gotunnel cert info --domain example.com

# Force renewal
gotunnel cert renew --domain example.com

# Revoke certificate
gotunnel cert revoke --domain example.com
```

### ACME Configuration

```yaml
acme:
  email: admin@example.com
  directory_url: https://acme-v02.api.letsencrypt.org/directory
  renewal_window: 720h  # 30 days
  max_retries: 5
  preferred_challenge: dns-01
```

---

## DNS Management

### DNS Providers

Configure DNS providers for DNS-01 challenges:

```yaml
dns:
  providers:
    - name: cloudflare
      type: cloudflare
      credentials:
        api_token: ${CLOUDFLARE_API_TOKEN}
      zones:
        - example.com
        - example.org
```

### DNS Commands

```bash
# List providers
gotunnel dns providers

# List records for a zone
gotunnel dns records --zone example.com

# Test DNS propagation
gotunnel dns test --domain app.example.com
```

---

## Security Policies

### Network Policies

```yaml
security:
  network:
    allowed_cidrs:
      - 10.0.0.0/8      # Corporate network
      - 172.16.0.0/12    # VPN
    blocked_cidrs:
      - 1.2.3.0/24       # Blocked IPs
    max_connections: 10000
    timeout: 30s
```

### TLS Policies

```yaml
security:
  tls:
    min_version: "1.3"
    max_version: "1.3"
    cipher_suites:
      - TLS_AES_256_GCM_SHA384
      - TLS_CHACHA20_POLY1305_SHA256
    require_client_cert: false
```

### Rate Limiting

```yaml
security:
  rate_limit:
    enabled: true
    requests_per_second: 100
    burst_size: 200
    window_size: 1m
    key_field: ip  # or "user", "tunnel"
```

### Security Commands

```bash
# View security policies
gotunnel security policies

# Check network security
gotunnel security network

# Check TLS configuration
gotunnel security tls

# Check rate limit status
gotunnel security rate-limit

# View audit log
gotunnel security audit
```

---

## Collaborative Sessions

### Creating Sessions

```bash
# Share a tunnel
gotunnel tunnel share --name myapp

# Invite specific users
gotunnel tunnel share --name myapp --invite user@example.com
```

### Joining Sessions

```bash
# Join with session token
gotunnel session join --token SESSION_TOKEN
```

### Session Commands

```bash
# List active sessions
gotunnel session list

# Annotate a request
gotunnel session annotate \
  --request-id REQ_ID \
  --message "This request is failing"

# Replay a request
gotunnel session replay --request-id REQ_ID

# Replay with modifications
gotunnel session replay \
  --request-id REQ_ID \
  --modify-header "X-Debug: true"
```

---

## Plugins

### Loading Plugins

```bash
# List installed plugins
gotunnel plugin list

# Load a plugin
gotunnel plugin load /path/to/plugin.so

# Unload a plugin
gotunnel plugin unload plugin-name
```

### Plugin Development

Plugins must export a `Plugin` symbol:

```go
package main

import "gotunnel/internal/cli"

var Plugin = &cli.Plugin{
    Name:        "my-plugin",
    Version:     "1.0.0",
    Description: "My custom plugin",
    Commands: map[string]*cli.Command{
        "mycommand": {
            Name:  "mycommand",
            Short: "My custom command",
            Run:   runMyCommand,
        },
    },
}

func runMyCommand(args []string) error {
    // Implementation
    return nil
}
```

Build the plugin:

```bash
go build -buildmode=plugin -o myplugin.so plugin.go
```

---

## Profiles

### Creating Profiles

```bash
# Create development profile
gotunnel profile create \
  --name dev \
  --env dev \
  --description "Development environment"

# Create production profile
gotunnel profile create \
  --name prod \
  --env prod \
  --description "Production environment"
```

### Using Profiles

```bash
# List profiles
gotunnel profile list

# Show profile details
gotunnel profile show dev

# Apply profile
gotunnel profile apply dev

# Use profile with command
gotunnel --profile prod tunnel start --name myapp
```

### Profile Storage

Profiles are stored in `~/.gotunnel/profiles/`:

```yaml
# ~/.gotunnel/profiles/prod.yaml
name: prod
description: Production environment
environment: prod
variables:
  GOTUNNEL_TOKEN: prod-token-123
  DATABASE_URL: postgres://prod-db:5432/gotunnel
config:
  relay:
    broker_url: https://prod-broker.example.com
```

---

## CI/CD Integration

### Detecting CI Environment

```bash
gotunnel cicd detect
```

Output:
```json
{
  "provider": "github",
  "branch": "main",
  "commit": "abc123",
  "build_id": "12345",
  "is_ci": true
}
```

### Generating CI Configs

```bash
# GitHub Actions
gotunnel cicd generate --provider github > .github/workflows/deploy.yml

# GitLab CI
gotunnel cicd generate --provider gitlab > .gitlab-ci.yml

# Jenkins
gotunnel cicd generate --provider jenkins > Jenkinsfile

# CircleCI
gotunnel cicd generate --provider circleci > .circleci/config.yml
```

### Secret Management

```bash
# List available secret providers
gotunnel cicd secrets

# Get a secret
gotunnel cicd secrets get --provider vault --key GOTUNNEL_TOKEN

# Set a secret
gotunnel cicd secrets set --provider vault --key GOTUNNEL_TOKEN --value abc123
```

---

## Containerization

### Generate Dockerfile

```bash
gotunnel container generate --type dockerfile > Dockerfile
```

### Generate Docker Compose

```bash
gotunnel container generate --type docker-compose > docker-compose.yaml
```

### Generate Kubernetes Manifests

```bash
gotunnel container generate --type kubernetes > k8s.yaml
```

### Generate Helm Chart

```bash
gotunnel container generate --type helm > helm-chart.yaml
```

### Docker Usage

```bash
# Build image
docker build -t gotunnel .

# Run container
docker run -d \
  --name gotunnel \
  -p 8080:8080 \
  -p 8443:8443 \
  -e GOTUNNEL_TOKEN=your-token \
  gotunnel

# Run with docker-compose
docker-compose up -d
```

### Kubernetes Usage

```bash
# Apply manifests
kubectl apply -f k8s.yaml

# Check status
kubectl get pods -n gotunnel

# View logs
kubectl logs -f deployment/gotunnel -n gotunnel
```

---

## Shell Completion

### Bash

```bash
# Add to ~/.bashrc
eval "$(gotunnel completion bash)"

# Or save to file
gotunnel completion bash > /etc/bash_completion.d/gotunnel
```

### Zsh

```bash
# Add to ~/.zshrc
eval "$(gotunnel completion zsh)"

# Or save to file
gotunnel completion zsh > ~/.zsh/completions/_gotunnel
```

### Fish

```bash
# Save to completions directory
gotunnel completion fish > ~/.config/fish/completions/gotunnel.fish
```

---

## Troubleshooting

### Common Issues

#### Authentication Failed

```bash
# Check token
echo $GOTUNNEL_TOKEN

# Re-login
gotunnel login --token NEW_TOKEN

# Clear saved token
rm ~/.config/gotunnel/token
```

#### Connection Refused

```bash
# Check if broker is running
curl http://localhost:8090/health

# Check firewall rules
sudo iptables -L

# Check network connectivity
ping relay.example.com
```

#### Certificate Errors

```bash
# Check certificate status
gotunnel cert info --domain example.com

# Force renewal
gotunnel cert renew --domain example.com

# Check ACME logs
gotunnel --log-level debug cert renew --domain example.com
```

#### DNS Propagation Issues

```bash
# Test DNS
gotunnel dns test --domain app.example.com

# Check DNS records
dig TXT _acme-challenge.app.example.com

# Force DNS update
gotunnel dns update --domain app.example.com
```

### Debug Mode

Enable debug logging for troubleshooting:

```bash
gotunnel --log-level debug tunnel start --name myapp
```

### Dry Run Mode

Test commands without executing:

```bash
gotunnel --dry-run tunnel start --name myapp
gotunnel --dry-run cert renew --domain example.com
```

### Verbose Output

Get detailed output:

```bash
gotunnel --verbose tunnel start --name myapp
```

### JSON Output

Machine-readable output:

```bash
gotunnel --json tunnel list
gotunnel --json cert list
gotunnel --json security policies
```

---

## Examples

### Development Workflow

```bash
# 1. Login
export GOTUNNEL_TOKEN=dev-token-123
gotunnel login --token $GOTUNNEL_TOKEN

# 2. Start local service
npm start  # or go run main.go

# 3. Create tunnel
gotunnel tunnel start \
  --name dev \
  --local-url http://localhost:3000 \
  --subdomain mydev

# 4. Share with team
gotunnel tunnel share --name dev --invite team@example.com

# 5. Inspect traffic
gotunnel tunnel inspect --name dev
```

### Production Deployment

```bash
# 1. Apply production profile
gotunnel profile apply prod

# 2. Validate configuration
gotunnel config validate --file gotunnel-prod.yaml

# 3. Start production tunnel
gotunnel tunnel start \
  --name production \
  --config gotunnel-prod.yaml \
  --production

# 4. Monitor
gotunnel tunnel list
gotunnel cert list
gotunnel security audit
```

### CI/CD Pipeline

```bash
# In your CI/CD pipeline
export GOTUNNEL_TOKEN=${{ secrets.GOTUNNEL_TOKEN }}

# Validate config
gotunnel config validate --file gotunnel.yaml

# Deploy
gotunnel tunnel start \
  --name ci-deploy \
  --profile staging

# Verify
gotunnel tunnel list --json
```

---

## See Also

- [Architecture Documentation](architecture.md)
- [Configuration Reference](configuration.md)
- [Security Best Practices](security.md)
- [API Reference](api.md)
- [Plugin Development Guide](plugins.md)