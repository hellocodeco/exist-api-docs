---
title: Creating attributes
---

!!! note

    **This section only applies for OAuth2 clients.**



## Create new attributes

Write scopes allow your client to create new attributes for a user in the matching scopes. These attributes can be templated, or entirely custom. The `manual_write` scope allows creating manual attributes in any group. If you choose not to use a template, an entirely custom attribute can be defined, with its own label and value type. It must belong to an existing group.

### Request

`POST /api/2/attributes/create/`

=== "Shell"
    ```shell
    curl -X POST "https://exist.io/api/2/attributes/create/?success_objects=1" -H "Authorization: Bearer [token]" -H "Content-Type: application/json" -d '[{"template":"facebook_reactions"}]'
    ```

=== "Python"

    ```python
    import requests, json

    url = "https://exist.io/api/2/attributes/create/"

    attributes = [{'template':'facebook_reactions'}]

    response = requests.post(url, 
        headers={'Authorization':'Bearer [your_token]', 
                 'Content-Type':'application/json'},
        params={'success_objects':'1'}, 
        data=json.dumps(attributes))
    ```

### Parameters

Clients should send a JSON array of objects where either the `template` field or the full set of `label`, `group`, and `value_type` are included. Providing `manual` is also optional. The array can contain a maximum of 35 objects.

Name      | Description
----------|-------------
**For templated attributes** ||
`template` | The name of the [attribute template](/reference/object_types/#list-of-attribute-templates) to use 
 **For custom attributes**||
`label`    | String user-facing title for the new attribute
`group`    | String name of the group this attribute will belong to
`value_type` | Integer [value type](/reference/object_types/#attribute-value-types) of the attribute
**Optional flags** ||
`manual`   | Optional boolean defining if the attribute is manually tracked, defaulting to true
`success_objects` | Optional boolean which, if set in query params, returns full attribute objects on success rather than the contents of your request


### Response

Returns a JSON object with `success` and `failed` fields containing your requested attributes. Returns `200 OK` if all attributes were successful or `202 Created` if some failed. Each object in the `failed` array will contain an `error` description and `error_code` string.

```json
{ "success": [
    { "template":"facebook_reactions",
      "name":"facebook_reactions",
      "label":"Facebook reactions",
      "group":{
        "name":"social",
        "label":"Social",
        "priority":7
      },
      "service":{
        "name":"exist_for_ios",
        "label":"Exist for iOS"
      },
      "active":true,
      "priority":2,
      "manual":true,
      "value_type":0,
      "value_type_description":"Integer",
      "available_services":[]
    }
  ],
  "failed":[]
}
```
