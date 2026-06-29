---
icon: lucide/users-round
---

# SSO and SCIM

Arcessa supports OIDC, SAML, multi-IdP login, and SCIM 2.0 provisioning.

## OIDC

Environment-based provider:

```bash
OIDC_ISSUER=https://idp.example.com
OIDC_CLIENT_ID=arcessa
OIDC_CLIENT_SECRET=<secret>
OIDC_REDIRECT_URL=http://localhost:8080/api/v1/auth/oidc/callback
OIDC_PROVIDER_NAME=Company SSO
```

Role and team mapping:

```bash
SSO_ROLE_CLAIMS=roles,groups,realm_access.roles
SSO_TEAM_CLAIMS=groups
SSO_ROLE_MAP=platform-admin:admin,tool-editor:editor
SSO_TEAM_MAP=/engineering:engineering,/security:security
SSO_DEFAULT_ROLE=viewer
SSO_SYNC_TEAMS=true
```

## Multi-IdP providers

Admins can create additional OIDC providers through the Authentication page or the provider API. Provider creation validates OIDC discovery and encrypts the client secret when `MASTER_KEY` is set.

Use cases:

- Separate workforce and partner IdPs.
- Separate Okta, Entra, and Keycloak tenants.
- Pilot a new IdP without changing the primary login button.

## SAML

SAML is configured as an SP with IdP metadata:

```bash
SAML_IDP_METADATA_URL=https://idp.example.com/metadata
SAML_SP_CERT="$(cat sp.crt)"
SAML_SP_KEY="$(cat sp.key)"
SAML_ROOT_URL=http://localhost:8080
```

Production checklist:

- Use signed assertions.
- Validate audience and recipient.
- Keep SP private key in a secret store.
- Fetch IdP metadata through SSRF-guarded HTTP.
- Test browser SP-init flow against the real IdP.

## SCIM 2.0

Enable SCIM:

```bash
SCIM_BEARER_TOKEN=<strong provisioning secret>
SCIM_ORG_ID=<target-org-id>
```

SCIM endpoints:

```text
/scim/v2/ServiceProviderConfig
/scim/v2/ResourceTypes
/scim/v2/Schemas
/scim/v2/Users
```

Supported behavior includes user CRUD, filtering, PATCH, PUT replace, pagination, and enterprise-extension tolerance. `/Groups` and `/Bulk` are optional SCIM capabilities and should be documented as unsupported unless implemented.

## Real provider testing

Use the matrix stack for provider differences:

```bash
make real-up
make matrix-up
make matrix-identity
make matrix-saml
```

The matrix should exercise Keycloak, Dex, Hydra, SCIM RFC resource endpoints, and SAML browser flows.
