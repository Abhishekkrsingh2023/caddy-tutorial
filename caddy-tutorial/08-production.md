# 08 — Production: systemd, Reload, Validate, and Format

**Goal:** Run Caddy reliably in production using systemd, and update the config without downtime.

---

## Theory

In development you run `caddy run` in a terminal. In production, the server must:

- Start automatically when the machine boots.
- Restart if it crashes.
- Allow config changes without dropping existing connections.

**systemd** is Ubuntu's service manager. It handles all three. When you install Caddy via APT, a systemd unit file is included at `/usr/lib/systemd/system/caddy.service`. This service:

- Runs Caddy as a dedicated `caddy` user (not root) for security.
- Grants permission to bind ports 80 and 443 without being root (via `AmbientCapabilities`).
- Reads config from `/etc/caddy/Caddyfile`.
- Restarts Caddy automatically on failure.

**Zero-downtime reload** means you can change the Caddyfile and apply it *without* restarting Caddy. Active connections finish normally; new connections use the new config. The command is `sudo systemctl reload caddy` (or `caddy reload` if running manually).

---

## Code

**`/etc/caddy/Caddyfile`** (the production config file location)

```caddyfile
# Global options block — applies to the whole server
{
    email admin@example.com
}

example.com {
    encode gzip zstd

    log {
        output file /var/log/caddy/access.log
        format json
    }

    header {
        X-Frame-Options DENY
        X-Content-Type-Options nosniff
        -Server
    }

    reverse_proxy localhost:8000
}
```

**`examples/08-production/Caddyfile`** (same content, for reference)

```caddyfile
{
    email admin@example.com
}

example.com {
    encode gzip zstd

    log {
        output file /var/log/caddy/access.log
        format json
    }

    header {
        X-Frame-Options DENY
        X-Content-Type-Options nosniff
        -Server
    }

    reverse_proxy localhost:8000
}
```

---

## Line-by-line Explanation

```
{
    email admin@example.com
}
```
This is the **global options block** — it has no site address, just a bare `{ }`. Options here apply to the whole server. `email` is the address Let's Encrypt uses to send expiry warnings if automatic renewal fails. **This is the only place in the tutorial that affects certificate issuance.**

```
example.com {
```
Real domain — Caddy fetches a certificate automatically.

The remaining directives (`encode`, `log`, `header`, `reverse_proxy`) are identical to previous lessons.

---

## Run It

### Step 1 — Copy your Caddyfile into place

```bash
sudo cp examples/08-production/Caddyfile /etc/caddy/Caddyfile
```

### Step 2 — Validate before applying

Always validate before reloading in production:

```bash
caddy validate --config /etc/caddy/Caddyfile
```

**Expected output:**

```
Valid configuration
```

### Step 3 — Format the Caddyfile

`caddy fmt` rewrites the Caddyfile with consistent indentation and spacing. Run it before committing to version control:

```bash
caddy fmt --overwrite /etc/caddy/Caddyfile
```

### Step 4 — Enable and start the service

```bash
sudo systemctl enable caddy   # start on boot
sudo systemctl start caddy    # start now
```

### Step 5 — Check status

```bash
sudo systemctl status caddy
```

**Expected output (abbreviated):**

```
● caddy.service - Caddy
     Active: active (running)
```

---

## Updating the Config (Zero-Downtime Reload)

```bash
# 1. Edit the config
sudo nano /etc/caddy/Caddyfile

# 2. Validate
caddy validate --config /etc/caddy/Caddyfile

# 3. Reload (no downtime)
sudo systemctl reload caddy
```

`systemctl reload caddy` sends a signal to Caddy to re-read its config. Caddy applies the new config to new requests while existing connections continue on the old config until they finish.

---

## Useful Commands Reference

| Command | What it does |
|---------|-------------|
| `sudo systemctl start caddy` | Start the service |
| `sudo systemctl stop caddy` | Stop the service |
| `sudo systemctl reload caddy` | Reload config, no downtime |
| `sudo systemctl status caddy` | Show current status and recent logs |
| `journalctl -u caddy -f` | Stream live logs from the service |
| `caddy validate --config /etc/caddy/Caddyfile` | Check for syntax errors |
| `caddy fmt --overwrite /etc/caddy/Caddyfile` | Format the Caddyfile |

---

## A note on JSON config

Caddy also supports a full JSON configuration API (instead of the Caddyfile). JSON config is more verbose but enables dynamic changes via HTTP API without a file reload. See <https://caddyserver.com/docs/json/> if you need programmatic config management.

---

## Common Mistakes

1. **Editing `/etc/caddy/Caddyfile` without validating first** — A broken config + `systemctl reload` will cause Caddy to log an error but *keep the old config running*. Always validate before reloading.
2. **Not setting `email` in the global block** — Without an email, Let's Encrypt can't notify you if renewal fails. Add it before going to production.
3. **Restarting instead of reloading** — `systemctl restart caddy` causes a brief outage (the process exits and restarts). Use `systemctl reload caddy` for zero-downtime config updates.

---

## Reference

- Command line reference: <https://caddyserver.com/docs/command-line>
- Install (systemd unit details): <https://caddyserver.com/docs/install>

---

## Next

→ [09-troubleshooting.md](09-troubleshooting.md) — Common problems and fixes
