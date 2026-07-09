---
tags: [homelab, keycloak, sso, security]
created: 2026-07-01
---

# Homelab SSO / Segmentation Architecture

## Service Inventory (target plan)

| IP             | Port | Hostname/Service | VLAN | Domain                 | Use SSO | Native OIDC   |
| -------------- | ---- | ---------------- | ---- | ---------------------- | ------- | ------------- |
| Managed        | 80   | OPNsense         | 0    | N/A                    | No      | N/A           |
| 192.168.99.101 | 8006 | Pve1             | 99   | pve1.keenan.sec        | No      | N/A           |
| 192.168.99.102 | 8006 | Pve2             | 99   | pve2.keenan.sec        | No      | N/A           |
| Dynamic        | N/A  | TestVM           | Any  | N/A                    | No      | N/A           |
| 192.168.10.101 | 81   | NPM              | 10   | npm.keenan.sec         | No      | N/A           |
| 192.168.20.101 |      | WikiJS           | 20   | wiki.keenan.sec        | Yes     | Yes (Native)  |
| 192.168.20.102 | 5678 | n8n              | 20   | n8n.keenan.sec         | Yes     | No            |
| 192.168.20.103 |      | EspoCRM          | 20   | crm.keenan.sec         | Yes     | Yes (Native)  |
| 192.168.20.104 |      | Immich           | 20   | immich.keenan.sec      | Yes     | Yes (Native)  |
| 192.168.20.105 | 3000 | Linkwarden       | 20   | linkwarden.keenan.sec  | Yes     | Yes (Native)  |
| 192.168.30.100 | 80   | Pihole           | 30   | pihole.keenan.sec      | Yes     | No            |
| 192.168.30.101 | 8000 | Vaultwarden      | 30   | vaultwarden.keenan.sec | Yes     | Yes           |
| 192.168.30.102 |      | Keycloak         | 30   | keycloak.keenan.sec    | No      | N/A           |
| 192.168.30.103 | N/A  | WinServ/AD/DC    | 30   | keenan.sec             | No      | N/A           |
| 192.168.40.101 |      | Wazuh            | 40   | wazuh.keenan.sec       | No      | Yes           |
| 192.168.40.102 |      | osTicket         | 40   | ticket.keenan.sec      | Yes     | Yes           |
| 192.168.40.103 |      | thehive          | 40   | hive.keenan.sec        | Yes     | Yes           |

## Core Enforcement Rules

1. All traffic passes through OPNsense (firewall).
2. All web panels route through NPM (reverse proxy). No direct client access to any backend service IP/port.
3. Firewall rule per service VLAN: only NPM's IP (or oauth2-proxy's IP where used) may reach the service's admin port. Default deny everything else. Check rule order, first match wins in OPNsense.
4. Keycloak is the single identity source (federated to AD over LDAPS). Nothing else independently authenticates against AD.
5. Services with native OIDC talk to Keycloak directly. Services without it sit behind oauth2-proxy, which is the only thing that actually validates tokens and redirects unauthenticated users.

## Correction Log

- Authelia does **not** support the OIDC Relying Party role (only Provider, in beta). It cannot sit downstream of Keycloak. Ruled out as a shared gateway.
- Authentik *can* act as a relying party to an upstream OIDC IdP, if a shared gateway is ever wanted instead of N oauth2-proxy containers. Bigger footprint (Postgres + Redis).
- Wazuh has **native SAML SSO** support via the indexer's opensearch-security plugin, config'd against Keycloak. Not just "no SSO," reclassify below.

## Service Classification

| Service        | SSO Method                     | Notes                                                                                                                                           |
| -------------- | ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Pi-hole        | oauth2-proxy to Keycloak (OIDC)| No native auth integration. `keycloak-oidc` provider, not legacy `keycloak`.                                                                    |
| Linkwarden     | Native OIDC to Keycloak        | Built-in since v2.3. `KEYCLOAK_CLIENT_ID`/`SECRET`/`ISSUER` env vars.                                                                           |
| Vaultwarden    | Native OIDC to Keycloak        | `SSO_ENABLED`, `SSO_AUTHORITY`, `SSO_CLIENT_ID`, `SSO_CLIENT_SECRET` env vars. Master password still required after SSO login, that's expected. |
| WikiJs         | Native OIDC to Keycloak        | Admin, Modules, Authentication, Generic OpenID Connect / OAuth2 strategy.                                                                       |
| Immich         | Native OIDC to Keycloak        | Admin, Settings, OAuth Authentication. Keycloak explicitly supported.                                                                           |
| Wazuh          | Native SAML to Keycloak        | Configured at indexer level (`opensearch-security/config.yml`), not dashboard-only. Inherits Keycloak's TOTP policy.                            |
| OPNsense       | None (by design)               | Local accounts + native TOTP. Don't wire to Keycloak, firewall must stay accessible if Keycloak is down.                                        |
| NPM            | None natively                  | No OIDC support in NPM itself. If NPM's own admin panel needs protecting, put oauth2-proxy in front of it (separate from its ingress role).     |
| Windows Server | AD-native                      | Out of scope for this SSO layer, AD is the backing directory for Keycloak already.                                                              |

