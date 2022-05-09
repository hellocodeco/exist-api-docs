---
title: Averages
---

## Get current averages


Returns the most recent average values for each attribute. 

### Request

`GET /api/1/users/:username/averages/`

```shell
curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/1/users/\$self/averages/
```

```python
import requests

requests.get("https://exist.io/api/1/users/$self/averages/",
    headers={'Authorization':'Token [YOUR_TOKEN]'})
```

### Response 

Returns a JSON array:

```json
[
    {
        "attribute": "steps", 
        "date": "2020-04-29", 
        "overall": 4174.0, 
        "monday": 4057.0, 
        "tuesday": 6614.0, 
        "wednesday": 4001.0, 
        "thursday": 3923.0, 
        "friday": 4528.0, 
        "saturday": 3649.0, 
        "sunday": 3904.0
    }, 
    {
        "attribute": "floors", 
        "date": "2020-04-29", 
        "overall": 13.0, 
        "monday": 13.0, 
        "tuesday": 16.0, 
        "wednesday": 14.0, 
        "thursday": 12.0, 
        "friday": 14.0, 
        "saturday": 13.0, 
        "sunday": 12.0
    }
]
```


### Examples

Using different parameters you could get all historical averages for an attribute rather
than the current averages for all attributes:

