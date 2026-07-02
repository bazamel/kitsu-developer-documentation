# Security Assertion Markup Language (SAML)

Authentication can be delegated to a SAML 2.0 identity provider (Keycloak,
Microsoft Entra ID / Azure AD, Okta, OneLogin, ...). When enabled, a "Login
with &lt;provider&gt;" button appears on the Kitsu login page. Users are
redirected to the provider and, on return, a matching Kitsu account is found by
email (or created on first login).

## Activate SAML

SAML is enabled with its own flag, independently of `AUTH_STRATEGY`. Set the
following variables (in `/etc/zou/zou.env`):

```
SAML_ENABLED=true
SAML_IDP_NAME=Keycloak
SAML_METADATA_URL=https://keycloak.example.com/realms/myrealm/protocol/saml/descriptor
```

`SAML_IDP_NAME` is the label shown on the login button. `SAML_METADATA_URL`
must point at your identity provider's SAML metadata document; Zou downloads it
at startup to read the provider's endpoints and signing certificates.

## Required environment variables

* `SAML_ENABLED` (default: "False"): set to True to enable SAML SSO.
* `SAML_IDP_NAME` (default: ""): display name shown on the login button.
* `SAML_METADATA_URL` (default: ""): identity provider SAML metadata URL.

## Service provider configuration

When registering Kitsu (the service provider) with your identity provider, use:

* **Entity ID**: `<DOMAIN_PROTOCOL>://<DOMAIN_NAME>/api/auth/saml/login`
* **Assertion Consumer Service (ACS) URL**:
  `<DOMAIN_PROTOCOL>://<DOMAIN_NAME>/api/auth/saml/sso`

For example, with `https://kitsu.example.com`, the ACS URL is
`https://kitsu.example.com/api/auth/saml/sso`.

Zou's service provider expects signed assertions (`want_assertions_signed`) and
does not sign its own authentication requests, so no service-provider signing
certificate needs to be configured on the identity provider side.

## Attribute mapping

The user email is taken from the assertion's `NameID` (subject). The following
optional attributes are read from the assertion and stored on the Kitsu account
when present:

* `first_name`
* `last_name`
* `phone`
* `departments`
* `studio_id`
* `country`

Map your identity provider's attributes to these names so profiles are populated
on login.

## Account linking and provisioning

* **Linking**: on login, Zou looks up the Kitsu account whose email matches the
  assertion's `NameID` and signs that user in. Mapped attributes are refreshed
  from the assertion.
* **Provisioning**: when no account matches, one is created with the `user`
  role on first login. Anyone who can authenticate with the configured provider
  therefore gets a Kitsu account; scope membership in your identity provider
  accordingly.

## Note about Kitsu

When a SAML user is created or linked, the email, first name and last name come
from the identity provider's assertion and are refreshed on each login.
