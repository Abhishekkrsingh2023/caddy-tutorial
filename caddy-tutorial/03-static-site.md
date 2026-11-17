# 03 — Serve a Static Site with `file_server`

**Goal:** Write your first Caddyfile and use it to serve HTML files from a folder.

---

## Theory

A **static site** is a collection of files — HTML, CSS, images — that the server sends to the browser as-is. No code runs on the server when a request arrives; Caddy simply maps the URL path to a file on disk and returns it.

Caddy's configuration file is called a **Caddyfile**. It is designed to be human-readable: minimal punctuation, plain-English directive names. The core idea is a *site block* — you write the **address** you want to serve, then inside curly braces you write **directives** (instructions) that say what to do with requests.

The `file_server` directive turns Caddy into a static file server. By default it serves files from the current directory. Add `browse` to show a directory listing when no `index.html` exists.

---

## Code

**`examples/03-static-site/index.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Hello Caddy</title></head>
<body>
  <h1>Hello from Caddy!</h1>
  <p>This file is served by the <code>file_server</code> directive.</p>
</body>
</html>
```

**`examples/03-static-site/Caddyfile`**

```caddyfile
localhost

file_server
```

---

## Line-by-line Explanation

```
localhost
```
This is the **site address** — the domain (or host) Caddy will listen on. `localhost` means your own machine. Caddy automatically listens on port 443 (HTTPS) with a local certificate, and on port 80 (HTTP) with a redirect to 443.

```
file_server
```
This **directive** tells Caddy to serve static files. With no arguments, it serves files from the directory where you run the command. When a browser requests `/`, Caddy looks for `index.html` in that directory and returns it.

---

## Run It

```bash
# Change into the example directory
cd examples/03-static-site

# Validate the Caddyfile syntax before running
caddy validate --config Caddyfile

# Start Caddy (runs in the foreground; Ctrl+C to stop)
caddy run --config Caddyfile
```

**Expected output from `caddy validate`:**

```
Valid configuration
```

**Expected output from `caddy run`:**

```
{"level":"info","msg":"using provided configuration","config_file":"Caddyfile","config_adapter":"caddyfile"}
{"level":"info","msg":"serving initial configuration"}
```

> **Why `caddy run` and not `caddy start`?**  
> `caddy run` keeps the process in the foreground so you can see logs and stop it with Ctrl+C — ideal for development. `caddy start` backgrounds the process — better for production (covered in [08-production.md](08-production.md)).

---

## Check It Worked

In another terminal:

```bash
curl http://localhost
```

**Expected output:**

```html
<!DOCTYPE html>
<html lang="en">
...
<h1>Hello from Caddy!</h1>
...
```

Or open `https://localhost` in a browser. You will see a certificate warning because the certificate is self-signed (local only). Click **Advanced → Accept the Risk** to proceed. This is normal for `localhost` — covered in [05-https.md](05-https.md).

---

## Caddyfile Structure Quick Reference

```
<site-address>  {
    <directive> [arguments]
    <directive> [arguments]
}
```

- **Site address** can be a domain (`example.com`), IP, or `localhost`.
- **Directives** are single words like `file_server`, `reverse_proxy`, `encode`.
- Braces are optional when there is only one site block and no special options — that is why the example above has no braces.
- Indentation is optional but recommended.

---

## Common Mistakes

1. **Running `caddy run` from the wrong directory** — `file_server` serves the *current working directory*. If you are not in the folder with your HTML files, Caddy will return 404. Use `file_server /absolute/path` to be explicit.
2. **Naming the file something other than `Caddyfile`** — Caddy looks for a file named exactly `Caddyfile` (capital C) when you don't use `--config`. Always pass `--config Caddyfile` explicitly to be safe.
3. **Browser caching old content** — If you edit `index.html` but the browser shows the old version, do a hard refresh: **Ctrl+Shift+R** (Linux/Windows) or **Cmd+Shift+R** (Mac).

---

## Reference

- Getting started: <https://caddyserver.com/docs/getting-started>
- `file_server` directive: <https://caddyserver.com/docs/caddyfile/directives/file_server>
- Caddyfile concepts: <https://caddyserver.com/docs/caddyfile/concepts>

---

## Next

→ [04-reverse-proxy.md](04-reverse-proxy.md) — Proxy traffic to a Python app
