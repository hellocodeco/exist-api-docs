---
title: Attributes
---


## Get attribute templates

## Get attributes

## Get attributes with values


Return the user's attributes, all attributes with the last week's values by default.

If you need to get a specific date range or combination of attribute values, you may combine the optional parameters to do so.


### Request

`GET /api/2/attributes/with-values/`

### Parameters

Name  | Description
------|--------
`limit` | Number of values to return, starting with the newest date. Optional, max is 31.
`attributes` | Optional comma-separated list of attributes, e.g. `mood,mood_note`
`groups`     | Optional comma-separated list of groups to filter attributes by, e.g. `mood,health`
`date_max`   | Optional date specifying the newest value that can be returned, in format `YYYY-mm-dd`.


```shell
curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/1/users/\$self/attributes/
```

```python
import requests

requests.get("https://exist.io/api/1/users/$self/attributes/",
    headers={'Authorization':'Token [YOUR_TOKEN]'})
```

### Response

Returns a list of attribute objects each with a list of values, by default all attributes:

```json
[
    {
        "attribute": "steps",
        "label": "Steps",
        "group": {
            "name": "activity",
            "label": "Activity",
            "priority": 1
        },
        "service": "Fitbit",
        "private": false,
        "values": [
            {
                "value": 331,
                "date": "2014-08-01"
            },
            {
                "value": 5872,
                "date": "2014-07-31"
            },
            {
                "value": 2832,
                "date": "2014-07-30"
            },
            {
                "value": 5153,
                "date": "2014-07-29"
            },
            {
                "value": 4354,
                "date": "2014-07-28"
            },
            {
                "value": 7132,
                "date": "2014-07-27"
            },
            {
                "value": 4144,
                "date": "2014-07-26"
            }
        ]
    },
    {
        "attribute": "floors",
        "label": "Floors",
        "group": {
            "name": "activity",
            "label": "Activity",
            "priority": 1
        },
        "service": "Fitbit",
        "private": false,
        "values": [ ]
    }
]
```

## Get a specific attribute

Returns a paged list of all values from an attribute.

```shell
curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/1/users/\$self/attributes/steps/
```

```python
import requests

requests.get("https://exist.io/api/1/users/$self/attributes/steps/",
    headers={'Authorization':'Token [YOUR_TOKEN]'})
```

> Returns a list of attribute values, omitting the attribute object itself:

```json
{
    "count": 655,
    "next": "https://exist.io/api/1/users/josh/attributes/steps/?page=2",
    "previous": null,
    "results": [
        {
            "value": 3783,
            "date": "2015-05-08"
        }, {
            "value": 6177,
            "date": "2015-05-07"
        }, {
            "value": 7811,
            "date": "2015-05-06"
        }, {
            "value": 6632,
            "date": "2015-05-05"
        }, {
            "value": 6014,
            "date": "2015-05-04"
        }, {
            "value": 4376,
            "date": "2015-05-03"
        }, {
            "value": 8671,
            "date": "2015-05-02"
        }
    ]
}

```


### Request

`GET /api/1/users/:username/attributes/:attribute/`

### Parameters

Name  | Description
------|--------
`limit` | Number of values to return per page. Optional, max is 100.
`page`  | Page index. Optional, default is 1.
`date_max` | Most recent date (inclusive) of results to be returned, in format `YYYY-mm-dd`. Optional.

