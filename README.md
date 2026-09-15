# PAM + Zero Trust Access Lab

A hands-on identity security project demonstrating privileged access management (PAM) and Zero Trust network access principles using HashiCorp Vault, PostgreSQL, and Tailscale.

## Overview

This project simulates a common enterprise security problem: eliminating standing privileged access and implicit network trust. It demonstrates two core identity security capabilities:

1. **Privileged Access Management (PAM)** — dynamic, short-lived credentials instead of static passwords
2. **Zero Trust Network Access (ZTNA)** — identity-based, policy-enforced connectivity instead of network-perimeter trust

## Problem This Solves

Traditional environments rely on:
- Static, long-lived credentials that rarely rotate
- "Trusted" internal networks where access is granted by network location, not identity
- Standing privileged access with no time boundary or approval step

This project addresses each of those with a working implementation.

## Architecture

```
[ Add your architecture diagram here — a simple box diagram showing:
  User -> Tailscale (identity-verified mesh) -> Vault (credential broker) -> Postgres (managed secrets) ]
```

*(Diagram tip: draw this in draw.io, Excalidraw, or even a simple ASCII diagram, and drop the image in an `/assets` folder.)*

## Components

### 1. Privileged Access Management (Vault + PostgreSQL)

- **Tool:** HashiCorp Vault (open source)
- **What it does:** Issues short-lived, auto-expiring database credentials on request instead of static passwords
- **Key capability demonstrated:** Dynamic secrets + automatic rotation
- **Status:** ⬜ Not started / 🟨 In progress / ✅ Complete

**Setup steps:**
```bash
# Install
brew install vault
brew install --cask docker

# Run Vault in dev mode
vault server -dev

# Spin up Postgres
docker run --name pg -e POSTGRES_PASSWORD=yourpassword -p 5432:5432 -d postgres

# Enable the database secrets engine
vault secrets enable database

# [Add your actual config commands here as you complete each step]
```

### 2. Just-in-Time (JIT) Access

- **What it does:** Requires an approval step before privileged credentials are issued, with automatic revocation after a time window
- **Key capability demonstrated:** No standing privilege — access is granted only when needed, only for as long as needed
- **Status:** ⬜ Not started

### 3. Zero Trust Network (Tailscale)

- **Tool:** Tailscale (WireGuard-based mesh VPN)
- **What it does:** Every device authenticates before connecting; access is never granted based on network location alone
- **Key capability demonstrated:** Identity-based access control, least-privilege network policies
- **Status:** ⬜ Not started

**Setup steps:**
```bash
# Install Tailscale on macOS
brew install --cask tailscale

# [Add your ACL policy file and configuration here]
```

## Threat Model

| Threat | How this project addresses it |
|---|---|
| Credential theft (static passwords) | Dynamic, short-lived credentials via Vault |
| Lateral movement after breach | Zero Trust segmentation via Tailscale ACLs |
| Standing privileged access | JIT access with approval + auto-revocation |
| Implicit network trust | Identity-based access, not network-location-based |

## What I Learned

*(Fill this in as you build — this section matters as much as the technical setup. Hiring managers read this part closely.)*

- 
- 
- 

## Tech Stack

- HashiCorp Vault
- PostgreSQL
- Docker / Docker Compose
- Tailscale

## Next Steps / Future Improvements

- [ ] Add Terraform to fully automate infrastructure provisioning
- [ ] Integrate with a SIEM (e.g., Splunk free tier) for access logging and alerting
- [ ] Add a second PAM tool comparison (e.g., CyberArk Conjur) to show vendor-agnostic understanding

## About This Project

Built as a hands-on portfolio project to demonstrate practical identity security skills, including privileged access management and Zero Trust network design principles.

---

*Questions or feedback? Open an issue or reach out on [LinkedIn].*

