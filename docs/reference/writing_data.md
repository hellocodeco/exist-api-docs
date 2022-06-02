---
title: Updating values
---


!!! note

    **This section only applies for OAuth2 clients.**

## Overview

If you jumped straight here looking for how to send data for your own attributes, here's a recap of the steps you may have missed:

1. Make sure you have an [OAuth2 client](/reference/authentication/oauth2/) set up
2. Make sure your token has the [correct write scope](/reference/authentication/oauth2/#scopes) (set this from your [app management page](https://exist.io/account/apps/))
3. Make sure you have either [created the attribute](/reference/attribute_ownership/#create-new-attributes) or [acquired ownership](/reference/attribute_ownership/#acquire-attributes) of the attribute before updating (just once, not each time)
4. Now you can update.

This process is covered in more detail [in the guide](/guide/write_client/).

## Update attribute values


This endpoint allows services to update attribute data for the authenticated user in batches of up to 35 updates in one call. This may include, for example, 35 days of data for one attribute, or one day's values across 35 attributes. Batching updates is strongly encouraged to avoid multiple calls to the API. 

Data is stored on a single day granularity, so each update contains `name`, `date`, and `value`. Make sure the date is local to the user — though you do not have to worry about timezones directly, if you are using your local time instead of the user's local time, you may be a day ahead or behind!

Valid values are described by the attribute's `value_type` and `value_type_description` fields. However, values are only validated broadly by type and so care must be taken to send correct data. Do not rely on Exist to validate your values beyond enforcing the correct type. This endpoint accepts total values for attributes and overwrites the previous value with the (validated) value you send. Non-null values cannot be set back to null, to prevent accidental data loss.

Check value types for each attribute in [list of supported attributes](#list-of-attributes).

### Request

`POST /api/2/attributes/update/`

=== "Shell"

    ```shell
    curl "https://exist.io/api/2/attributes/update/" -H "Content-Type: application/json" -H "Authorization: Bearer [token]" -X POST -d '[{"name":"mood", "date":"2022-05-20", "value":7}, {"name":"mood_note", "date":"2022-05-20", "value":"Great day playing with the Exist API"}]'
    ```

=== "Python"

    ```python
    import requests, json

    url = 'https://exist.io/api/2/attributes/update/'

    attributes = [{"name":"mood", "date":"2022-05-20", "value":7}, {"name":"mood_note", "date":"2022-05-20", "value":"Great day playing with the Exist API"}]

    response = requests.post(url, headers={'Authorization':'Bearer [token]'},
        data=json.dumps(attributes))
    ```


### Parameters

Clients must send a JSON-encoded array of objects containing a `name`, `date`, and `value`. 

**The array must not exceed 35 objects in length.**

Name  | Description
------|--------
`name` | The attribute name, eg. `mood_note`
`date` | String of format `YYYY-mm-dd`
`value` | A valid value for this attribute type: string, integer, or float


### Response

Returns `200 OK` if all attributes were processed successfully, or `202 Accepted` if some attributes failed. The content is a JSON object containing `success` and `failed` arrays, where each item in the array is an attribute sent in the prior request. Failed attributes get `error` and `error_code` fields added. 


```json
{ "success": [ 
    { "name":"mood_note",
      "date":"2015-05-20",
      "value":"Great day playing with the Exist API"
    }
  ],
  "failed": [
    { "name":"mood",
      "date":"2015-05-20",  
      "error_code":"missing_field",
      "error":"Object at index 0 missing field(s) 'value'"
    }
  ]
}
```

## Increment attribute values

While the update endpoint requires total values to be sent for a day, the increment endpoint allows sending deltas, or the change in value to be applied to the existing value. For example, to set a value `fridge_opens`, a count of how many times the refrigerator door is opened for a day, the update endpoint requires storing the total each day and sending this new total each time the door is opened. In contrast, the increment endpoint is happy to accept a `1` value for `fridge_opens`, removing the need for the client to remember the current state.

The attributes must already exist and be owned by your service.

Valid values are described by the attribute's `value_type` and `value_type_description` fields. However, values are only validated broadly by type and so care must be taken to send correct data. Do not rely on Exist to validate your values beyond enforcing the correct type. **This endpoint will not allow incrementing string, scale, or time of day attributes.**

Check value types for each attribute in [list of supported attributes](/reference/object_types/#list-of-attribute-templates).


### Request

`POST /api/2/attributes/increment/`

=== "Shell"

    ```shell
    curl "https://exist.io/api/2/attributes/increment/" -H "Content-Type: application/json" -H "Authorization: Bearer [token]" -X POST -d '[{"name":"steps", "date":"2022-05-20", "value":700}, {"name":"steps_active_min", "value":5}]'
    ```

=== "Python"

    ```python
    import requests, json

    url = 'https://exist.io/api/2/attributes/increment/'

    attributes = [{"name":"steps", "date":"2022-05-20", "value":700}, {"name":"steps_active_min", "value":5}]

    response = requests.post(url, headers={'Authorization':'Bearer [token]'},
        data=json.dumps(attributes))
    ```


### Parameters

Clients must send a JSON-encoded array of objects containing a `name`, `value`, and optional `date`. If date is omitted, the current date is asuumed. 

**The array must not exceed 35 objects in length.**

Name  | Description
------|--------
`name` | The attribute name, eg. `mood_note`
`date` | String of format `YYYY-mm-dd`, optional (defaults to today)
`value` | A valid value for this attribute type: integer or float


### Response

Returns `200 OK` if all attributes were processed successfully, or `202 Accepted` if some attributes failed. The content is a JSON object containing `success` and `failed` arrays, where each item in the array is an attribute sent in the prior request. Failed attributes get `error` and `error_code` fields added. Successful objects get a `current` field containing the new total for the related day.


```json
{ "success": [ 
    { "name":"steps",
      "value":700,
      "current":1700
    }
  ],
  "failed": [
    { "name":"steps_active_min",
      "value": 9.0,
      "error_code":"validation",
      "error":"Cannot apply a float to an integer type"
    }
  ]
}
```
