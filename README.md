# PAM + Zero Trust Access Lab

A hands-on identity security project demonstrating privileged access management (PAM) and Zero Trust network access principles using HashiCorp Vault, PostgresApp, and Tailscale.

## Overview
This project simulates enterprise security challenges by eliminating standing privileged access and implicit network trust through native application layers.

## Architecture
- **Privileged Access Management (Vault + PostgreSQL):** Issues short-lived, auto-expiring database credentials via HashiCorp Vault and PostgresApp.
- **Just-in-Time & Policy Access:** Enforces read-only capability mapping with automated TTL expiration.
- **Zero Trust Network (Tailscale):** Implements WireGuard-based mesh overlay with strict endpoint authentication and identity-driven access controls (`autogroup:member`).

### Infrastructure Status Validation
![Vault Operational Matrix](vault-status.png)

### Environment Startup Details
![Vault Startup Overview](vault-startup.png)

### Tailscale Secure Mesh Verification
![Tailscale Network Mesh Fabric](tailscale-mesh.png)

### Native PostgreSQL Backend Verification
![PostgresApp Running Instance](postgres-status.png)

## Setup

### Prerequisites
- macOS (Intel or Apple Silicon)
- Homebrew
- Tailscale account (free tier)

### 1. Install Vault and PostgresApp
```bash
brew install vault
# Install PostgresApp from https://postgresapp.com
```

### 2. Start Vault in dev mode
```bash
vault server -dev
```
Save the root token printed in the output — you'll need it to authenticate.

### 3. Configure the database secrets engine
```bash
export VAULT_ADDR='http://127.0.0.1:8200'
vault secrets enable database

vault write database/config/postgresql \
  plugin_name=postgresql-database-plugin \
  connection_url="postgresql://{{username}}:{{password}}@localhost:5432/postgres?sslmode=disable" \
  allowed_roles="readonly-role" \
  username="vaultadmin" \
  password="[your admin password]"

vault write database/roles/readonly-role \
  db_name=postgresql \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"
```

### 4. Generate a short-lived credential
```bash
vault read database/creds/readonly-role
```
This returns a username/password valid for 1 hour, then automatically revoked.

### 5. Install and configure Tailscale
```bash
brew install --cask tailscale
```
Sign in through the Tailscale app, then set up an ACL policy in the Tailscale admin console (https://login.tailscale.com/admin/acls) restricting access using `autogroup:member` so only authenticated devices on your tailnet can reach lab resources.

## Threat Model Mitigation Matrix

| Threat | Mitigation |
|---|---|
| Credential Theft | Dynamic, short-lived tokens via Vault |
| Lateral Session Movement | Tailscale zero-trust segmentation policies |
| Standing Administrative Privileges | 1-hour TTL expiration windows |

## What I Learned

- **Why short-lived credentials matter in practice, not just theory.** 
Watching a Vault-issued credential actually expire and get rejected made the "eliminate standing privilege" concept concrete in a way reading about it never did.
* I set database credentials to expire after 1 hour because it's a good middle ground: short enough that a stolen credential is only useful for a short window, but long enough that it doesn't get in the way of getting work done. In a Zero Trust setup, this kind of tradeoff comes up a lot — the more often credentials rotate, the more secure things are, but you also add more overhead (more requests to Vault, more chances for something to break mid-task). One hour felt like a reasonable balance for a lab environment like this.

1. Ran out of disk space trying to run Postgres in Docker
My Mac (an older Intel model running macOS Sequoia) only had about 6 GB of free disk space. When I tried running Postgres in a Docker container like the original guide suggested, Docker's virtualization threw an I/O error and froze the terminal. I tried compiling Postgres a different way through Homebrew instead, but that required a 23 GB Xcode update my drive couldn't fit.
To fix it, I force-killed the stuck Docker processes (pkill -9 -f Docker), cleared out Homebrew's cache files (rm -rf ~/Library/Caches/Homebrew/*), and freed up the drive back to about 4.4 GB. Then I switched approaches entirely — instead of Docker or compiling from source, I used PostgresApp, a lightweight pre-built Mac app version of Postgres. It uses almost no memory, didn't need Xcode at all, and had the database running on port 5432 within minutes.
2. Vault wouldn't install through Homebrew, so I downloaded it manually
Since my Homebrew install was broken (from the earlier Intel Mac issue), running Vault commands failed with command not found. When I tried copy-pasting a terminal command to download Vault directly, it accidentally grabbed a corrupted webpage instead of the actual program file, which threw an extraction error.
So I downloaded Vault directly through my browser instead, then verified the file wasn't corrupted or tampered with by checking its checksum — basically a fingerprint — with shasum -a 256 and comparing it against the official one HashiCorp publishes. Once I confirmed it matched, I had to remove a security flag macOS puts on downloaded files (xattr -d com.apple.quarantine) so it would actually run, and I ran it directly from the folder I downloaded it to.
3. Tailscale rejected my access policy until I used the right kind of identifier
When setting up network access rules, I tried referencing my computer by its nickname ("macbook-pro"), but Tailscale's policy system rejected it with an "invalid address" error.
Turns out Tailscale's access rules don't recognize computer nicknames — they need an actual verified account or a specific tag instead. I switched to using autogroup:member, which is a built-in Tailscale tag that automatically covers every device signed into your account (in my case, my Mac and iPhone). That fixed the syntax error and let the policy save correctly.

* What I built vs. what a real company would run:
Vault: I ran a single, temporary Vault server that stores everything in memory — closing my terminal wipes it out completely. A real company runs Vault as a cluster spread across multiple data centers, saves secrets to durable storage instead of memory, and uses cloud key management to unlock itself automatically instead of someone typing in a master key by hand.
Database: I used a single local Postgres instance with full admin rights. A real company uses managed, multi-server database clusters, and Vault connects with a limited-permission service account — not a root login — that can only create and delete temporary accounts, not touch anything else. At scale, this system is generating and deleting thousands of temporary credentials per minute as traffic goes up and down.
Network access (Tailscale): I logged in with my personal email and grouped my own devices together. A real company connects this to their actual employee login system (like Okta or Azure AD), so access is based on which team you're on, not a personal account. They also check that your laptop is updated, has antivirus running, and has its firewall on before letting it connect at all — something I skipped in the lab version.
Getting credentials: I manually typed a command to get a temporary password. In production, this is fully automated — an application starts up, silently asks Vault for a password using its own verified identity, uses it, and no human ever sees it.

## Tech Stack
- HashiCorp Vault
- PostgreSQL Engine (PostgresApp)
- Tailscale (WireGuard)
