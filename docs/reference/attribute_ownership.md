---
title: Attribute ownership
---

!!! note

    **This section only applies for OAuth2 clients.**

Only one service may have ownership of any user attribute at any given time. Services must acquire ownership of an attribute to be able to write data for this attribute, and can release ownership if needed, for example if the user closes their account with this service or chooses to turn off certain attributes.

The process of acquiring an attribute only needs to happen once, not each time the attribute is updated, although you may get an error if you attempt to update an attribute you don't own.

## Acquire attributes

```shell
curl https://exist.io/api/1/attributes/acquire/ -H "Content-Type: application/json" -H "Authorization: Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737" -X POST -d '[{"name":"mood", "active":true}, {"name":"mood_note", "active":true}]'
```

```python
import requests, json

url = 'https://exist.io/api/1/attributes/acquire/'

attributes = [{"name":"mood", "active":True}, {"name":"mood_note", "active":True}]

response = requests.post(url, headers={'Authorization':'Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737'},
    data=json.dumps(attributes))
```

> Returns JSON and a status code of `202 Accepted` if some attributes failed (just for example, the above is correct)

```json
{ "success": [ 
    { "name":"mood_note",
      "active":"true"
    }
  ],
  "failed": [
    { "name":"mood",
      "error_code":"missing_field",
      "error":"Object at index 0 missing field(s) 'active'"
    }
  ]
}
```

Allows a service to update attribute data for these attributes. Users do not have to approve this (mostly because this would be cumbersome) so please explain/confirm this behaviour with users within your own application.

### Request

`POST /api/1/attributes/acquire/`

### Parameters

Clients must send a JSON-encoded array of objects, where each object contains a `name` string and an `active` boolean. Setting `active` to `false` indicates you'd like to deactivate this attribute without giving up ownership.

Name  | Description
------|--------
`name` | The attribute name, eg. `mood_note`
`active` | `true` or `false` to set this attribute to active or inactive
`private` | Optional `true` or `false` to change the privacy status of this attribute. Please notify users if you are making previously private attributes public and only do this with good reason.

### Response

Returns `200 OK` if all attributes were processed successfully, or `202 Accepted` if some attributes failed. The content is a JSON object containing `success` and `failed` arrays, where each item in the array is an attribute sent in the prior request. Failed attributes get `error` and `error_code` fields added. 

## Release attributes

```shell
curl https://exist.io/api/1/attributes/release/ -H "Content-Type: application/json" -H "Authorization: Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737" -X POST -d '[{"name":"mood"}, {"name":"mood_note"}]'
```

```python
import requests, json

url = 'https://exist.io/api/1/attributes/release/'

attributes = [{"name":"mood"}, {"name":"mood_note"}]

response = requests.post(url, headers={'Authorization':'Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737'}, 
    data=json.dumps(attributes))
```

> Returns JSON and a status code of `202 Accepted` if some attributes failed (just for example, the above is correct)

```json
{ "success": [ 
    { "name":"mood_note" }
  ],
  "failed": [
    { "name":"mood",
      "error_code":"unauthorised",
      "error":"Attribute 'mood' does not belong to this service"
    }
  ]
}
```

Do this to release your ownership of any attributes. The attributes' ownership will pass to another service, if the user has another supplied that has indicated it can handle this attribute, or otherwise become inactive.

### Request

`POST /api/1/attributes/release/`

### Parameters

Clients must send a JSON-encoded array of objects, where each object contains a `name` string. The objects may seem superfluous but this is to be consistent with the `acquire` endpoint.

Name  | Description
------|--------
`name` | The attribute name, eg. `mood_note`

### Response

Returns `200 OK` if all attributes were processed successfully, or `202 Accepted` if some attributes failed. The content is a JSON object containing `success` and `failed` arrays, where each item in the array is an attribute sent in the prior request. Failed attributes get `error` and `error_code` fields added. 


## Create new attributes

TODO

## List owned attributes

```shell
curl https://exist.io/api/1/attributes/owned/ -H "Authorization: Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737"
```

```python
import requests

url = 'https://exist.io/api/1/attributes/owned/'

response = requests.get(url, headers={'Authorization':'Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737'})
```

> Returns a JSON array of attributes for the authenticated user and owned by this service:

```json
[
    {
        "attribute": "steps", 
        "label": "Steps", 
        "value": null, 
        "service": "fitbit", 
        "priority": 1, 
        "private": false, 
        "value_type": 0, 
        "value_type_description": "Integer"
    }, 
    {
        "attribute": "steps_active_min", 
        "label": "Active minutes", 
        "value": null, 
        "service": "fitbit", 
        "priority": 2, 
        "private": false, 
        "value_type": 0, 
        "value_type_description": "Integer"
    }
]
```

This is a convenience endpoint to list all attributes for the authenticated user currently owned by this service.

### Request

`GET /api/1/attributes/owned/`

