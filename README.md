# CFDE Eval Protector

Reverse proxy that gates access to [cfde-eval.netlify.app](https://cfde-eval.netlify.app/) behind ORCID authentication using Traefik and oauth2-proxy.

Only users whose ORCID iD appears in `allowed-orcids.txt` can access the site.

## Architecture

```
Browser → Traefik (TLS) → oauth2-proxy (ORCID OIDC) → cfde-eval.netlify.app
```

- **Traefik** terminates TLS and routes all traffic to oauth2-proxy
- **oauth2-proxy** handles the ORCID OAuth2/OIDC flow and proxies authenticated requests upstream

## Setup

### 1. Register an ORCID OAuth application

1. Sign in to your ORCID account at https://orcid.org
2. Go to https://orcid.org/developer-tools
3. If this is your first app, click **Register for the free ORCID public API**
4. Fill out the application form:
   - **Application name**: e.g. "CFDE Eval Protector"
   - **Application URL**: `https://127.0.0.1`
   - **Description**: Brief description of the application
   - **Redirect URIs**: `https://127.0.0.1/oauth2/callback`
5. Click the save icon to register the app
6. Note the **Client ID** (e.g. `APP-XXXXXXXXXXXX`) and **Client Secret** shown on the page

### 2. Configure environment

```bash
cp .env.example .env
```

Fill in your ORCID credentials and generate a cookie secret:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(32))"
```

### 3. Add allowed ORCID iDs

Add one ORCID iD per line to `allowed-orcids.txt`:

```
0000-0002-8991-6458
```

### 4. Run

```bash
docker compose up -d
```

Visit https://127.0.0.1 and accept the self-signed certificate warning. You'll be redirected to ORCID to sign in.

## Files

| File | Purpose |
|---|---|
| `docker-compose.yml` | Traefik, oauth2-proxy, and cert-gen services |
| `traefik.yml` | Traefik static configuration |
| `dynamic/routes.yml` | Traefik routing and TLS config |
| `allowed-orcids.txt` | Allowlist of ORCID iDs (one per line) |
| `.env` | Secrets (not committed) |
