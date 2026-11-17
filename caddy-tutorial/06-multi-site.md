# 06 — Multiple Sites in One Caddyfile

**Goal:** Serve two different sites — a static site and a proxied app — from a single Caddyfile.

---

## Theory

As your infrastructure grows, you will likely want one server to handle several domains: perhaps a marketing site at `example.com` and an API at `api.example.com`. Running a separate Caddy process for each would waste resources and complicate management.

Caddy supports **multiple site blocks** in one Caddyfile. Each block is independent — it has its own address, its own directives, and (if using real domains) its own TLS certificate. Caddy fetches all certificates in parallel on startup.

Each site block matches requests by the **Host** header the browser sends. Caddy routes each request to the matching block.

---

## Code

**`examples/06-multi-site/Caddyfile`**

```caddyfile
# Site 1 — static marketing site
localhost:8080 {
    root * /var/www/site1
    file_server
}

# Site 2 — proxied API app
localhost:8443 {
    reverse_proxy localhost:8000
}
```

> This example uses different ports on `localhost` so you can test it without a real domain. In production, replace with real domain names (see below).

### Production version (real domains, one Caddyfile)

```caddyfile
example.com {
    root * /var/www/example
    file_server
}

api.example.com {
    reverse_proxy localhost:8000
}
```

---

## Line-by-line Explanation

```
localhost:8080 {
```
A site block that listens on `localhost` port `8080`. Specifying a custom port disables automatic HTTPS (Caddy only auto-enables HTTPS for standard ports 80/443 and named domains without explicit ports).

```
    root * /var/www/site1
```
The `root` directive sets the **document root** — the base directory on disk that file paths are resolved from. The `*` is a **path matcher** meaning "match all paths." Without `root`, `file_server` defaults to the current working directory.

```
    file_server
```
Serves files from the document root set above.

```
localhost:8443 {
```
A second, independent site block on a different port.

```
    reverse_proxy localhost:8000
```
Forwards all requests to the backend app on port 8000.

---

## Run It

```bash
# Create dummy site1 directory
sudo mkdir -p /var/www/site1
echo "<h1>Site 1</h1>" | sudo tee /var/www/site1/index.html

# Start a fake backend for site 2
python3 -m http.server 8000 &

cd examples/06-multi-site
caddy validate --config Caddyfile
caddy run --config Caddyfile
```

**Expected output from validate:**

```
Valid configuration
```

---

## Check It Worked

```bash
# Site 1 — static
curl http://localhost:8080
```

**Expected:**
```html
<h1>Site 1</h1>
```

```bash
# Site 2 — proxied
curl http://localhost:8443
```

**Expected:** Response from whatever `localhost:8000` serves.

---

## Sharing Config with Snippets

If two sites share the same directives (e.g. the same headers), you can use a **snippet** to avoid repetition:

```caddyfile
(common_headers) {
    header X-Frame-Options DENY
    header X-Content-Type-Options nosniff
}

example.com {
    import common_headers
    file_server
}

api.example.com {
    import common_headers
    reverse_proxy localhost:8000
}
```

Snippets are defined with `(name) { ... }` and imported with `import name`. They are covered in the [Caddyfile concepts docs](https://caddyserver.com/docs/caddyfile/concepts).

---

## Common Mistakes

1. **Two blocks with the same address** — Caddy will reject this with a config error. Each site block must have a unique address.
2. **Forgetting `root` when using `file_server` with multiple sites** — Without `root`, both sites would serve from the same working directory. Always set `root` explicitly when you have more than one site.
3. **Mixing ports and domains carelessly** — `localhost:8080` and `localhost` are different addresses. If you want HTTPS, use a real domain name or `localhost` (without a port number).

---

## Reference

- Caddyfile concepts: <https://caddyserver.com/docs/caddyfile/concepts>
- `root` directive: <https://caddyserver.com/docs/caddyfile/directives/root>

---

## Next

→ [07-extras.md](07-extras.md) — Compression, redirects, logging, and headers
