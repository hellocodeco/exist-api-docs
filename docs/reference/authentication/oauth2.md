---
title: OAuth2 authentication
---

### Overview

All endpoints require authentication, except those that are part of the OAuth2 authorisation flow.
Make sure your access token is included in your requests by including the `Authorization: Bearer` header with every request.

You may mix and match OAuth2 authentication with simple token or even session-based authentication as you test the API. API endpoints will respond with JSON within your browser if you are logged in to the site.

Remember you need to create an OAuth2 client in Exist to use this method. OAuth2 clients can be created by all users. You can create one from your [app management page](https://exist.io/account/apps/) within your Exist account. Make a note of your client's credentials as you'll need them to authorise users.

### Scopes

Be responsible and request as few scopes as required to provide your necessary functionality. Only attributes that match the scope (that is, are in the groups you're allowed to read or write, or are manual) will be accessible. For example, if you request the contents of the `mood` group in an attributes endpoint, but you only have `activity_read` scope, you will receive an empty list.

**Read scopes** allow you to *retrieve attributes and their values* for those that match the scope.

**Write scopes** allow you to *own, create, and write values for attributes* that match the scope.


| Scope  | Description |
|--------|------|
| **Read scopes** | |
| `activity_read` | Activity group |
| `productivity_read` | Productivity group |
| `mood_read` | Mood group |
| `sleep_read` | Sleep group |
| `workouts_read` | Workouts group |
| `events_read` | Finance group |
| `food_read` | Food and drink group |
| `health_read` | Health and body group |
| `location_read` | Location group |
| `media_read` | Media group |
| `social_read` | Social group |
| `weather_read` | Weather group |
| `custom_read` | Custom tags group |
| `manual_read` | Read any manually tracked attribute |
| **Write scopes** | |
| `activity_write` | Activity group |
| `productivity_write` | Productivity group |
| `mood_write` | Mood group |
| `sleep_write` | Sleep group |
| `workouts_write` | Workouts group |
| `events_write` | Finance group |
| `food_write` | Food and drink group |
| `health_write` | Health and body group |
| `location_write` | Location group |
| `media_write` | Media group |
| `social_write` | Social group |
| `weather_write` | Weather group |
| `custom_write` | Custom tags group |
| `manual_write` | Write any manually tracked attribute |


### Authorisation flow


Let's go through the multiple steps required to receive a token for a user:

1. Send your user to the "request authorisation" page at `/oauth2/authorize` with these parameters:
    *  `response_type=code` to request an auth code in return   
    *  `redirect_uri` with the URI to which Exist returns the user (**must be HTTPS**)
    *  `scope=scope1+scope2` with the list of scopes you're asking for
    *  `client_id` which is your OAuth2 client ID
2. User views the details of your client and its requested scopes, then authorises your application with the requested scopes (by hitting 'Allow' in the browser)
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

#### Code example

Send your user to the authorisation page at `https://exist.io/oauth2/authorize`

=== "Shell"

    ```shell
    # We can't really do this from the shell, but your URL would look like this:

    curl "https://exist.io/oauth2/authorize?response_type=code&client_id=[your_id]&redirect_uri=[your_uri]&scope=[your_scope]"
    ```

=== "Python"

    ```python
    # in django, we would do something like this
    return redirect('https://exist.io/oauth2/authorize?response_type=code&client_id=%s&redirect_uri=%s&scope=%s' % (CLIENT_ID, REDIRECT_URI,"[your_scope]")
    )
    ```

User authorises your client by hitting 'Allow', and
Exist returns the user to your `redirect_uri` with `code=[some_code]` in the query string.
Exchange your code for an access token: 

=== "Shell"
    ```shell
    curl -X POST "https://exist.io/oauth2/access_token" -d "grant_type=authorization_code" -d "code=[some_code]" -d "client_id=[your_id]" -d "client_secret=[your_secret]" -d "redirect_uri=[your_uri]"
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
     "scope": "your_scopes" 
}
```


### Refresh an access token

Tokens expire in a year and can be refreshed at any time, invalidating the original access and refresh tokens.

=== "Shell"

    ```shell
    curl -X POST "https://exist.io/oauth2/access_token" -d "grant_type=refresh_token" -d "refresh_token=[token]" -d "client_id=[your_id]" -d "client_secret=[your_secret]"
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

### Sign requests

Sign all authenticated requests by adding the Authorization header, `Authorization: Bearer [access_token]`. Note that this differs from the simple token-based authentication by using `Bearer`, *not* `Token`.



=== "Shell"

    ```shell
    # With curl, you can just pass the correct header with each request
    curl "api_endpoint_here" -H "Authorization: Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737"
    ```

=== "Python"

    ```python
    import requests

    requests.post(url,
        headers={'Authorization':'Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737'})
    ```


