---
title: Attribute ownership
---

!!! note

    **This section only applies for OAuth2 clients.**

Only one service may have ownership of any user attribute at any given time. Services must acquire ownership of an attribute to be able to write data for this attribute, and can release ownership if needed, for example if the user closes their account with this service or chooses to turn off certain attributes. Users can change which services own their attributes from [their attributes page](https://exist.io/account/attributes/).

The process of acquiring an attribute only needs to happen once, not each time the attribute is updated, although you may get an error if you attempt to update an attribute you don't own. Read more about approaches for handling this [in the guide](/guide/write_client/).

## Acquire attributes

Acquiring an attribute makes your client the owner of the attribute. This allows a service to write values for this attribute. Users do not have to approve this (mostly because this would be cumbersome) so please explain/confirm this behaviour with users within your own application. Acquiring a templated attribute the user doesn't have yet **will create this attribute** and give you ownership.

### Request

`POST /api/2/attributes/acquire/`


=== "Shell"

    ```shell
    curl "https://exist.io/api/2/attributes/acquire/" -H "Content-Type: application/json" -H "Authorization: Bearer [your_token]" -X POST -d '[{"template":"mood"}, {"template":"mood_note"}]'
    ```

=== "Python"

    ```python
    import requests, json

    url = 'https://exist.io/api/2/attributes/acquire/'

    attributes = [{"template":"mood"}, {"template":"mood_note"}]

    response = requests.post(url, headers={'Authorization':'Bearer [your_token]', 'Content-Type':'application/json'},
        data=json.dumps(attributes))
    ```

### Parameters

Clients must send a JSON-encoded array of objects. Maximum is 35 objects in the array.

Name  | Description
------|--------
**For templated attributes** ||
`template` | The name of the [attribute template](/reference/object_types/#list-of-attribute-templates) to use 
**For acquiring any attribute** ||
`name` | The attribute name, eg. `mood_note`
**Other fields**||
`manual` | Boolean flag to set this attribute as manually updated or not
`success_objects` | Boolean flag in query parameters which, if set, provides a full attribute object in the response for each successful acquisition

To learn more about when you might want to `template` vs `name`, read about this topic [in the guide](/guide/write_client/).

### Response

Returns `200 OK` if all attributes were processed successfully, or `202 Accepted` if some attributes failed. The content is a JSON object containing `success` and `failed` arrays, where each item in the array is an attribute sent in the prior request. Failed attributes get `error` and `error_code` fields added. 


```json
{ "success": [ 
    { "name":"mood_note",
      "active":"true"
    }
  ],
  "failed": [
    { "title":"mood",
      "error_code":"missing_field",
      "error":"Object at index 0 missing field(s) 'name'"
    }
  ]
}
```



## Release attributes

Do this to release your ownership of any attributes. The attributes' ownership will pass to another service, if the user has another supplied that has indicated it can handle this attribute, or otherwise become inactive.

### Request

`POST /api/2/attributes/release/`

=== "Shell"
    ```shell
    curl "https://exist.io/api/2/attributes/release/" -H "Content-Type: application/json" -H "Authorization: Bearer [your_token]" -X POST -d '[{"name":"mood"}, {"name":"mood_note"}]'
    ```

=== "Python"

    ```python
    import requests, json

    url = 'https://exist.io/api/2/attributes/release/'

    attributes = [{"name":"mood"}, {"name":"mood_note"}]

    response = requests.post(url, headers={'Authorization':'Bearer [your_token]'}, 
        data=json.dumps(attributes))
    ```

### Parameters

Clients must send a JSON-encoded array of objects, where each object contains a `name` string. The objects may seem superfluous but this is to be consistent with the `acquire` endpoint. Maximum is 35 objects in the array.

Name  | Description
------|--------
`name` | The attribute name, eg. `mood_note`

### Response

Returns `200 OK` if all attributes were processed successfully, or `202 Accepted` if some attributes failed. The content is a JSON object containing `success` and `failed` arrays, where each item in the array is an attribute sent in the prior request. Failed attributes get `error` and `error_code` fields added. 

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

## List owned attributes

Retrieve a list of the user's attributes, without any values. Results are limited to what your service already owns.

### Request

`GET /api/2/attributes/owned/`

=== "Shell"

    ```shell
    curl "https://exist.io/api/2/attributes/owned/" -H "Authorization: Bearer [token]"
    ```

=== "Python"
    ```python
    import requests

    requests.get("https://exist.io/api/2/attributes/owned/",
                 headers = {'Authorization': 'Bearer [token]'})
    ```


### Parameters

All parameters are optional.

| Parameter | Description |
|-----------|-------------|
| `page` | Page index. Optional, default is 1. |
| `limit` | Integer defining how many results to a page |
| `groups`  | Comma-separated list of groups to filter by, e.g. `activity,workouts`|
| `attributes` | Comma-separated list of attributes to filter by |
| `exclude_custom` | Boolean flag, set to `true` to only show templated attributes |
| `manual` | Boolean flag, set to `true` to only show manual attributes or `false` to exclude |
| `include_inactive` | Boolean flag, set to `true` to include attributes with `active = False`, usually hidden |
| `include_low_priority` | Boolean flag, set to `true` to include attributes with a `priority` >= 10 |

### Response

A paged list of attribute objects belonging to this user. `available_services` shows the services a user has connected which have indicated they can provide data for this attribute.

```json
{
  "count": 34,
  "next": "https://exist.io/api/2/attributes/owned/?page=2",
  "previous": null,
  "results": [
    {
      "template": "steps",
      "name": "steps",
      "label": "Steps",
      "group": {
        "name": "activity",
        "label": "Activity",
        "priority": 1
      },
      "service": {
        "name": "test500",
        "label": "My Test Client"
      },
      "active": true,
      "priority": 1,
      "manual": false,
      "value_type": 0,
      "value_type_description": "Integer",
      "available_services": [
        {
          "name": "test500",
          "label": "My Test Client"
        },
        {
          "name": "oura",
          "label": "Oura"
        },
        {
          "name": "withings",
          "label": "Withings"
        }
      ]
    },
    // ...snip!...
  ]
}
```


