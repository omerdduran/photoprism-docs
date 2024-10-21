# Single Sign-On via OpenID Connect

[OpenID Connect (OIDC)](../../developer-guide/api/oidc.md) allows users to log in and optionally register through an external identity provider instead of manually entering a username and password:

![oidc-login](../../developer-guide/api/img/oidc-login.jpg){ class="shadow" }

## Config Options

| Environment              | CLI Flag        | Default            | Description                                                                                         |
|--------------------------|-----------------|--------------------|-----------------------------------------------------------------------------------------------------|
| PHOTOPRISM_OIDC_URI      | --oidc-uri      |                    | issuer `URI` for single sign-on via OpenID Connect, e.g. https://accounts.google.com                |
| PHOTOPRISM_OIDC_CLIENT   | --oidc-client   |                    | client `ID` for single sign-on via OpenID Connect                                                   |
| PHOTOPRISM_OIDC_SECRET   | --oidc-secret   |                    | client `SECRET` for single sign-on via OpenID Connect                                               |
| PHOTOPRISM_OIDC_PROVIDER | --oidc-provider |                    | custom identity provider `NAME`, e.g. Google                                                        |
| PHOTOPRISM_OIDC_ICON     | --oidc-icon     |                    | custom identity provider icon `URI`                                                                 |
| PHOTOPRISM_OIDC_REDIRECT | --oidc-redirect |                    | automatically redirect unauthenticated users to the configured identity provider                    |
| PHOTOPRISM_OIDC_REGISTER | --oidc-register |                    | allow new users to create an account when they sign in with OpenID Connect                          |
| PHOTOPRISM_OIDC_USERNAME | --oidc-username | preferred_username | preferred username `CLAIM` for new OpenID Connect users (preferred_username, name, nickname, email) |
| PHOTOPRISM_OIDC_WEBDAV   | --oidc-webdav   |                    | allow new OpenID Connect users to use WebDAV when they have a role that allows it                   |
| PHOTOPRISM_DISABLE_OIDC  | --disable-oidc  |                    | disable single sign-on via OpenID Connect, even if an identity provider has been configured         |

