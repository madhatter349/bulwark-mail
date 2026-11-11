# bulwark-mail

Bulwark webmail in front of the existing MXroute mailboxes, deployed on the VPS
by Coolify. Plain IMAP/SMTP stays where it is: `legacy-proxy` (the bridge that
ships with Bulwark) turns it into JMAP for the UI. No migration, no MX change.

- Upstream UI: <https://github.com/bulwarkmail/webmail> (AGPL-3.0)
- Upstream bridge: <https://github.com/bulwarkmail/legacy-proxy> (pre-1.0, no security audit)

## Two hostnames

| Service | Host | Purpose |
|---|---|---|
| `webmail` | `mail2.382972.xyz` | The UI |
| `legacy-proxy` | `jmap.382972.xyz` | JMAP API; the browser talks to it directly, so it cannot be internal-only |

## Required environment

| Variable | Why |
|---|---|
| `JMAP_PUBLIC_URL` | Public origin of the proxy, e.g. `https://jmap.382972.xyz` |
| `VAULT_KEY` | base64 32 bytes; AES-256-GCM vault holding each mailbox's IMAP password |
| `SESSION_HMAC_KEY` | base64 32 bytes; signs the proxy's session tokens |
| `BULWARK_SESSION_SECRET` | Signs Bulwark's own session cookies |

```sh
openssl rand -base64 32   # VAULT_KEY, SESSION_HMAC_KEY
openssl rand -hex 32      # BULWARK_SESSION_SECRET
```

Losing `VAULT_KEY` means every connected mailbox must re-authenticate.

## How this project is built and run

| Concern | Where it happens |
|---|---|
| Tests | `scripts/verify.sh` — run by the pre-push hook and nightly on the VPS |
| CI service | **none.** GitHub Actions is deliberately not used |
| Deploy | Coolify on the VPS, triggered by the GitHub push webhook |
| Scheduled work | a systemd timer on the VPS, not a GitHub `schedule:` cron |

## Local

```sh
./scripts/verify.sh     # the gate; must pass before pushing
```
