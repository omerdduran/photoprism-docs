# OAuth2 Grant Types

The OAuth 2.0 specification is an authorization framework that contains a set of methods, or grants, that a client application can use to obtain an access token. Each grant type is designed for a specific use case:

- [Password Grant](auth.md#app-passwords)
- [Client Credentials](auth.md#client-credentials)

The access token can then be passed to an API endpoint, which checks it to determine validity and [authorization scope](auth.md#authorization-scopes).

!!! example ""
    Support for the [Authorization Code Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow) is planned for a future release.

## Server Endpoints

| Resource                                                              | Endpoint                  | Methods   |
|-----------------------------------------------------------------------|---------------------------|-----------|
| [Authorization](https://github.com/photoprism/photoprism/issues/4368) | `/api/v1/oauth/authorize` | GET, POST |
| [Token](https://github.com/photoprism/photoprism/issues/3943)         | `/api/v1/oauth/token`     | POST      |
| [UserInfo](https://github.com/photoprism/photoprism/issues/4369)      | `/api/v1/oauth/userinfo`  | GET, POST |
| Registration                                                          | not implemented yet       |           | 
| Introspection                                                         | not implemented yet       |           |
| [Revocation](https://github.com/photoprism/photoprism/issues/3943)    | `/api/v1/oauth/revoke`    | POST      |
| End Session                                                           | not implemented yet       |           |
| Device Authorization                                                  | not implemented yet       |           |

Clients can query the `/.well-known/oauth-authorization-server` and `/.well-known/openid-configuration` endpoints for automatic service discovery:

- <https://demo.photoprism.app/.well-known/oauth-authorization-server>
- <https://demo.photoprism.app/.well-known/openid-configuration>

!!! example ""
    Note that the [Authorization](https://github.com/photoprism/photoprism/blob/develop/internal/api/oauth_authorize.go) and [UserInfo](https://github.com/photoprism/photoprism/blob/develop/internal/api/oauth_userinfo.go) endpoints cannot be used yet as they are still [under development](https://github.com/photoprism/photoprism/issues/4368).

## Access Tokens

When clients have a valid [access token](auth.md#access-tokens), e.g. obtained through the `POST /api/v1/oauth/token` endpoint, they can use a standard *Bearer Authorization* header to authenticate their requests:

```
Authorization: Bearer <token>
```

Submitting the [access token](auth.md#access-tokens) with a custom `X-Auth-Token` header is supported as well:

```bash
curl -H "X-Auth-Token: 7dbfa37b5a3db2a9e9dd186479018bfe2e3ce5a71fc2f955" \
http://localhost:2342/api/v1/photos?count=10
```

Besides using the API endpoints provided for this, you can also generate valid [access tokens](auth.md#access-tokens) by running the `photoprism auth add` command in a terminal.

[Learn more ›](auth.md#access-tokens)

!!! example ""
    [App passwords](../../user-guide/settings/account.md#apps-and-devices) can be used as [access tokens](auth.md#access-tokens) in the *Bearer Authorization* header without first creating a session access token, and to obtain short-lived session access tokens through the `POST /api/v1/session` endpoint.

## Related Issues

- [API: Expose Prometheus-style metrics endpoint #3730](https://github.com/photoprism/photoprism/pull/3730)
- [Monitoring: Add a Prometheus-compatible API endpoint #213](https://github.com/photoprism/photoprism/issues/213)
- [Account: Add Support for 2-Factor Authentication #808](https://github.com/photoprism/photoprism/issues/808)
- [Account: Add Support for OpenID Connect (OIDC) #782](https://github.com/photoprism/photoprism/issues/782)
- [Multi-User Photo Gallery with private and shared photos/albums #98](https://github.com/photoprism/photoprism/issues/98)

## Protocol References

- https://prometheus.io/docs/prometheus/latest/configuration/configuration/#oauth2
- https://www.scottbrady91.com/oauth/client-authentication#:~:text=OAuth%20client%20secrets
- https://www.scottbrady91.com/oauth/oauth-is-not-user-authorization
- https://www.oauth.com/oauth2-servers/access-tokens/refreshing-access-tokens/
- https://www.oauth.com/oauth2-servers/access-tokens/access-token-lifetime/
- https://www.oauth.com/oauth2-servers/access-tokens/access-token-response/ 
- https://www.oauth.com/oauth2-servers/client-registration/client-id-secret/
- https://www.oauth.com/oauth2-servers/authorization/the-authorization-request/
- https://www.oauth.com/oauth2-servers/authorization/requiring-user-login/
- https://www.oauth.com/oauth2-servers/authorization/the-authorization-interface/
- https://www.oauth.com/oauth2-servers/authorization/the-authorization-response/
- https://learn.microsoft.com/en-us/linkedin/shared/authentication/programmatic-refresh-tokens
- https://oauth.net/2/grant-types/client-credentials/
- https://oauth.net/2/scope/
- https://datatracker.ietf.org/doc/html/rfc6749#section-4.4
- https://auth0.com/docs/get-started/authentication-and-authorization-flow/call-your-api-using-the-client-credentials-flow#example-post-to-token-url
- https://auth0.com/intro-to-iam/what-is-oauth-2
- https://auth0.com/docs/authenticate/protocols/oauth
- https://auth0.com/docs/secure/tokens/id-tokens/id-token-structure
- https://auth0.com/docs/authenticate/protocols/openid-connect-protocol
- https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2#grant-type-client-credentials
- https://aaronparecki.com/oauth-2-simplified/
- https://rclone.org/webdav/
- https://owncloud.dev/clients/rclone/webdav-sync-oidc/
- https://www.webdavsystem.com/server/documentation/choosing_authentication/
- http://www.webdav.org/specs/rfc2617.html#rfc.section.4.1
- https://frontegg.com/blog/oauth-grant-types

## Authentication Libraries

- https://github.com/zitadel/oidc
- https://pkg.go.dev/golang.org/x/oauth2
- https://pkg.go.dev/golang.org/x/oauth2/jwt
- https://github.com/go-oauth2/oauth2
- https://github.com/pquerna/otp

## Documentation Examples

- https://learn.microsoft.com/en-us/machine-learning-server/operationalize/how-to-manage-access-tokens
- https://docs.semui.co/administration-guide/openid
- https://api.stackexchange.com/docs/authentication
- https://dev.fitbit.com/build/reference/web-api/developer-guide/authorization/
- https://cloudentity.com/developers/basics/oauth-client-authentication/client-secret-authentication/
- https://developer.okta.com/docs/reference/api/oidc/#get-started
- https://www.authelia.com/configuration/identity-providers/open-id-connect/
- https://goauthentik.io/integrations/sources/oauth/#openid-connect
- https://developers.google.com/identity/openid-connect/openid-connect
- https://connect2id.com/products/server/docs/api
- https://connect2id.com/products/server/docs/api/discovery
- https://connect2id.com/products/server/docs/api/authorization
- https://connect2id.com/products/server/docs/api/token
- https://www.zoho.com/accounts/protocol/oauth/token-limits.html
- https://docs.github.com/en/authentication/securing-your-account-with-two-factor-authentication-2fa/configuring-two-factor-authentication-recovery-methods
- https://help.akana.com/content/current/cm/api_oauth/oauth_oauth20/m_oauth20_getTokenPOST.htm
- https://help.akana.com/content/current/cm/api_oauth/aaref/Ref_Values.htm#values_oauthgranttypes
