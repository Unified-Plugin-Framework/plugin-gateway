# UPF API Gateway

Core API Gateway service for the Unified Plugin Framework.

## Overview

The API Gateway is a core UPF service that:

- **Routes requests** - Routes incoming requests to appropriate backend plugins
- **Translates protocols** - Converts gRPC-Web to native gRPC
- **Validates authentication** - Validates tokens via the Auth plugin
- **Rate limiting** - Protects backend services from overload
- **Serves manifests** - Provides plugin manifests to the UI Shell
- **WebSocket support** - Handles real-time bidirectional communication

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Clients                               │
│   Web (gRPC-Web)  │  iOS (gRPC)  │  Android (gRPC)          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      API Gateway                             │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │   Router    │  │  Protocol   │  │    Rate     │          │
│  │             │  │  Translator │  │   Limiter   │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │    Auth     │  │  Manifest   │  │  WebSocket  │          │
│  │  Validator  │  │   Server    │  │   Handler   │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Backend Plugins                           │
│  Plugin A  │  Plugin B  │  Plugin C  │  Auth Plugin         │
└─────────────────────────────────────────────────────────────┘
```

## Quick Start

### Using Docker

```bash
docker run -d \
  --name upf-gateway \
  -p 8080:8080 \
  -p 50051:50051 \
  -e REGISTRY_URL=registry:50052 \
  -e AUTH_URL=auth:50053 \
  ghcr.io/unified-plugin-framework/plugin-gateway:latest
```

### Using Docker Compose

```yaml
services:
  gateway:
    image: ghcr.io/unified-plugin-framework/plugin-gateway:latest
    ports:
      - "8080:8080"    # HTTP/gRPC-Web
      - "50051:50051"  # gRPC
    environment:
      REGISTRY_URL: registry:50052
      AUTH_URL: auth:50053
      RATE_LIMIT_RPS: 100
    depends_on:
      - registry
      - auth
```

## Endpoints

| Port | Protocol | Description |
|------|----------|-------------|
| `8080` | HTTP/gRPC-Web/WebSocket | Client-facing endpoint |
| `50051` | gRPC | Native gRPC endpoint |

## Configuration

| Environment Variable | Description | Default |
|---------------------|-------------|----------|
| `HTTP_PORT` | HTTP/gRPC-Web port | `8080` |
| `GRPC_PORT` | Native gRPC port | `50051` |
| `REGISTRY_URL` | Plugin Registry URL | Required |
| `AUTH_URL` | Auth plugin URL | Required |
| `RATE_LIMIT_RPS` | Requests per second limit | `100` |
| `LOG_LEVEL` | Logging level | `info` |

## Features

### Request Routing

The gateway automatically routes requests based on service definitions from the Plugin Registry.

### Protocol Translation

Converts gRPC-Web requests from browsers to native gRPC calls:
- Supports both `application/grpc-web` and `application/grpc-web-text`
- Handles streaming for server-streaming and bidirectional RPCs

### Authentication

All requests are validated against the Auth plugin:
- Bearer token validation
- Permission checking
- Token refresh handling

### Rate Limiting

Protects backend services with configurable rate limits:
- Per-client rate limiting
- Per-endpoint limits
- Burst handling

## Development

```bash
# Clone the repository
git clone https://github.com/Unified-Plugin-Framework/plugin-gateway.git
cd plugin-gateway

# Install dependencies
bun install

# Run in development mode
bun run dev

# Run tests
bun test

# Build Docker image
docker build -t plugin-gateway .
```

## Documentation

See the [Architecture Overview](https://unified-plugin-framework.github.io/docs/architecture/overview) for complete details.

## License

MIT
