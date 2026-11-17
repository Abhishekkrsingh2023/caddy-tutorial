# 07 — Extras: Compression, Redirects, Logging, and Headers

**Goal:** Add compression, URL redirects, access logging, and security headers to your Caddyfile.

---

## Theory

A production web server does more than serve files or proxy requests. It should:

- **Compress** responses so they download faster.
- **Redirect** old URLs to new ones (e.g. `/blog` → `/posts`).
- **Log** every request so you can debug problems and spot attacks.
- **Set security headers** to protect browsers from common vulnerabilities.

Caddy provides simple directives for all of these. They are composable — you add them to any site block alongside `file_server` or `reverse_proxy`.

---

## Code

**`examples/07-extras/Caddyfile`**

```caddyfile
localhost {
    # 1. Compress responses with gzip and zstd
    encode gzip zstd

    # 2. Redirect /old-page to /new-page (301 = permanent)
    redir /old-page /new-page 301

    # 3. Access log — written to a file
    log {
        output file /var/log/caddy/access.log
        format json
    }

    # 4. Security headers
    header {
        X-Frame-Options DENY
        X-Content-Type-Options nosniff
        Referrer-Policy strict-origin-when-cross-origin
        # Remove the Server header to avoid leaking Caddy's version
        -Server
    }

    file_server
}
```

---

## Line-by-line Explanation

### `encode gzip zstd`

`encode` is the compression directive. It compresses HTTP response bodies before sending them. `gzip` is universally supported; `zstd` is faster and supported by modern browsers. Caddy picks the best algorithm based on the browser's `Accept-Encoding` request header. HTML, CSS, and JSON compress well (often 60–80% smaller).

### `redir /old-page /new-page 301`

`redir` sends an HTTP redirect. The browser that requests `/old-page` receives a `301 Moved Permanently` response with `Location: /new-page`. The browser then requests the new URL automatically. Use `302` for temporary redirects.

### `log { ... }`

The `log` directive configures access logging.

| Sub-directive | Meaning |
|---------------|---------|
| `output file /var/log/caddy/access.log` | Write logs to this file (creates it if absent) |
| `format json` | One JSON object per line — easy to parse with tools like `jq` |

Without `log`, Caddy logs to stderr (the terminal). In production you want file logging.

### `header { ... }`

The `header` directive adds, modifies, or removes HTTP response headers.

| Header | What it does |
|--------|-------------|
| `X-Frame-Options DENY` | Prevents your pages from being embedded in an `<iframe>` on another site (blocks clickjacking) |
| `X-Content-Type-Options nosniff` | Tells the browser not to guess the file type — prevents MIME-type confusion attacks |
| `Referrer-Policy strict-origin-when-cross-origin` | Controls how much URL information is sent when the browser follows a link to another site |
| `-Server` | The leading `-` **removes** the header. Removes `Server: Caddy` to avoid advertising your software version |

---

## Run It

```bash
# Create the log directory
sudo mkdir -p /var/log/caddy
sudo chown $USER /var/log/caddy

cd examples/07-extras
caddy validate --config Caddyfile
caddy run --config Caddyfile
```

**Expected output from validate:**

```
Valid configuration
```

---

## Check It Worked

**Compression:**

```bash
curl -H "Accept-Encoding: gzip" -I https://localhost/
```

**Expected response header:**

```
Content-Encoding: gzip
```

**Redirect:**

```bash
curl -I https://localhost/old-page
```

**Expected:**

```
HTTP/2 301
location: /new-page
```

**Security headers:**

```bash
curl -I https://localhost/ | grep -E "X-Frame|X-Content|Referrer"
```

**Expected:**

```
x-frame-options: DENY
x-content-type-options: nosniff
referrer-policy: strict-origin-when-cross-origin
```

**Log file:**

```bash
tail -1 /var/log/caddy/access.log | jq .
```

**Expected:** A JSON object with fields like `request`, `status`, `duration`, `bytes_sent`.

---

## Common Mistakes

1. **Log directory doesn't exist** — Caddy will fail to start if the log path's parent directory is missing. Always `mkdir -p` the log directory first.
2. **`encode` placed after `file_server`** — Directive order in Caddy is handled internally by an ordering system, but it is clearest to put `encode` before `file_server` or `reverse_proxy`.
3. **Redirect loop** — If you write `redir /new-page /new-page 301`, the browser will follow the redirect forever and show an error. Always redirect to a *different* path.

---

## Reference

- `encode` directive: <https://caddyserver.com/docs/caddyfile/directives/encode>
- `redir` directive: <https://caddyserver.com/docs/caddyfile/directives/redir>
- `log` directive: <https://caddyserver.com/docs/caddyfile/directives/log>
- `header` directive: <https://caddyserver.com/docs/caddyfile/directives/header>

---

## Next

→ [08-production.md](08-production.md) — Run Caddy as a systemd service
