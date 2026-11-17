# 02 — Install Caddy on Ubuntu

**Goal:** Install Caddy v2 on Ubuntu and confirm it works.

---

## Theory

Caddy is distributed as a single statically-linked binary — one file, no runtime dependencies. The recommended way to install on Ubuntu is through Caddy's official APT repository, maintained by the Caddy team. This ensures you get security updates automatically via `apt upgrade`.

The APT method adds Caddy's signing key so your system can verify the packages are authentic, then adds the repository, then installs the `caddy` package.

---

## Code

Run these commands one at a time in your terminal:

```bash
# Step 1 — Add required packages
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https curl

# Step 2 — Add Caddy's signing key
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' \
  | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg

# Step 3 — Add the repository
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' \
  | sudo tee /etc/apt/sources.list.d/caddy-stable.list

# Step 4 — Install
sudo apt update
sudo apt install caddy
```

---

## Line-by-line Explanation

| Line | What it does |
|------|--------------|
| `apt install … curl` | Installs tools needed to fetch the key and repo definition |
| `curl … gpg.key \| sudo gpg --dearmor …` | Downloads Caddy's GPG signing key and converts it to binary format; APT uses this to verify packages |
| `curl … debian.deb.txt \| sudo tee …` | Writes the repository address into `/etc/apt/sources.list.d/` so APT knows where to fetch Caddy |
| `sudo apt update` | Refreshes APT's package index to include the new repository |
| `sudo apt install caddy` | Downloads and installs the Caddy binary, its man pages, and the systemd unit file |

---

## Run It

After installation, verify Caddy is on your `PATH`:

```bash
caddy version
```

**Expected output:**

```
v2.9.1 h1:...
```

The hash after `h1:` will differ on your machine — that is normal.

Also check the systemd service was created:

```bash
systemctl status caddy
```

**Expected output (abbreviated):**

```
● caddy.service - Caddy
     Loaded: loaded (/usr/lib/systemd/system/caddy.service; enabled; …)
     Active: active (running) …
```

> **Note:** The APT install automatically starts and enables the caddy service. You can stop it for now with `sudo systemctl stop caddy` while doing local development.

---

## Check It Worked

Open a browser and go to `http://localhost`. You should see the Caddy default welcome page. Alternatively:

```bash
curl -I http://localhost
```

**Expected output:**

```
HTTP/1.1 200 OK
Server: Caddy
...
```

---

## Common Mistakes

1. **Running without `sudo`** — The `gpg` and `tee` commands write to system directories; they need root access.
2. **Skipping `apt update` after adding the repo** — Without the update, APT doesn't know the new package exists and will say `Package 'caddy' has no installation candidate`.
3. **Port 80 already in use** — If another program (like Apache or nginx) is already listening on port 80, Caddy will fail to start. Find the culprit with `sudo ss -tlnp | grep :80` and stop it.

---

## Reference

- Install docs: <https://caddyserver.com/docs/install>

---

## Next

→ [03-static-site.md](03-static-site.md) — Serve your first static website
