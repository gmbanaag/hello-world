# Go OAuth2 Server

This service implements [OAuth 2.0 specification](http://tools.ietf.org/html/rfc6749#section-4.3). Excerpts from the specification are included in this README file to describe different grant types. Please read the full spec for more detailed information.

# Index

* [Go OAuth2 Server](#go-oauth2-server)
* [Index](#index)
* [API](#api)
  * [OAuth 2.0](#oauth-20)    
    * [Grant Types](#grant-types)      
      * [Resource Owner Password Credentials](#resource-owner-password-credentials)
    * [Refreshing An Access Token](#refreshing-an-access-token)
    * [Token Introspection](#token-introspection)

# API

## OAuth 2.0

### Grant Types

#### Resource Owner Password Credentials

http://tools.ietf.org/html/rfc6749#section-4.3

The resource owner password credentials grant type is suitable in cases where the resource owner has a trust relationship with the client, such as the device operating system or a highly privileged application. The authorization server should take special care when enabling this grant type and only allow it when other flows are not viable.

This grant type is suitable for clients capable of obtaining the resource owner's credentials (username and password, typically using an interactive form). It is also used to migrate existing clients using direct authentication schemes such as HTTP Basic or Digest authentication to OAuth by converting the stored credentials to an access token.

```
+----------+
| Resource |
|  Owner   |
|          |
+----------+
     v
     |    Resource Owner
     (A) Password Credentials
     |
     v
+---------+                                  +---------------+
|         |>--(B)---- Resource Owner ------->|               |
|         |         Password Credentials     | Authorization |
| Client  |                                  |     Server    |
|         |<--(C)---- Access Token ---------<|               |
|         |    (w/ Optional Refresh Token)   |               |
+---------+                                  +---------------+

```

The resource owner provides the client with its username and password.

The client requests an access token from the authorization server's token endpoint by including the credentials received from the resource owner. When making the request, the client authenticates with the authorization server.

```
curl -v -X POST localhost:8080/tokens \
     -H "Content-Type:application/json" \
     -d '{ "username":"test@username","password":"test_password"}'
```

The authorization server authenticates the client and validates the resource owner credentials, and if valid, issues an access token.

```json
{
	"user_id": 1,
	"access_token": "00ccd40e-72ca-4e79-a4b6-67c95e2e3f1c",
	"expires_in": 3600,
	"token_type": "Bearer",
	"refresh_token": "6fd8d272-375a-4d8a-8d0f-43367dc8b791"
}
```


### Refreshing An Access Token

http://tools.ietf.org/html/rfc6749#section-6

If the authorization server issued a refresh token to the client, the client can make a refresh request to the token endpoint in order to refresh the access token.

```
curl -v -X POST localhost:8080/tokens \
     -H "Content-Type:application/json" \
     -d '{ "grant_type":"refresh_token","refresh_token": "6fd8d272-375a-4d8a-8d0f-43367dc8b791"}'
```

The authorization server MUST:

* require client authentication for confidential clients or for any client that was issued client credentials (or with other authentication requirements),

* authenticate the client if client authentication is included and ensure that the refresh token was issued to the authenticated client, and

* validate the refresh token.

If valid and authorized, the authorization server issues an access token.

```json
{
	"user_id": 1,
	"access_token": "1f962bd5-7890-435d-b619-584b6aa32e6c",
	"expires_in": 3600,
	"token_type": "Bearer",
	"refresh_token": "3a6b45b8-9d29-4cba-8a1b-0093e8a2b933"
}
```

The authorization server MAY issue a new refresh token, in which case the client MUST discard the old refresh token and replace it with the new refresh token.  The authorization server MAY revoke the old refresh token after issuing a new refresh token to the client.  If a new refresh token is issued, the refresh token scope MUST be identical to that of the refresh token included by the client in the request.

### Token Introspection

https://tools.ietf.org/html/rfc7662

If the authorization server issued a access token or refresh token to the client, the client can make a request to the tokeninfo endpoint in order to learn meta-information about a token.


```
curl -v -X POST localhost:8080/tokeninfo \
     -H "Content-Type:application/json" \
     -d '{ "token":"00ccd40e-72ca-4e79-a4b6-67c95e2e3f1c"}' 
```


The authorization server responds meta-information about a token.

```json
{
	"active": true,
	"username": "test@username",
	"token_type": "Bearer",
	"exp": 1454868090
}
```

