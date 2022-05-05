---
title: Authentication
---

# Authentication

## Overview

There are two authentication methods — simple tokens, and OAuth2 clients. So which one do you need?

**Simple token authentication** is read-only and exists as a basic means for users to access their own data from Exist. This method is available to everyone
by exchanging a username and password for a token that doesn't expire. This is only recommended for quickly building a single-user, read-only client,
and may be deprecated in future.

**OAuth2 clients** are superior to simple-token authentication as they can acquire control of attributes and write values for attributes.
One client application can create and use access tokens for many users. One user may have many clients authorised to access their Exist account, each with a separate token that can be revoked.

OAuth2 clients are now available to all users. You can create one from your [app management page](https://exist.io/account/apps/) within your Exist account.

## Simple token authentication

All endpoints require authentication. We use a simple token-based scheme which allows a single token per user.
Make sure this token is included in your requests by including the `Authorization` header with every request.

If you are logged in to Exist in the browser your session-based authentication will also work. This is handy for browsing the API
(assuming you've set up your browser to accept JSON) but shouldn't be relied on for programmatic access.

### Requesting a token 


```shell

curl https://exist.io/api/1/auth/simple-token/ -d username=bobby_tables -d password=existrulz123
```

```python
import requests

requests.post('https://exist.io/api/1/auth/simple-token/',
    {'username':'bobby_tables','password':'existrulz123'})
```

> Returns a token object in JSON:

```json
{ "token": "96524c5ca126d87eb18ee7eff408ca0e71e94737" }
```

Exchange your user credentials for a token. This token will not change or expire by design but may be deprecated as we move to OAuth2 in the future.

#### Request

`POST /api/1/auth/simple-token/`

#### Parameters

Key      | Example value
-------- | --------
`username` | bobby_tables
`password` | existrulz123

#### Response

A JSON object containing a token key.

### Signing requests

Include the `Authorization: Token [your_token]` header in all requests.


```python
import requests

requests.post(url,
    headers={'Authorization':'Token 96524c5ca126d87eb18ee7eff408ca0e71e94737'})
```

```shell
# With curl, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: Token 96524c5ca126d87eb18ee7eff408ca0e71e94737"
```

## OAuth2 authentication

All endpoints require authentication, except those that are part of the OAuth2 authorisation flow.
Make sure your access token is included in your requests by including the `Authorization: Bearer` header with every request.

You may mix and match OAuth2 authentication with simple token or even session-based authentication as you test the API. API endpoints will respond with JSON within your browser
if you are logged in to the site.

### Authorisation flow

Send your user to the authorisation page at `https://exist.io/oauth2/authorize`

=== "Shell"

    ```shell
    # We can't really do this from the shell, but your URL would look like this:

    curl https://exist.io/oauth2/authorize?response_type=code&client_id=[your_id]&redirect_uri=[your_uri]&scope=[your_scope]
    ```

=== "Python"

    ```python
    # in django, we would do something like this
    return redirect('https://exist.io/oauth2/authorize?response_type=code&client_id=%s&redirect_uri=%s&scope=%s' % (CLIENT_ID, REDIRECT_URI,"read+write"))
    ```

User authorises your client by hitting 'Allow', and
Exist returns the user to your `redirect_uri` with `code=[some_code]` in the query string.
Exchange your code for an access token: 

=== "Shell"
    ```shell
    curl -X POST https://exist.io/oauth2/access_token -d "grant_type=authorization_code" -d "code=[some_code]" -d "client_id=[your_id]" -d "client_secret=[your_secret]" -d "redirect_uri=[your_uri]"
    ```

=== "Python"
    ```python
    import requests

    url = 'https://exist.io/oauth2/access_token'

    response = requests.post(url,
            {'grant_type':'authorization_code',
                'code':code,
                'client_id':CLIENT_ID,
                'client_secret':CLIENT_SECRET,
                'redirect_uri':REDIRECT_URI })
    ```

Returns JSON if your request was successful:

```json
{ 
    "access_token": "122bb8707b6aee134e7746a40feca41868ddd578", 
    "token_type": "Bearer",
     "expires_in": 31535999, 
     "refresh_token": "ac45027ad037f53b3ce91be272b163f55a4a87e9", 
     "scope": "read write read+write" 
}
```

The OAuth2 authorisation flow is vastly simpler than the original OAuth 1.0:

1. Send your user to the "request authorisation" page at `/oauth2/authorize` with these parameters:
  *  `response_type=code` to request an auth code in return   
  *  `redirect_uri` with the URI to which Exist returns the user (**must be HTTPS**)
  *  `scope=read` or `scope=read+write` to request read or read/write permissions
  *  `client_id` which is your OAuth2 client ID
2. User authorises your application within the requested scopes (by hitting 'Allow' in the browser)
3. Exist returns the user to your `redirect_uri` (GET request) with the following:
  *  `code` parameter upon success
  *  `error` parameter if the user didn't authorise your client, or any other error with your request
4. Exchange this code for an access token by POSTing to `/oauth2/access_token` these parameters:
  *  `grant_type=authorization_code`
  *  `code` with the code you just received
  *  `client_id` with your OAuth2 client ID
  *  `client_secret` with your OAuth2 client secret
  *  `redirect_uri` with the URI you used earlier
5. If successful you will receive a JSON object with an `access_token`, `refresh_token`, `token_type`, `scope`, and `expires_in` time in seconds. 

### Refreshing an access token

=== "Shell"

    ```shell
    curl -X POST https://exist.io/oauth2/access_token -d "grant_type=refresh_token" -d "refresh_token=[token]" -d "client_id=[your_id]" -d "client_secret=[your_secret]"
    ```

=== "Python"

    ```python
    import requests

    url = 'https://exist.io/oauth2/access_token'

    response = requests.post(url,
            {'grant_type':'refresh_token',
                'refresh_token':token,
                'client_id':CLIENT_ID,
                'client_secret':CLIENT_SECRET 
            })
    ```

Returns JSON if your request was successful:

```json
{ 
  "access_token": "122bb8707b6aee134e7746a40feca41868ddd578", 
  "token_type": "Bearer", 
  "expires_in": 31535999, 
  "refresh_token": "ac45027ad037f53b3ce91be272b163f55a4a87e9", 
  "scope": "read write read+write" 
}
```

Tokens expire in a year and can be refreshed at any time, invalidating the original access and refresh tokens.


#### Request

`POST /oauth2/access_token`

#### Parameters

Name  | Description
------|--------
`refresh_token` | The refresh token previously received in the auth flow
`grant_type` | `refresh_token`
`client_id` | Your OAuth2 client ID
`client_secret` | Your OAuth2 client secret

#### Response

The same as your original access token response, a JSON object with an `access_token`, `refresh_token`, `token_type`, `scope`, and `expires_in` time in seconds. 

### Signing requests


=== "Shell"

    ```shell
    # With curl, you can just pass the correct header with each request
    curl "api_endpoint_here"
    -H "Authorization: Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737"
    ```

=== "Python"

    ```python
    import requests

    requests.post(url,
        headers={'Authorization':'Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737'})
    ```

Sign all authenticated requests by adding the Authorization header, `Authorization: Bearer [access_token]`. Note that this differs from the simple token-based authentication by using `Bearer`, *not* `Token`.


