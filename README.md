# Enterprise Security Lab

A self-hosted homelab built on Proxmox to practice enterprise security engineering: network segmentation, centralized identity, single sign-on, MFA, reverse-proxied ingress, and SIEM monitoring. Every service is documented as a build note or a troubleshooting note in the folders below.

## Architecture

- **Firewall / router:** OPNsense, six VLANs, per-VLAN rulesets with a default deny at the bottom of each.
- **Identity:** Keycloak (v26, Postgres backend) federated to Active Directory over LDAPS, with an Enterprise CA (AD CS) issuing the DC certificate.
- **SSO / MFA:** Keycloak is the single identity source. Services with native OIDC/SAML talk to it directly; services without it sit behind oauth2-proxy. TOTP enforced once at the realm level so every SSO service inherits it.
- **Ingress:** Nginx Proxy Manager is the only entry point. Clients reach services by domain name, never by IP and port, so SSO and MFA cannot be bypassed by hitting the backend directly.
- **DNS:** Pi-hole as local resolver with split-horizon records pointing every service domain at NPM, plus a conditional forward to AD for `keenan.sec`.
- **Monitoring:** Wazuh SIEM, with critical alerts intended to open tickets in osTicket.
- **Remote access:** Tailscale on the Proxmox nodes.
- **Secrets:** Vaultwarden for API keys, VM credentials, and database passwords.

Full design, service inventory, and the 2FA plan are in [Overview/Overview.md](Overview/Overview.md).

## VLAN Layout

| VLAN | Purpose  | Example services            |
| ---- | -------- | --------------------------- |
| 0    | WAN/LAN  | OPNsense                    |
| 10   | Ingress  | Nginx Proxy Manager         |
| 20   | Services | WikiJS, n8n, EspoCRM, Immich, Linkwarden |
| 30   | Identity | Keycloak, AD/DC, Pi-hole, Vaultwarden |
| 40   | Security | Wazuh, osTicket, theHive    |
| 60   | Home     | Client devices              |
| 99   | Mgmt     | Proxmox nodes               |

Firewall rules per VLAN: [Overview/Firewall Rules.md](Overview/Firewall%20Rules.md).

## Documentation Index

### Overview
- [Architecture, SSO, and 2FA plan](Overview/Overview.md)
- [Firewall Rules (per VLAN)](Overview/Firewall%20Rules.md)
- [Creating Aliases](Overview/Creating%20Aliases.md)
- [DNS Not Resolving Upstream](Overview/DNS%20Not%20Resolving%20Upstream.md)
- [Wrong CIDR Breaks Connectivity](Overview/Wrong%20CIDR%20Breaks%20Connectivity.md)

### OPNsense
- [Creating a VLAN and Subnet](OPNsense/Creating%20a%20VLAN%20%26%20Subnet.md)
- [Configuring Firewall Rules](OPNsense/Configuring%20Firewall%20Rules.md)

### Active Directory
- [Setup](Active%20Directory/Setup.md)
- [Fixing the NIC so the install could proceed](Active%20Directory/Fixing%20the%20NIC%20so%20the%20install%20could%20proceed.md)
- [Pi-hole + AD DNS Resolution](Active%20Directory/Pi-hole%20%2B%20AD%20DNS%20Resolution.md)

### Keycloak
- [Install (v26 + Postgres)](Keycloak/Keycloak%20Install.md)
- [AD Federation](Keycloak/Keycloak%20AD%20Federation.md)
- [MFA (TOTP)](Keycloak/MFA%20Keycloak.md)
- [Issue: AD DNS and LDAPS Certificate](Keycloak/Issues/AD%20DNS%20and%20LDAPS%20Certificate.md)

### Nginx Proxy Manager
- [Overview](Nginx%20Proxy%20Manager/Overview.md)
- [Setup](Nginx%20Proxy%20Manager/Setup.md)
- [Issue: Proxmox 401 No ticket behind NPM](Nginx%20Proxy%20Manager/Issues/Fix%20Proxmox%20401%20No%20ticket%20behind%20NPM.md)
- [Issue: Linkwarden showing the default Apache page](Nginx%20Proxy%20Manager/Issues/Fix%20Linkwarden%20showing%20the%20default%20Apache%20page.md)
- [Issue: External name resolution failure](Nginx%20Proxy%20Manager/Issues/Fixing%20Failure%20in%20External%20Name%20Resolution.md)
- [Issue: Cannot access internal domains from Linux VM](Nginx%20Proxy%20Manager/Issues/Unable%20to%20access%20internal%20domains%20from%20linux%20vm.md)

### Wazuh
- [Setup](Wazuh/Setup.md)

### Linkwarden
- [Reset password via Postgres](Linkwarden/Forgot%20Password.md)

### Tailscale
- [Setup](Tailscale/Setup.md)

## Notes

Domain names, IPs, and any credentials shown in screenshots or commands are lab-internal. Passwords in code blocks are placeholders and are not the values actually in use.
