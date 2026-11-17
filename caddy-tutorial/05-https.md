# 05 — Automatic HTTPS

**Goal:** Understand how Caddy obtains and renews TLS certificates automatically, and know the difference between local and real-domain HTTPS.

---

## Theory

**TLS** (Transport Layer Security) is the encryption protocol behind HTTPS. To use it, your server needs a **certificate** — a file signed by a trusted **Certificate Authority (CA)** that proves you control the domain. Browsers reject certificates they don't trust.

There are two scenarios:

| Scenario | Certificate type | Who trusts it |
|----------|-----------------|---------------|
| `localhost` / local dev | Self-signed, issued by Caddy's local CA | Only your machine (after one-time trust) |
| Real domain (e.g. `example.com`) | Signed by Let's Encrypt | All browsers, automatically |

### How Caddy gets a real certificate

Caddy uses the **ACME protocol** (Automatic Certificate Management Environment) to talk to Let's Encrypt:

1. You set your domain's DNS to point to your server's IP.
2. Caddy notices the site address is a public domain (not `localhost` or an IP).
3. Caddy asks Let's Encrypt: "Please give me a certificate for `example.com`."
4. Let's Encrypt replies: "Prove you control it." — Caddy responds to this **challenge** automatically (via HTTP-01: it briefly serves a secret file on port 80).
5. Let's Encrypt issues the certificate. Caddy stores it and serves HTTPS immediately.
6. Caddy renews the certificate automatically before it expires (every ~60 days).

**You do nothing.** The whole process is triggered by the domain name in your Caddyfile.

### Local development with `localhost`

When the site address is `localhost`, Caddy uses its built-in local CA instead. The first time you run Caddy, it creates a local CA and installs its root certificate into your system's trust store (requires root/sudo). After that, `https://localhost` shows a green padlock in the browser.

---

## Code

### Local development (use this for testing on your machine)

```caddyfile
localhost {
    file_server
}
```

Caddy handles HTTPS automatically. No extra config needed.

### Real domain (use this when you have a public server and DNS pointing to it)

```caddyfile
example.com {
    file_server
}
```

Replace `example.com` with your actual domain. That is the **only** change needed. Caddy will:
- Obtain a certificate from Let's Encrypt.
- Serve HTTPS on port 443.
- Redirect HTTP (port 80) to HTTPS automatically.

---

## Line-by-line Explanation

```
example.com {
```
Caddy sees a public domain name and activates automatic HTTPS. The certificate is requested on first startup.

```
    file_server
```
Serves static files, same as lesson 03.

```
}
```
Closes the block.

There is no `tls` directive here — none is needed. Automatic HTTPS is Caddy's default behaviour for any named domain.

---

## Run It

For the real-domain example, you need:
1. A registered domain name.
2. An A/AAAA DNS record pointing to your server's public IP.
3. Ports 80 and 443 open in your firewall.

```bash
# On the server, in the directory with your files:
caddy validate --config Caddyfile
sudo caddy run --config Caddyfile
```

> **Why `sudo` here?** Binding to ports 80 and 443 requires root on Linux. The systemd service (covered in [08-production.md](08-production.md)) handles this more cleanly.

**Expected log output (first run with real domain):**

```
{"level":"info","msg":"obtaining certificate","server":"acme-v02.api.letsencrypt.org/directory","identifier":"example.com"}
{"level":"info","msg":"certificate obtained successfully","identifier":"example.com"}
{"level":"info","msg":"serving initial configuration"}
```

---

## Check It Worked

```bash
curl -I https://example.com
```

**Expected output:**

```
HTTP/2 200
server: Caddy
...
```

Check the certificate details:

```bash
curl -v https://example.com 2>&1 | grep "subject:"
```

**Expected output:**

```
*  subject: CN=example.com
```

---

## What Caddy Will NOT Do Automatically

- Obtain a certificate for a bare IP address (e.g. `1.2.3.4`) — not covered in the docs.
- Obtain a certificate if port 80 is blocked by your firewall — the ACME HTTP-01 challenge will fail.
- Obtain a certificate for `localhost` from a public CA — it uses its own local CA instead.

---

## Common Mistakes

1. **DNS not propagated yet** — If you just set the DNS record, it may take minutes to hours to propagate. Caddy will log an error and retry, but it won't serve HTTPS until the challenge succeeds.
2. **Port 80 blocked** — Let's Encrypt's HTTP-01 challenge requires port 80 to be reachable. Check your cloud provider's firewall/security group settings.
3. **Running as non-root on a production server** — Ports below 1024 require root on Linux. Use the systemd service (which grants this via `AmbientCapabilities`) instead of running `caddy run` as a regular user.

---

## Reference

- Automatic HTTPS: <https://caddyserver.com/docs/automatic-https>

---

## Next

→ [06-multi-site.md](06-multi-site.md) — Serve multiple sites from one Caddyfile
