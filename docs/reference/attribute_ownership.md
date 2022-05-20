---
title: Attribute ownership
---

!!! note

    **This section only applies for OAuth2 clients.**

Only one service may have ownership of any user attribute at any given time. Services must acquire ownership of an attribute to be able to write data for this attribute, and can release ownership if needed, for example if the user closes their account with this service or chooses to turn off certain attributes. Users can change which services own their attributes from [their attributes page](https://exist.io/account/attributes/).

The process of acquiring an attribute only needs to happen once, not each time the attribute is updated, although you may get an error if you attempt to update an attribute you don't own. Read more about approaches for handling this [in the guide](/guide/write_client/).

## Acquire attributes

Acquiring an attribute makes your client the owner of the attribute. This allows a service to update attribute data for these attributes. Users do not have to approve this (mostly because this would be cumbersome) so please explain/confirm this behaviour with users within your own application. Acquiring a templated attribute the user doesn't have yet **will create this attribute** and give you ownership.

### Request

`POST /api/2/attributes/acquire/`


=== "Shell"

    ```shell
    curl "https://exist.io/api/2/attributes/acquire/" -H "Content-Type: application/json" -H "Authorization: Bearer [your_token]" -X POST -d '[{"name":"mood"}, {"name":"mood_note"}]'
    ```

=== "Python"

    ```python
    import requests, json

    url = 'https://exist.io/api/2/attributes/acquire/'

    attributes = [{"name":"mood"}, {"name":"mood_note"}]

    response = requests.post(url, headers={'Authorization':'Bearer [your_token]', 'Content-Type':'application/json'},
        data=json.dumps(attributes))
    ```

### Parameters

Clients must send a JSON-encoded array of objects. Maximum is 35 objects in the array.

Name  | Description
------|--------
`name` | The attribute name, eg. `mood_note`
`manual` | Boolean flag to set this attribute as manually updated or not
`allow_user_defined` | Boolean flag in query parameters to allow non-templated attributes
`success_objects` | Boolean flag in query parameters which, if set, provides a full attribute object for each successful acquisition

To learn more about when you might want to use `allow_user_defined`, read about this topic [in the guide](/guide/write_client/).

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

To list all the attributes your client currently owns, see ["Attributes" under Reading data](/reference/attributes/#get-a-users-attributes) and note the `owned` flag.