!!! note ""
    Your PhotoPrism instance and the [OpenID Connect Identity Provider (IdP)](#identity-providers) must be accessible **via HTTPS** and have valid TLS certificates configured for it. Please also make sure that the hostname in the [Redirect URL](#redirect-url) configured on the IdP matches the [Site URL](../../getting-started/config-options.md#site-information) used by PhotoPrism. Single sign-on via OIDC can otherwise not be enabled.

## Identity Providers

To authenticate users via OIDC, you can either set up and use a self-hosted identity provider such as [ZITADEL](https://zitadel.com/docs/self-hosting/deploy/compose) or [Keycloak](https://www.keycloak.org/), or configure a public identity provider service such as those operated by [Google](https://developers.google.com/identity/openid-connect/openid-connect), [Microsoft](https://entra.microsoft.com/), [GitHub](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/creating-an-oauth-app), or [Amazon](https://developer.amazon.com/apps-and-games/login-with-amazon).

Single sign-on can be configured automatically if the identity provider offers a standardized `/.well-known/openid-configuration` endpoint for [service discovery](https://developer.okta.com/docs/reference/api/oidc/#well-known-oauth-authorization-server), for example:

- [https://accounts.google.com/.well-known/openid-configuration](https://accounts.google.com/.well-known/openid-configuration){:target="_blank"}

### Issuer URI

The [Issuer URI](#config-options) in your configuration must match the `issuer` value returned by the [`/.well-known/openid-configuration`](https://accounts.google.com/.well-known/openid-configuration){:target="_blank"} endpoint of your [OpenID Connect Identity Provider (IdP)](#identity-providers), for example `https://accounts.google.com` if you use Google for authentication.

!!! note ""
    You may not modify the URI in any way, e.g. by adding or omitting slashes at the end. If the values do not match, the validation will fail and users cannot be redirected to your provider's login page. For security reasons, only a generic error message is displayed in this case.

### Redirect URL

The Redirect URL that must be [specified when registering a new client](../../developer-guide/api/img/redirect-url-example.jpg) with an [Identity Provider](#identity-providers) is as follows, where `{hostname}` must be replaced by the hostname in the [Site URL](../../getting-started/config-options.md#site-information), e.g. configured via `PHOTOPRISM_SITE_URL`:

```
https://{hostname}/api/v1/oidc/redirect
```

!!! note ""
    Note that both the [Site URL](../../getting-started/config-options.md#site-information) configured for your instance and the Redirect URL must start with `https://` and that their hostnames must match, as the [use of secure connections](../../getting-started/using-https.md) is a strict requirement for OpenID Connect.

## Preferred Username

When a new user signs in with OpenID Connect[^1], their preferred username may already be registered. In this case, a random 6-digit number is appended to resolve the conflict.

The config option `PHOTOPRISM_OIDC_USERNAME` allows you to change the [preferred username](#config-options) for new accounts from `preferred_username` to `name`, `nickname`, or verified `email`. Names are changed to lowercase handles so that, for example, "Jens Mander" becomes "jens.mander".

## Existing Accounts

[Super admins](../../user-guide/users/roles.md) can manually connect existing user accounts[^2] under [*Settings > Users*](../../user-guide/users/index.md) by changing the authentication to *OIDC* and then setting the *Subject ID* to match the account identifier from the configured [Identity Provider](#identity-providers):

![Edit Dialog](../../developer-guide/api/img/oidc-subject.jpg){ class="shadow" }

The *Edit Account* dialog may additionally contain a text field for the *Issuer* URL. It does not need to be entered manually as it is set automatically after the first login.

Alternatively, you can [run the following command](../../user-guide/users/cli.md#command-options) in [a terminal](../../getting-started/docker-compose.md#opening-a-terminal) to allow authentication via *OIDC* and set a *Subject ID* to connect existing accounts:

```bash
photoprism users mod --auth=oidc --auth-id=[sub] [username]
```

[Learn more ›](../../user-guide/users/cli.md#command-options)

### Passwords

Changing the authentication of an account to *OIDC* does not remove a previously set password, so that it can still be used to log in (optionally also in combination with [2FA](../../user-guide/users/2fa.md)).

If a [local password](../../user-guide/users/cli.md#changing-a-password) has been set for an account, you can remove it by running the following command [in a terminal](../../user-guide/users/cli.md#removing-a-password):

```bash
photoprism passwd --rm [username]
```

[Super admins](../../user-guide/users/roles.md#admin) can alternatively set the account password to a long random value through the [Admin Web UI](../../user-guide/users/index.md#changing-passwords) or [CLI](../../user-guide/users/cli.md#changing-a-password) to effectively prevent local authentication.

[Learn more ›](../../user-guide/users/cli.md#removing-a-password)

### Deleting Accounts

Deleted accounts remain linked to the *Subject ID*, so logging in via *OIDC* is no longer possible and no new account can be registered for the same *Subject ID* either.

If you wish to change the connected user account or create a new account instead, you must therefore [change the authentication](../../user-guide/users/index.md#editing-user-details) of the old account e.g. to *None* before [deleting it](../../user-guide/users/index.md#deleting-a-user):

```bash
photoprism users mod --auth=none [username]
```

To restore a previously deleted account, admins can follow the same steps as for [creating a new account](../../user-guide/users/cli.md#creating-a-new-account) with the same *username* through the [Admin Web UI](../../user-guide/users/index.md#adding-a-new-user) or the [`photoprism users add`](../../user-guide/users/cli.md#creating-a-new-account) command. You will then be asked if you want to restore the account.

[Learn more ›](../../user-guide/users/cli.md#creating-a-new-account)

## Frequently Asked Questions

### Is it possible to set a default role for new OIDC users?

For security reasons, our [Personal Editions](https://www.photoprism.app/editions#compare) currently default to the [Guest](../../user-guide/users/roles.md#guest) role, which admins can then upgrade after checking the eligibility of newly registered accounts. If you run our [Pro Edition](https://www.photoprism.app/teams#compare) in a trusted corporate network with appropriate security measures - including for the OIDC provider - [it can be configured](https://www.photoprism.app/pro/kb/config-options) to give new accounts a higher authorization level by default.

Please note in this context that using an external [Identity Provider](#identity-providers) for [authorization](https://en.wikipedia.org/wiki/Authorization), and not just for [authentication](https://en.wikipedia.org/wiki/Authentication), can easily lead to security issues such as the following, for which we do not want to get a CVE assigned nor do we want to be responsible for any private pictures of our users getting leaked as a result:

- https://www.microsoft.com/en-us/security/blog/2024/07/29/ransomware-operators-exploit-esxi-hypervisor-vulnerability-for-mass-encryption/
- https://support.broadcom.com/web/ecx/support-content-notification/-/external/content/SecurityAdvisories/0/24505

### Can I configure a custom claim for the preferred username?

You can choose between `preferred_username`, `name`, `nickname` and `email`, where `preferred_username` is the default. The other claims are used as fallback if no value is returned for the [configured claim](#config-options).

Please note that it is currently not possible to use [other standard](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims) or non-standard claims, as these may not be suitable for generating a username and no logic is implemented for doing so.

[Learn more ›](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims)

[^1]: `PHOTOPRISM_OIDC_REGISTER` must be set to `"true"` to allow new users to create an account
[^2]: Admins cannot change the authentication of their own user account through the [Admin Web UI](../../user-guide/users/index.md#editing-user-details) so that they do not accidentally lock themselves out e.g. by setting it to *None*.