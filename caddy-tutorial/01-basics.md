# 01 — Concepts: Web Servers, Reverse Proxies, and HTTPS

**Goal:** Understand the three ideas that every Caddy tutorial builds on.

---

## Theory

### The problem: how does a browser get a webpage?

When you type `http://example.com` in a browser, your computer sends a **request** — a message asking for a resource — across the internet. Something on the other end must receive that request and send back the content. That "something" is a **web server** (also called an HTTP server).

A web server is just a program that:
1. Listens on a **port** — a numbered slot on your machine (port 80 for HTTP, 443 for HTTPS).
2. Reads incoming requests.
3. Sends back a **response** — the HTML, image, or data the browser asked for.

### Static vs. dynamic content

A **static site** serves files exactly as they are on disk: HTML, CSS, images. No code runs on the server per request.

A **dynamic application** (like a FastAPI or Django app) runs code to generate the response. Those apps usually listen on a private port like 8000 and are not safe to expose directly to the internet — they lack TLS, rate limiting, and other protections.

The solution is a **reverse proxy**: a web server that sits in front of your app, receives public requests, and forwards them to your app's private port. The browser talks to the proxy; the proxy talks to your app.

```
Browser ──► Caddy (port 443) ──► Your app (port 8000)
```

### HTTPS and TLS

**HTTPS** is HTTP with encryption. The encryption layer is called **TLS** (Transport Layer Security). TLS requires a **certificate** — a small file that proves your domain belongs to you. Certificates are issued by a trusted authority like **Let's Encrypt** (free).

Historically, getting and renewing certificates was tedious. Caddy automates the entire process: it requests the certificate, proves domain ownership automatically, and renews before expiry — all without manual steps.

### Why Caddy?

| Feature | Caddy | nginx / Apache |
|---------|-------|----------------|
| Automatic HTTPS | ✅ built-in | ❌ manual setup |
| Config syntax | Simple Caddyfile | Complex, verbose |
| Reload without downtime | `caddy reload` | varies |
| HTTP/2 and HTTP/3 | ✅ default | partial |

Caddy's defaults are secure. If you specify a real domain name in your config, Caddy obtains and renews its certificate automatically — no plugins, no cron jobs.

---

## Common Mistakes

1. **Confusing port 80 and port 443** — Port 80 is plain HTTP; port 443 is HTTPS. Caddy uses both: 80 to redirect to 443, and 443 for secure traffic.
2. **Thinking HTTPS hides the URL** — HTTPS encrypts the *content* of the request, not the domain name (though Caddy v2.10+ adds ECH support for that). It is not the same as a VPN.
3. **Running your app on port 80 directly** — Apps like FastAPI were not designed to handle raw internet traffic. Always put a web server in front.

---

## Reference

- Caddy docs home: <https://caddyserver.com/docs/>
- Automatic HTTPS overview: <https://caddyserver.com/docs/automatic-https>

---

## Next

→ [02-install.md](02-install.md) — Install Caddy on Ubuntu
