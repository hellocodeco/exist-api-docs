---
title: Users
---

## Get profile stub for user

Returns an overview of the user's personal details.

### Request

`GET /api/2/accounts/profile/`


=== "Shell"

    ```shell
    curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/2/accounts/profile/
    ```

=== "Python"

    ```python
    import requests

    requests.get("https://exist.io/api/2/accounts/profile/",
        headers={'Authorization':'Token [YOUR_TOKEN]'})
    ```

### Response

Returns a user object in JSON:

```json
{
    "id": 1,
    "username": "josh",
    "first_name": "Josh",
    "last_name": "Sharp",
    "bio": "I made this thing you're using.",
    "url": "http://hellocode.co/",
    "avatar": "https://exist.io/static/media/avatars/josh_2.png",
    "timezone": "Australia/Melbourne",
    "local_time": "2020-07-31T22:33:49.359+10:00",
    "private": false,
    "imperial_units": false,
    "imperial_distance": false,
    "imperial_weight": false,
    "imperial_energy": false,
    "imperial_liquid": false,
    "imperial_temperature": false,
}
```
