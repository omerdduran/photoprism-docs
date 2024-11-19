# Single Sign-On via OpenID Connect

[OpenID Connect (OIDC)](https://openid.net/developers/how-connect-works/) extends [OAuth 2.0](oauth2.md) to allow users to log in and optionally register through an external identity provider instead of manually entering a username and password:

![oidc-login](img/oidc-login.jpg){ class="shadow" }

## Authentication Flow

![oidc-sso-flow](img/oidc-sso-flow.jpg)

[Learn more ›](https://dl.photoprism.app/pdf/publications/20220113-Volkmann_OpenID_Connect_Thesis.pdf)

## Config Options

| Environment              | CLI Flag        | Default                      | Description                                                                                         |
|--------------------------|-----------------|------------------------------|-----------------------------------------------------------------------------------------------------|
| PHOTOPRISM_OIDC_URI      | --oidc-uri      |                              | issuer `URI` for single sign-on via OpenID Connect, e.g. https://accounts.google.com                |
| PHOTOPRISM_OIDC_CLIENT   | --oidc-client   |                              | client `ID` for single sign-on via OpenID Connect                                                   |
| PHOTOPRISM_OIDC_SECRET   | --oidc-secret   |                              | client `SECRET` for single sign-on via OpenID Connect                                               |
| PHOTOPRISM_OIDC_SCOPES   | --oidc-scopes   | openid email profile address | client authorization `SCOPES` for single sign-on via OpenID Connect                                 |
| PHOTOPRISM_OIDC_PROVIDER | --oidc-provider |                              | custom identity provider `NAME`, e.g. Google                                                        |
| PHOTOPRISM_OIDC_ICON     | --oidc-icon     |                              | custom identity provider icon `URI`                                                                 |
| PHOTOPRISM_OIDC_REDIRECT | --oidc-redirect |                              | automatically redirect unauthenticated users to the configured identity provider                    |
| PHOTOPRISM_OIDC_REGISTER | --oidc-register |                              | allow new users to create an account when they sign in with OpenID Connect                          |
| PHOTOPRISM_OIDC_USERNAME | --oidc-username | preferred_username           | preferred username `CLAIM` for new OpenID Connect users (preferred_username, name, nickname, email) |
| PHOTOPRISM_OIDC_WEBDAV   | --oidc-webdav   |                              | allow new OpenID Connect users to use WebDAV when they have a role that allows it                   |
| PHOTOPRISM_DISABLE_OIDC  | --disable-oidc  |                              | disable single sign-on via OpenID Connect, even if an identity provider has been configured         |

!!! example ""
    Your PhotoPrism instance and the [OpenID Connect Identity Provider (IdP)](#identity-providers) must be accessible **via HTTPS** and have valid TLS certificates configured for it. Please also make sure that the hostname in the [Redirect URL](#redirect-url) configured on the IdP matches the [Site URL](../../getting-started/config-options.md#site-information) used by PhotoPrism. Single sign-on via OIDC can otherwise not be enabled.

## Identity Providers

To allow users to log in via OIDC, you can either set up and use a self-hosted identity provider such as [ZITADEL](https://zitadel.com/docs/self-hosting/deploy/compose) or [Keycloak](https://www.keycloak.org/), or choose a public authentication service such as those provided by [Google](https://developers.google.com/identity/openid-connect/openid-connect), [Microsoft](https://entra.microsoft.com/), [GitHub](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/creating-an-oauth-app), or [Amazon](https://developer.amazon.com/apps-and-games/login-with-amazon).

Single sign-on can then be configured automatically through your identity provider's `/.well-known/openid-configuration` [service discovery endpoint](https://developer.okta.com/docs/concepts/auth-servers/#discovery-endpoints-org-authorization-servers), for example:

- [https://accounts.google.com/.well-known/openid-configuration](https://accounts.google.com/.well-known/openid-configuration){:target="_blank"}

### Issuer URI

The [Issuer URI](#config-options) in your configuration must match the `issuer` value returned by the [`/.well-known/openid-configuration`](https://accounts.google.com/.well-known/openid-configuration){:target="_blank"} endpoint of your [OpenID Connect Identity Provider (IdP)](#identity-providers), for example `https://accounts.google.com` if you use Google for authentication.

!!! example ""
    You may not modify the URI in any way, e.g. by adding or omitting slashes at the end. If the values do not match, the validation will fail and users cannot be redirected to your provider's login page. For security reasons, only a generic error message is displayed in this case.

### Redirect URL

The Redirect URL that must be [specified when registering a new client](img/redirect-url-example.jpg) with an [Identity Provider](#identity-providers) is as follows, where `{hostname}` must be replaced by the hostname in the [Site URL](../../getting-started/config-options.md#site-information), e.g. configured via `PHOTOPRISM_SITE_URL`:

```
https://{hostname}/api/v1/oidc/redirect
```

!!! example ""
    Note that both the [Site URL](../../getting-started/config-options.md#site-information) configured for your instance and the Redirect URL must start with `https://` and that their hostnames must match, as the [use of secure connections](../../getting-started/using-https.md) is a strict requirement for OpenID Connect.

### Local Development

Our [development environment](../setup.md) comes with a [pre-configured Keycloak](https://github.com/photoprism/photoprism/blob/develop/compose.yaml#L217-L243) OIDC Identity Provider running at <https://keycloak.localssl.dev/> for local testing:

- <https://keycloak.localssl.dev/realms/master/.well-known/openid-configuration>

An `admin` account for managing users and a `user` account for testing single sign-on are pre-registered. Both have the password `photoprism`.

!!! example ""
    Please do not use this test identity provider in a production environment as its configuration is not secure.

## Preferred Username

When a new user signs in with OpenID Connect[^1], their preferred username may already be registered. In this case, a random 6-digit number is appended to resolve the conflict.

The config option `PHOTOPRISM_OIDC_USERNAME` allows you to change the [preferred username](#config-options) claim for new accounts from `preferred_username` to `name`, `nickname` or verified[^2] `email`. The other claims are used as fallback if no value is returned for the configured claim. Names are changed to lowercase so that, for example, "John Doe" becomes "john.doe".

[Learn more ›](#can-i-configure-a-custom-claim-for-the-preferred-username)

## Existing Accounts

[Super admins](../../user-guide/users/roles.md) can manually connect existing user accounts[^3] under [*Settings > Users*](../../user-guide/users/index.md) by changing the authentication to *OIDC* and then setting the *Subject ID* to match the account identifier from the configured [Identity Provider](#identity-providers):

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

## Service Discovery

It is not yet possible to use PhotoPrism as an [Identity Provider](#identity-providers), since not all the required [endpoints](https://github.com/photoprism/photoprism/issues/4368) and [grant types](oauth2.md) have been fully implemented.

However, querying the `/.well-known/openid-configuration` endpoint shows what has already been implemented, so the missing capabilities can be identified and added over time:

- <https://demo.photoprism.app/.well-known/openid-configuration>

## Related Issues

- [Auth: Add support for single sign-on via OpenID Connect (OIDC) #782](https://github.com/photoprism/photoprism/issues/782)
- [Auth: Add `userinfo` API endpoint to get information about the logged in user #4369](https://github.com/photoprism/photoprism/issues/4369)
- [Auth: Add `authorize` API endpoint to implement the authorization code flow #4368](https://github.com/photoprism/photoprism/issues/4368)

## Software Libraries

- https://github.com/zitadel/oidc by https://zitadel.com/

## Protocol References

- https://openid.net/developers/how-connect-works/
- https://dl.photoprism.app/pdf/publications/20220113-Volkmann_OpenID_Connect_Thesis.pdf
- https://oauth.net/openid-for-verifiable-credentials/
- https://developers.google.com/identity/openid-connect/openid-connect
- https://www.ory.sh/docs/oauth2-oidc/authorization-code-flow
- https://developer.okta.com/docs/concepts/oauth-openid/
- https://developer.okta.com/docs/reference/api/oidc/
- https://developer.okta.com/docs/reference/api/oauth-clients/
- https://auth0.com/docs/authenticate/protocols/openid-connect-protocol
- https://learn.microsoft.com/en-us/entra/identity-platform/scopes-oidc#openid-connect-scopes
- https://owncloud.dev/clients/rclone/webdav-sync-oidc/
- https://blog.cubieserver.de/2022/complete-guide-to-nextcloud-oidc-authentication-with-authentik/
- https://auth0.com/docs/get-started/applications/configure-applications-with-oidc-discovery
- https://connect2id.com/products/server/docs/api/authorization
- https://www.authlete.com/developers/definitive_guide/authorization_endpoint_spec/
- https://www.oauth.com/oauth2-servers/authorization/the-authorization-request/
- https://www.oauth.com/oauth2-servers/authorization/requiring-user-login/
- https://www.oauth.com/oauth2-servers/authorization/the-authorization-interface/
- https://www.oauth.com/oauth2-servers/authorization/the-authorization-response/

## Session Management

- https://openid.net/specs/openid-connect-session-1_0.html#OPiframe
- https://zitadel.com/blog/session-timeouts-logouts
- https://ldapwiki.com/wiki/Wiki.jsp?page=OpenID%20Connect%20Session%20Management
- https://ldapwiki.com/wiki/Wiki.jsp?page=OpenID%20Connect%20Back-Channel%20Logout
- https://openid.net/specs/openid-connect-rpinitiated-1_0.html
- https://auth0.com/docs/authenticate/login/logout/log-users-out-of-auth0
- https://medium.com/@robert.broeckelmann/openid-connect-logout-eccc73df758f

## Frequently Asked Questions

### Is it possible to set a default role for new OIDC users?

For security reasons, our [Personal Editions](https://www.photoprism.app/editions#compare) currently default to the [Guest](../../user-guide/users/roles.md#guest) role, which admins can then upgrade after checking the eligibility of newly registered accounts.

[Learn more ›](../../getting-started/advanced/openid-connect.md#frequently-asked-questions)

### Can I configure a custom claim for the preferred username?

You can choose between `preferred_username`, `name`, `nickname` and verified[^2] `email`, where `preferred_username` is the default. The other claims are used as fallback if no value is returned for the [configured claim](#config-options).

Please note that it is currently not possible to use [other standard](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims) or non-standard claims, as these may not be [suitable for generating a username](#preferred-username) and [no logic is implemented](https://github.com/photoprism/photoprism/blob/develop/internal/auth/oidc/username.go) for doing so.

[Learn more ›](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims)

### What if my provider does not return any claims for generating a username?

Certified [OIDC Identity Providers](#identity-providers), as well as public service providers such as [Google](https://developers.google.com/identity/openid-connect/openid-connect), should support (at least) a subset of the [standard claims](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims) that PhotoPrism can use to [generate a username](#preferred-username) for newly registered accounts. These are `preferred_username`, `name`, `nickname` and verified[^2] `email`, where `preferred_username` is the default.

If only an unverified `email` address[^2] or none of [these claims](#can-i-configure-a-custom-claim-for-the-preferred-username) are returned, we recommend that you report this to the vendor of the [Identity Provider](#identity-providers) you are using, as this is also likely to cause problems with other software. In a future release, we may offer additional [config options](#config-options) to work around this issue, e.g. by generating a random username. However, this is currently not possible.

[Learn more ›](#preferred-username)

[^1]: `PHOTOPRISM_OIDC_REGISTER` must be set to `"true"` to allow new users to create an account via OpenID Connect.
[^2]: The `email_verified` flag must be set by the [OIDC Identity Provider](#identity-providers) so that the `email` address can be used to send notifications and/or confirm the identity of users. If we do not insist on verification, this could otherwise have a negative impact on trust and security.
[^3]: Admins are unable to change the authentication method of their own user account through the [Admin Web UI](../../user-guide/users/index.md#editing-user-details), so they cannot accidentally lock themselves out e.g. by setting it to *None*.