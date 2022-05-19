---
title: Users
---

## Get profile for user

Returns some basic details and personal preferences for the authenticated user. No specific scope is required.

### Request

`GET /api/2/accounts/profile/`


=== "Shell"

    ```shell
    curl "https://exist.io/api/2/accounts/profile/" -H "Authorization: Bearer [YOUR_TOKEN]" 
    ```

=== "Python"

    ```python
    import requests

    requests.get("https://exist.io/api/2/accounts/profile/", 
                 headers={'Authorization':'Bearer [YOUR_TOKEN]'})
    ```

### Response

Returns a user object in JSON:

```json
{
  "username": "josh",
  "first_name": "Josh",
  "last_name": "Sharp",
  "avatar": "https://exist.io/media/avatars/josh.png",
  "timezone": "Australia/Sydney",
  "local_time": "2022-05-16T16:01:37.587301+10:00",
  "imperial_distance": false,
  "imperial_weight": false,
  "imperial_energy": false,
  "imperial_liquid": false,
  "imperial_temperature": false,
  "trial": false,
  "delinquent": false
}
```