## VLAN / Firewall Pattern (per service)

- Place each service on its own VLAN/segment.
- Rule: allow inbound on service's admin port **only from NPM's IP** (or oauth2-proxy's IP for Pi-hole).
- Deny all other sources to that port, default deny at bottom of VLAN ruleset.
- Verify: `curl -k https://<service-internal-ip>` from another VLAN should fail. Check OPNsense firewall logs to confirm the block is actually firing, not shadowed by a broader allow rule above it.
- DNS exception: Pi-hole port 53 stays open to client VLANs that use it for resolution. Only the admin UI port gets locked to NPM/oauth2-proxy.

## oauth2-proxy (Pi-hole only)

- Pin image version (`v7.15.3`), don't run `:latest`.
- Provider: `keycloak-oidc` (legacy `keycloak` provider is deprecated).
- Do not enable `--reverse-proxy` unless required; do not use `--skip_auth_routes`/`--skip-auth-regex` (CVE-2026-40575 auth bypass requires this combination).
- NPM `auth_request` block hits `/oauth2/auth`; unauth'd requests get bounced to Keycloak login via oauth2-proxy's redirect.

## Keycloak Client Registration (per service, general pattern)

1. Clients, Create client. Client ID matches the service's expected value.
2. Client authentication: On (confidential).
3. Standard flow: enabled.
4. Valid redirect URIs: service's documented callback path.
5. Copy client secret from Credentials tab.
6. Set service's env vars / admin UI fields to match.

Can script repeat registrations via `kcadm.sh` instead of clicking through UI each time.

# 2FA Plan

## Principle

Anything behind Keycloak SSO inherits Keycloak's MFA policy automatically, once TOTP is enforced at the realm/flow level, every service in the SSO list gets it for free. Anything **not** behind Keycloak (OPNsense) needs its own native 2FA, since it deliberately stays off SSO.

## Keycloak TOTP (governs Pi-hole, Linkwarden, Vaultwarden, WikiJs, Immich, Wazuh)

1. Admin console, realm, Authentication, Policies, OTP Policy. Defaults (TOTP, SHA1, 6 digits, 30s) work with Google Authenticator, Authy, etc.
2. Authentication, Required Actions, enable "Configure OTP."
3. To force for everyone: Authentication, Browser flow, duplicate the flow, set OTP execution to **Required** (not Conditional), bind the new flow as the realm's browser flow.
4. To scope to specific accounts only (e.g. admin-tier): use a **Conditional OTP** subflow keyed on group membership instead of forcing realm-wide.

This single setting covers every service above once its OIDC/SAML client is wired up, no per-service MFA config needed.

## Wazuh specifically

Once the SAML client is registered in Keycloak (indexer's `openid_auth_domain` or `saml_auth_domain` config), Wazuh dashboard login redirects to Keycloak, MFA enforced there. No separate MFA setup inside Wazuh itself needed. There's an open native-MFA feature request in Wazuh (GitHub issue #30561), not implemented; Keycloak SSO is the workaround.

## OPNsense (native, not tied to Keycloak)

1. System, Access, Users, edit target user.
2. Enable "OTP seed," generate/scan the QR with Google Authenticator.
3. System, Settings, Administration, confirm the auth method requires OTP for GUI login.

Keep this independent of Keycloak. If Keycloak or its VM goes down, you still need to get into the firewall to fix things.

## NPM admin panel

No native MFA. If protecting the NPM admin UI itself matters, same oauth2-proxy-in-front pattern as Pi-hole, evaluate whether that's worth the complexity given NPM is also the ingress point everything else depends on.

# To Do

- [ ] Keycloak: enable OTP policy, set Required in browser flow (or conditional by group)
- [ ] Register Keycloak OIDC clients: Linkwarden, Vaultwarden, WikiJs, Immich
- [ ] Register Keycloak SAML client: Wazuh (indexer-level config)
- [ ] Deploy oauth2-proxy for Pi-hole only (pinned v7.15.3, `keycloak-oidc`, no skip-auth flags)
- [ ] VLAN placement for each service
- [ ] Firewall rules: service admin ports restricted to NPM/oauth2-proxy IP only, default deny
- [ ] Verify segmentation: cross-VLAN curl test + firewall log check
- [ ] OPNsense native TOTP enabled for admin accounts
- [ ] Decide on NPM admin panel protection (optional oauth2-proxy layer)
- [ ] Enable FIM on `/etc/pihole/custom.list` to catch changes to local DNS records
- [ ] Mandate all services only accept inbound traffic from NPM
