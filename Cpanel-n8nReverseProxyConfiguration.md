RewriteEngine On

# Forward original scheme/ip to n8n (avoids policy issues)
RequestHeader set X-Forwarded-Proto "https"
RequestHeader set X-Forwarded-SSL "on"
RequestHeader set X-Forwarded-For %{REMOTE_ADDR}s

# --- Timeouts for long-lived WS / SSE ---
ProxyTimeout 3600
Timeout 3600

# --- WebSocket upgrade ---
RewriteCond %{HTTP:Upgrade} =websocket [NC]
RewriteCond %{HTTP:Connection} upgrade [NC]
RewriteRule ^/(.*)$ ws://127.0.0.1:5678/$1 [P,L]

# --- WebSocket endpoint for n8n push events ---
ProxyPass        "/rest/push"  "ws://127.0.0.1:5678/rest/push"
ProxyPassReverse "/rest/push"  "ws://127.0.0.1:5678/rest/push"

# --- Main app/API ---
ProxyPass        "/"  "http://127.0.0.1:5678/" connectiontimeout=5 timeout=3600 keepalive=On retry=0
ProxyPassReverse "/"  "http://127.0.0.1:5678/"

# Improve stream/log delivery & reduce proxy pooling hang issues
SetEnv proxy-sendchunks 1
SetEnv proxy-initial-not-pooled 1
