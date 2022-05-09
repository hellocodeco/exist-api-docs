---
title: Updating values
---


!!! note

    **This section only applies for OAuth2 clients.**

## Overview

If you jumped straight here looking for how to send data for your own attributes, here's a recap of the steps you may have missed:

1. Make sure you have an [OAuth2 client](#) set up
2. Make sure your token has the correct write scope (set this from your [app management page](https://exist.io/account/apps/))
3. Make sure you have either [created the attribute](#) or [acquired ownership](#) of the attribute before updating (just once, not each time)
4. Now you can update.

## Updating attribute values


This endpoint allows services to update attribute data for the authenticated user. Data is stored on a single day granularity, so each update contains `name`, `date`, and `value`. Make sure the date is local to the user — though you do not have to worry about timezones directly, if you are using your local time instead of the user's local time, you may be a day ahead or behind!

Valid values are described by the attribute's `value_type` and `value_type_description` fields. However, values are only validated broadly by type and so care must be taken to send correct data. Do not rely on Exist to validate your values beyond
enforcing the correct type.

Check value types for each attribute in [list of supported attributes](#list-of-attributes).

### Request

`POST /api/1/attributes/update/`

### Parameters

Clients must send a JSON-encoded array of objects containing a `name`, `date`, and `value`. 

**The array must not exceed 35 objects in length.**

Name  | Description
------|--------
`name` | The attribute name, eg. `mood_note`
`date` | String of format `YYYY-mm-dd`
`value` | A valid value for this attribute type: string, integer, or float




```shell
curl https://exist.io/api/1/attributes/update/ -H "Content-Type: application/json" -H "Authorization: Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737" -X POST -d '[{"name":"mood", "date":"2015-05-20", "value":5}, {"name":"mood_note", "date":"2015-05-20", "value":"Great day playing with the Exist API"}]'
```

```python
import requests, json

url = 'https://exist.io/api/1/attributes/update/'

attributes = [{"name":"mood", "date":"2015-05-20", "value":5}, {"name":"mood_note", "date":"2015-05-20", "value":"Great day playing with the Exist API"}]

response = requests.post(url, headers={'Authorization':'Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737'},
    data=json.dumps(attributes))
```


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

## Incrementing attribute values

TODO
