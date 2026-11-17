# 09 — Troubleshooting

**Goal:** Diagnose and fix the most common Caddy problems.

---

## Theory

Most Caddy problems fall into three categories:

1. **Config errors** — the Caddyfile has a syntax mistake or impossible setting.
2. **Network/system errors** — a port is taken, a firewall is blocking traffic, or file permissions are wrong.
3. **TLS errors** — a certificate couldn't be obtained because DNS isn't set or port 80 is blocked.

The golden rule: **read the logs first.** Caddy's log messages are precise and usually tell you exactly what failed.

---

## How to Read Caddy Logs

**Foreground (development):** logs print to the terminal where you ran `caddy run`.

**systemd (production):**

```bash
# Last 50 lines
journalctl -u caddy -n 50

# Live stream
journalctl -u caddy -f
```

Each log line is JSON (if configured). Key fields:

| Field | Meaning |
|-------|---------|
| `"level":"error"` | Something went wrong |
| `"msg"` | Human-readable description |
| `"error"` | The underlying error message |
| `"identifier"` | The domain involved (for TLS issues) |

---

## Problem 1 — Port Already in Use

**Symptom:**

```
Error: listen tcp :80: bind: address already in use
```

**Cause:** Another process (Apache, nginx, or another Caddy instance) is already listening on port 80 or 443.

**Fix:**

```bash
# Find which process is using port 80
sudo ss -tlnp | grep :80

# Example output:
# tcp LISTEN 0 128 0.0.0.0:80 0.0.0.0:* users:(("apache2",pid=1234,...))

# Stop the conflicting service
sudo systemctl stop apache2

# Then start Caddy
sudo systemctl start caddy
```

---

## Problem 2 — `caddy validate` Fails

**Symptom:**

```
Error: adapting config using caddyfile: ...
```

**Cause:** A typo or structural error in your Caddyfile.

**Fix:** Run `caddy validate --config Caddyfile` and read the error. Common mistakes:

- Missing closing `}` brace.
- Directive name misspelled (e.g. `fileserver` instead of `file_server`).
- Using a directive that doesn't exist in your Caddy version.

```bash
# Always validate before reloading
caddy validate --config /etc/caddy/Caddyfile
```

If you can't spot the issue, run `caddy fmt --overwrite Caddyfile` — formatting sometimes reveals structural problems.

---

## Problem 3 — 502 Bad Gateway

**Symptom:** Browser shows "502 Bad Gateway".

**Cause:** Caddy can't reach the backend (the app on port 8000 is not running or crashed).

**Fix:**

```bash
# Check if the backend is running
curl http://localhost:8000/

# If not, restart it
cd /path/to/app
uvicorn app:app --host 127.0.0.1 --port 8000
```

---

## Problem 4 — Certificate Not Obtained (TLS Error)

**Symptom in logs:**

```
{"level":"error","msg":"obtaining certificate","error":"...ACME server..."}
```

**Causes and fixes:**

| Cause | Fix |
|-------|-----|
| DNS not pointing to this server | Update the A record in your DNS provider's dashboard; wait for propagation (`dig example.com`) |
| Port 80 blocked | Open port 80 in your firewall/security group — it is required for the ACME HTTP-01 challenge |
| Rate-limited by Let's Encrypt | Let's Encrypt allows 5 certificates per domain per week. Wait and retry, or use the Let's Encrypt staging server for testing |

**Check DNS propagation:**

```bash
dig +short example.com
# Should return your server's public IP
```

**Check port 80 is reachable from outside:**

```bash
# From a different machine:
curl http://example.com
```

---

## Problem 5 — Permission Denied (Log File or Port)

**Symptom:**

```
Error: opening log file: open /var/log/caddy/access.log: permission denied
```

or

```
Error: listen tcp :443: bind: permission denied
```

**Fix for log file:**

```bash
sudo mkdir -p /var/log/caddy
sudo chown caddy:caddy /var/log/caddy
```

**Fix for port binding:** Use the systemd service (which grants `CAP_NET_BIND_SERVICE`) instead of running `caddy run` as a regular user.

---

## Problem 6 — `caddy reload` Has No Effect

**Symptom:** You edited `/etc/caddy/Caddyfile` and reloaded, but the behaviour didn't change.

**Cause:** You may have edited the wrong file (not the one Caddy is actually using).

**Fix:**

```bash
# Check which config file Caddy loaded at startup
journalctl -u caddy | grep "config"
# Look for: "config_file":"/etc/caddy/Caddyfile"

# Confirm you saved the right file
caddy validate --config /etc/caddy/Caddyfile
sudo systemctl reload caddy
```

---

## Problem 7 — Browser Warns About Certificate on `localhost`

**Symptom:** "Your Connection Is Not Private" on `https://localhost`.

**Fix:**

```bash
caddy trust   # requires root; restart browser after
```

---

## Quick Checklist

- [ ] `caddy validate --config /path/to/Caddyfile` → "Valid configuration"?
- [ ] `journalctl -u caddy -n 50` → any error lines?
- [ ] `sudo ss -tlnp | grep -E ':80|:443'` → ports available?
- [ ] `dig +short yourdomain.com` → returns your server's IP?
- [ ] `curl http://localhost:8000` → backend responding?

Still stuck? Search <https://caddy.community> — most problems are already solved there.

---

## Reference

- Command line: <https://caddyserver.com/docs/command-line>
- Automatic HTTPS troubleshooting: <https://caddyserver.com/docs/automatic-https>
- Community help: <https://caddy.community>
