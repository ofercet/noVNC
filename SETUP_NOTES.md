# noVNC Remote VNC Server Setup Notes

## Goal
Connect to a remote VNC server (running websockify with WebSocket support on port 443 over HTTPS) using noVNC client in GitHub Codespaces.

## Remote VNC Server Details
- **URL**: https://fictional-space-invention-695wv5xwvwvfwgv-6080.app.github.dev
- **Port**: 443 (HTTPS/WSS)
- **Protocol**: WebSocket (WSS - Secure WebSocket)

## Local noVNC Proxy Setup

### Prerequisites
- noVNC repository cloned and ready
- OpenSSL available for certificate generation

### Steps

#### 1. Generate SSL Certificate
```bash
cd /workspaces/noVNC
openssl req -new -x509 -days 365 -nodes -out self.pem -keyout self.pem -subj "/C=US/ST=State/L=City/O=Org/CN=localhost"
```

**Purpose**: novnc_proxy needs an SSL certificate to serve HTTPS and use WSS (Secure WebSocket) protocol.

#### 2. Start novnc_proxy
```bash
./utils/novnc_proxy --vnc fictional-space-invention-695wv5xwvwvfwgv-6080.app.github.dev:443 --listen 6081 --cert self.pem
```

**Parameters**:
- `--vnc`: Remote VNC server address and port (HTTPS/WSS on port 443)
- `--listen`: Local port for the proxy (6081)
- `--cert`: SSL certificate file for secure connections

#### 3. Important Notes
- The proxy will show: `SSL/TLS support` in startup logs
- Do NOT use `--ssl-only` flag (it breaks the setup)
- Ignore warnings about missing `numpy` module - they don't affect functionality

### Accessing the VNC Client

#### Local Testing (Development)
```
http://localhost:6081/vnc.html
```

#### Remote Access via Codespaces Tunnel (Production)
```
https://silver-garbanzo-v64vgpgpwj3p7gv-6081.app.github.dev/vnc.html?host=fictional-space-invention-695wv5xwvwvfwgv-6080.app.github.dev&port=443&encrypt=1
```

**Note**: Must use HTTPS URL (not HTTP) to force wss:// WebSocket connection.

### URL Parameters Explained
- `host=fictional-space-invention-695wv5xwvwvfwgv-6080.app.github.dev` - Remote websockify server (lowercase, no protocol prefix)
- `port=443` - Remote port
- `encrypt=1` - Use secure connection (wss://)

### Expected Behavior

#### In Terminal Logs
- **Good**: `Secure WebSocket (wss://) WebSocket connection`
- **Bad**: `Plain non-SSL (ws://) WebSocket connection`

To get WSS, always use the HTTPS URL (not HTTP).

### Troubleshooting

#### Issue: `ws://` instead of `wss://`
**Cause**: Using HTTP URL instead of HTTPS
**Solution**: Use the full HTTPS Codespaces tunnel URL

#### Issue: `Port 6081 in use`
**Solution**: 
```bash
fuser -k 6081/tcp
# Then restart novnc_proxy
```

#### Issue: `Could not find self.pem`
**Solution**: Generate certificate as shown in step 1

#### Issue: Connection refused to remote server
**Cause**: Remote websockify server not running or not accessible
**Solution**: Verify remote server is online and port 443 is open

## Quick Start (Repeat Setup)

```bash
cd /workspaces/noVNC

# Generate cert (if not already done)
openssl req -new -x509 -days 365 -nodes -out self.pem -keyout self.pem -subj "/C=US/ST=State/L=City/O=Org/CN=localhost"

# Kill any existing process
pkill -f novnc_proxy

# Start proxy
./utils/novnc_proxy --vnc fictional-space-invention-695wv5xwvwvfwgv-6080.app.github.dev:443 --listen 6081 --cert self.pem
```

Then access via HTTPS URL above.

## Key Takeaways
1. Always use HTTPS URL to force wss:// protocol
2. The proxy must have an SSL certificate
3. Remote host parameter must be lowercase without protocol prefix
4. Port 443 is used for secure connection to remote websockify
5. Local proxy listens on port 6081

---
*Setup completed: February 4, 2026*

