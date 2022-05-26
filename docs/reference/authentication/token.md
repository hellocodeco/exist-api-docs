---
title: Simple token authentication
---

### Overview

All endpoints require authentication. We provide a simple token-based scheme which allows a single read-only token per user.
Make sure this token is included in your requests by including the `Authorization` header with every request.

If you are logged in to Exist in the browser your session-based authentication will also work. This is handy for browsing the API
(assuming you've set up your browser to accept JSON) but shouldn't be relied on for programmatic access.

### Request a token 

Exchange your user credentials for a token. This token will not change or expire by design but may be deprecated in the future.


#### Request

`POST /api/1/auth/simple-token/`

=== "Shell"

    ```shell

    curl -X POST "https://exist.io/api/1/auth/simple-token/" -d "username=bobby_tables" -d "password=existrulz123"
    ```

=== "Python"

    ```python
    import requests

    requests.post('https://exist.io/api/1/auth/simple-token/',
        {'username':'bobby_tables','password':'existrulz123'})
    ```


#### Parameters

Key      | Example value
-------- | --------
`username` | bobby_tables
`password` | existrulz123

#### Response

A JSON object containing a token key.


```json
{ "token": "96524c5ca126d87eb18ee7eff408ca0e71e94737" }
```


### Sign requests

Include the `Authorization: Token [your_token]` header in all requests.

=== "Shell"

    ```shell
    # With curl, you can just pass the correct header with each request
    curl "api_endpoint_here" \
      -H "Authorization: Token 96524c5ca126d87eb18ee7eff408ca0e71e94737"
    ```
    
=== "Python"
    ```python
    import requests

    requests.post(url,
        headers={'Authorization':'Token 96524c5ca126d87eb18ee7eff408ca0e71e94737'})
    ```
