# JumpServer MCP Server

## 1) Prepare .env

Create a `.env` file in the project root:

```ini
# JumpServer host (without /api/v1)
jumpserver_url=http://<jumpserver-host>

# Optional: server reads Swagger using this token (if required)
api_token=<JumpServerBearer>

# Optional inbound auth for this MCP server
# If set, client must send: Authorization: Bearer <api_key>
# api_key=<server-inbound-token>

# Optional overrides
# server_port=8099
# base_path=/sse
# log_level=INFO
```

Note: Keep jumpserver_url as host root (e.g., `http://192.168.1.10`). Do not append `/api/v1`.

## 2) Build and run locally with Docker

Note: Docker requires the Buildx plugin to build this image (Linux manual install example):

```bash
mkdir -p ~/.docker/cli-plugins
curl -L https://github.com/docker/buildx/releases/latest/download/buildx-v0.29.1.linux-amd64 \
  -o ~/.docker/cli-plugins/docker-buildx
chmod +x ~/.docker/cli-plugins/docker-buildx
```

```bash
# Build image from local source
docker build -t jms-mcp:local .

# Run container using your .env
docker run -d --name jms_mcp \
  --env-file .env -p 8099:8099 jms-mcp:local
```

## 3) Get a JumpServer Bearer token (example)

```bash
TOKEN=$(curl -s -X POST http://<jumpserver-host>/api/v1/authentication/auth/ \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"xxxx"}' \
  --insecure | jq -r '.token')
echo "Your Bearer token: $TOKEN"
```

## 4) Connect an MCP client

The MCP server forwards the client's `Authorization` header to JumpServer.

```json
{
  "type": "sse",
  "url": "http://127.0.0.1:8099/sse",
  "headers": { "Authorization": "Bearer <JumpServerBearer>" }
}
```

Security tip: Run behind TLS or on a trusted network; server-side Swagger fetch uses `verify=false`.
