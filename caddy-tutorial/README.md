# Caddy Web Server Tutorial

> **Caddy version covered:** v2.9.1  
> **Target OS:** Ubuntu Linux  
> **Audience:** Developers who know basic programming but have never run a web server

Caddy is an open-source web server that handles HTTPS automatically — no manual certificate setup needed.

---

## File Index

| File | What it covers |
|------|----------------|
| [01-basics.md](01-basics.md) | What a web server, reverse proxy, and HTTPS are; why Caddy |
| [02-install.md](02-install.md) | Install Caddy on Ubuntu and verify with `caddy version` |
| [03-static-site.md](03-static-site.md) | Serve static files with `file_server`; Caddyfile structure |
| [04-reverse-proxy.md](04-reverse-proxy.md) | Proxy requests to a FastAPI app running on port 8000 |
| [05-https.md](05-https.md) | How automatic HTTPS works; local vs. real domain |
| [06-multi-site.md](06-multi-site.md) | Serve multiple sites from one Caddyfile |
| [07-extras.md](07-extras.md) | Compression, redirects, logging, and custom headers |
| [08-production.md](08-production.md) | systemd service, `caddy reload`, `caddy validate`, `caddy fmt` |
| [09-troubleshooting.md](09-troubleshooting.md) | Common mistakes and how to fix them |

---

## Examples

Each lesson has a matching folder under `examples/` containing a ready-to-run `Caddyfile` and any supporting files.

```
examples/
├── 03-static-site/   Caddyfile + index.html
├── 04-reverse-proxy/ Caddyfile + app.py
├── 05-https/         Caddyfile (real domain variant)
├── 06-multi-site/    Caddyfile (two-site example)
├── 07-extras/        Caddyfile (compression + redirect + logging)
└── 08-production/    Caddyfile (production-ready)
```

---

## Quick Start (5 minutes)

1. Install Caddy → [02-install.md](02-install.md)
2. Serve a folder → [03-static-site.md](03-static-site.md)
3. Add HTTPS → [05-https.md](05-https.md)

---

## References

- Caddy docs: <https://caddyserver.com/docs/>
- Latest release: <https://github.com/caddyserver/caddy/releases>
- Community forum: <https://caddy.community>
