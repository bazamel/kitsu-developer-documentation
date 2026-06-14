# OpenID Connect (OIDC) SSO

Authentication can be delegated to an OpenID Connect identity provider
(Keycloak, Microsoft Entra ID / Azure AD, Okta, Google, ...). When enabled, a
"Login with &lt;provider&gt;" button appears on the Kitsu login page. Users are
redirected to the provider and, on return, a matching Kitsu account is found by
email (or created on first login).

## Activate OIDC

OIDC is enabled with its own flag, independently of `AUTH_STRATEGY`. Set the
following variables (in `/etc/zou/zou.env`):

```
OIDC_ENABLED=true
OIDC_IDP_NAME=Keycloak
OIDC_DISCOVERY_URL=https://keycloak.example.com/realms/myrealm/.well-known/openid-configuration
OIDC_CLIENT_ID=kitsu
OIDC_CLIENT_SECRET=<secret from the provider's client>
```

`OIDC_IDP_NAME` is the label shown on the login button. `OIDC_DISCOVERY_URL`
must point at the provider's OpenID configuration document (it ends with
`/.well-known/openid-configuration`); Zou reads the endpoints, signing keys and
supported features from it.

Register this redirect URI on the provider's client:

```
<DOMAIN_PROTOCOL>://<DOMAIN_NAME>/api/auth/oidc/callback
```

For example `https://kitsu.example.com/api/auth/oidc/callback`. Authentication
uses the authorization-code flow with PKCE.

::: info
OIDC relies on Flask's signed-cookie session to carry the `state`, `nonce` and
PKCE values between the login redirect and the callback, so `SECRET_KEY` must be
set: it already is in any standard deployment.
:::

## Required environment variables

* `OIDC_ENABLED` (default: "false"): set to True to enable OIDC SSO.
* `OIDC_IDP_NAME` (default: ""): display name shown on the login button.
* `OIDC_DISCOVERY_URL` (default: ""): provider OpenID configuration URL.
* `OIDC_CLIENT_ID` (default: ""): OAuth client identifier registered with the
  provider.
* `OIDC_CLIENT_SECRET` (default: ""): OAuth client secret.

## Optional environment variables

* `OIDC_SCOPES` (default: "openid email profile"): space-separated scopes to
  request.
* `OIDC_EMAIL_CLAIM` (default: "email"): claim used as the account email.
* `OIDC_GIVEN_NAME_CLAIM` (default: "given_name"): claim used for the first
  name.
* `OIDC_FAMILY_NAME_CLAIM` (default: "family_name"): claim used for the last
  name.
* `OIDC_REQUIRE_EMAIL_VERIFIED` (default: "True"): when True, the provider must
  assert `email_verified == true`; logins with an absent or false claim are
  rejected. Set to False only for providers that do not emit the claim and
  whose emails are otherwise trusted.
* `OIDC_SKIP_2FA` (default: "False"): when True, OIDC sessions skip Kitsu's 2FA
  setup gate (trust the provider for MFA). When False, `ENFORCE_2FA` applies as
  usual.

The standard OIDC claim names work out of the box. Override the `OIDC_*_CLAIM`
variables only for providers that use non-standard names.

## Account linking and provisioning

* **Linking**: on login, Zou looks up the Kitsu account whose email matches
  the `OIDC_EMAIL_CLAIM` value and signs that user in. First and last names are
  refreshed from the provider's claims.
* **Provisioning**: when no account matches, one is created with the `user`
  role on first login. Anyone who can authenticate with the configured provider
  therefore gets a Kitsu account; scope membership in your identity provider
  accordingly.
* **Email verification**: with `OIDC_REQUIRE_EMAIL_VERIFIED` enabled (the
  default), an explicit `email_verified == true` claim is required before
  linking or provisioning, which prevents account takeover through providers
  that allow unverified addresses.

## Provider notes

The same configuration shape works for Keycloak, Microsoft Entra ID / Azure AD,
Okta and Google: point `OIDC_DISCOVERY_URL` at the provider's discovery
document.

Some providers (notably Azure AD / Entra ID) omit `given_name` / `family_name`
from the ID token and expose them only on the userinfo endpoint. Zou fetches the
userinfo endpoint as a fallback when the ID token carries no name claims, so
accounts are still created with names.
