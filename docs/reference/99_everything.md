---
title: Exist API reference
search: true
---


# Users

## Get profile stub for user

```shell
curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/1/users/\$self/profile/
```

```python
import requests

requests.get("https://exist.io/api/1/users/$self/profile/",
    headers={'Authorization':'Token [YOUR_TOKEN]'})
```

> Returns a user object in JSON:

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
    "attributes": [ ]
}
```

Returns an overview of the user's personal details.

### Request

`GET /api/1/users/:username/profile/`


## Get current overview for user

```shell
curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/1/users/\$self/today/
```

```python
import requests

requests.get("https://exist.io/api/1/users/$self/today/",
    headers={'Authorization':'Token [YOUR_TOKEN]'})
```

> Returns a user object in JSON:

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
    "attributes": [
        {
            "group": "activity",
            "label": "Activity",
            "priority": 1, 
            "items": [
                {
                    "attribute": "steps", 
                    "label": "Steps", 
                    "value": 258, 
                    "service": "Fitbit", 
                    "priority": 1, 
                    "private": false,
                    "value_type": 0,
                    "value_type_description": "Integer"
                }, 
                {
                    "attribute": "floors", 
                    "label": "Floors", 
                    "value": 2, 
                    "service": "Fitbit", 
                    "priority": 2, 
                    "private": false,
                    "value_type": 0,
                    "value_type_description": "Integer"
                }
            ]
        }
    ]
}
```

Returns an overview of the user's personal details, and their grouped attributes containing current values.
This is analogous to the "Today" tiles in the dashboard.

When requesting the currently authenticated user, you will receive all attributes. For other users,
this will return a user stub.

### Request

`GET /api/1/users/:username/today/`

# Attributes

## Get multiple attributes

```shell
curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/1/users/\$self/attributes/
```

```python
import requests

requests.get("https://exist.io/api/1/users/$self/attributes/",
    headers={'Authorization':'Token [YOUR_TOKEN]'})
```

> Returns a list of attribute objects each with a list of values, by default all attributes:

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

Return the user's attributes, all attributes with the last week's values by default. Currently this method is only allowed for the authenticated user.

If you need to get a specific date range or combination of attribute values, you may combine the optional parameters to do so.

### Request

`GET /api/1/users/:username/attributes/`

### Parameters

Name  | Description
------|--------
`limit` | Number of values to return, starting with the newest date. Optional, max is 31.
`attributes` | Optional comma-separated list of attributes, e.g. `mood,mood_note`
`groups`     | Optional comma-separated list of groups to filter attributes by, e.g. `mood,health`
`date_max`   | Optional date specifying the newest value that can be returned, in format `YYYY-mm-dd`.

## Get a specific attribute

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

Returns a paged list of all values from an attribute.

### Request

`GET /api/1/users/:username/attributes/:attribute/`

### Parameters

Name  | Description
------|--------
`limit` | Number of values to return per page. Optional, max is 100.
`page`  | Page index. Optional, default is 1.
`date_max` | Most recent date (inclusive) of results to be returned, in format `YYYY-mm-dd`. Optional.


# Insights

## Get all insights

```shell
curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/1/users/\$self/insights/
```

```python
import requests

requests.get("https://exist.io/api/1/users/$self/insights/",
    headers={'Authorization':'Token [YOUR_TOKEN]'})
```

> Returns a JSON object containing a paged array:

```json
{
    "count": 740, 
    "next": "https://exist.io/api/1/users/josh/insights/?page=2", 
    "previous": null, 
    "results": [
        {
            "created": "2015-05-09T01:00:02Z", 
            "target_date": "2015-05-08", 
            "html": "<div class=\"secondary\">Friday night: Shortest sleep for 3 days</div>...", 
            "text": "Friday night: Shortest sleep for 3 days\r\n", 
            "type": {
                "name": "sleep_worst_since_x", 
                "period": 1, 
                "priority": 2, 
                "attribute": {
                    "name": "sleep", 
                    "label": "Time asleep", 
                    "group": {
                        "name": "sleep", 
                        "priority": 3
                    }, 
                    "priority": 2
                }
            }
        }, 
        {
            "created": "2015-05-08T21:00:03Z", 
            "target_date": null, 
            "html": "<div class=\"number\">09:38</div>...",
            "text": "09:38 average time asleep...",
            "type": {
                "name": "sleep_end_average_week", 
                "period": 7, 
                "priority": 3, 
                "attribute": {
                    "name": "sleep_end", 
                    "label": "Wake time", 
                    "group": {
                        "name": "sleep", 
                        "priority": 3
                    }, 
                    "priority": 4
                }
            }
        }
    ]
}
```

Returns a paged list of user's insights. Only available for the currently authenticated user.

### Request

`GET /api/1/users/:username/insights/`

### Parameters

Name  | Description
------|--------
`limit` | Number of values to return per page, starting with today. Optional, max is 100.
`page`  | Page index. Optional, default is 1.
`date_min` | Oldest date (inclusive) of results to be returned, in format `YYYY-mm-dd`. Optional.
`date_max` | Most recent date (inclusive) of results to be returned, in format `YYYY-mm-dd`. Optional.

## Get all insights for attribute

```shell
curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/1/users/\$self/insights/attribute/sleep/
```

```python
import requests

requests.get("https://exist.io/api/1/users/$self/insights/attribute/sleep/",
    headers={'Authorization':'Token [YOUR_TOKEN]'})
```

> Returns a JSON object containing a paged array:

```json
{
    "count": 220, 
    "next": "https://exist.io/api/1/users/josh/insights/attribute/sleep/?page=2", 
    "previous": null, 
    "results": [
        {
            "created": "2015-05-09T01:00:02Z", 
            "target_date": "2015-05-08", 
            "html": "<div class=\"secondary\">Friday night: Shortest sleep for 3 days</div>...", 
            "text": "Friday night: Shortest sleep for 3 days...",
            "type": {
                "name": "sleep_worst_since_x", 
                "period": 1, 
                "priority": 2, 
                "attribute": {
                    "name": "sleep", 
                    "label": "Time asleep", 
                    "group": {
                        "name": "sleep", 
                        "priority": 3
                    }, 
                    "priority": 2
                }
            }
        } 
    ]
}
```

Returns a paged list of user's insights for a specific attribute. Only available for the currently authenticated user.

### Request

`GET /api/1/users/:username/insights/attribute/:attribute/`

### Parameters

Name  | Description
------|--------
`limit` | Number of values to return per page, starting with today. Optional, max is 100.
`page`  | Page index. Optional, default is 1.
`date_min` | Oldest date (inclusive) of results to be returned, in format `YYYY-mm-dd`. Optional.
`date_max` | Most recent date (inclusive) of results to be returned, in format `YYYY-mm-dd`. Optional.

# Averages

## Get current averages

```shell
curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/1/users/\$self/averages/
```

```python
import requests

requests.get("https://exist.io/api/1/users/$self/averages/",
    headers={'Authorization':'Token [YOUR_TOKEN]'})
```

> Returns a JSON array:

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

Returns the most recent average values for each attribute. Only available for the currently authenticated user.

### Request

`GET /api/1/users/:username/averages/`

## Get all averages for attribute

```shell
curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/1/users/\$self/averages/attribute/steps/
```

```python
import requests

requests.get("https://exist.io/api/1/users/$self/averages/attribute/steps/",
    headers={'Authorization':'Token [YOUR_TOKEN]'})
```

> Returns a JSON object containing a paged array of results:

```json
{
    "count": 11, 
    "next": null, 
    "previous": null, 
    "results": [
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
            "attribute": "steps", 
            "date": "2020-03-30", 
            "overall": 4062.0, 
            "monday": 4057.0, 
            "tuesday": 6618.0, 
            "wednesday": 3610.0, 
            "thursday": 3923.0, 
            "friday": 4063.0, 
            "saturday": 3636.0, 
            "sunday": 3904.0
        }
    ]
}
```

Returns a paged list of average values for an attribute.

### Request

`GET /api/1/users/:username/averages/attribute/:attribute/`

### Parameters

Name  | Description
------|--------
`limit` | Number of values to return per page, starting with today. Optional, max is 100.
`page`  | Page index. Optional, default is 1.
`date_min` | Oldest date (inclusive) of results to be returned, in format `YYYY-mm-dd`. Optional.
`date_max` | Most recent date (inclusive) of results to be returned, in format `YYYY-mm-dd`. Optional.


# Correlations

## Get all correlations for attribute

```shell
curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/1/users/\$self/correlations/attribute/steps/
```

```python
import requests

requests.get("https://exist.io/api/1/users/$self/correlations/attribute/steps/",
    headers={'Authorization':'Token [YOUR_TOKEN]'})
```

> Returns a JSON object containing an array of results:

```json
{
    "count": 479, 
    "next": "https://exist.io/api/1/users/josh/correlations/attribute/steps/?page=2", 
    "previous": null, 
    "results": [
        {
            "date": "2015-05-11", 
            "period": 90, 
            "attribute": "steps", 
            "attribute2": "steps_distance", 
            "value": 0.999735821732415,
            "p": 5.43055953485446e-146,
            "percentage": 43.254411196924906,
            "stars": 2,
            "second_person": "You get more steps when you spend more time active.",
            "second_person_elements": [
                "you get more steps",
                "when",
                "you spend more time active"
            ],
            "attribute_category": null,
            "strength_description": "Quite often go together",
            "stars_description": "Certain to be related",
            "description": null,
            "occurrence": null,
            "rating": null
        }, 
        {
            "date": "2019-09-01",
            "period": 365,
            "attribute": "steps",
            "attribute2": "floors",
            "value": 0.389185953425964,
            "p": 1.2396329763201e-11,
            "percentage": 38.918595342596404,
            "stars": 5,
            "second_person": "You get more steps when you climb more floors.",
            "second_person_elements": [
                "you get more steps",
                "when",
                "you climb more floors"
            ],
            "attribute_category": null,
            "strength_description": "Quite often go together",
            "stars_description": "Certain to be related",
            "description": "If you're out and about and walking around more, you're also more likely to be climbing floors. Especially if your home or workplace is multi-level.",
            "occurrence": "Common",
            "rating": {
                "positive": false,
                "rating": "Too obvious"
            }
        }
    ]
}
```



Returns a paged list of all correlations generated relating to this attribute, ordered by date.
Correlations may appear more than once, with different results, as this relationship changes over time.


### Request

`GET /api/1/users/:username/correlations/attribute/:attribute/`

### Parameters

Name  | Description
------|--------
`limit` | Number of values to return per page, starting with today. Optional, max is 100.
`page`  | Page index. Optional, default is 1.
`date_min` | Oldest date (inclusive) of results to be returned, in format `YYYY-mm-dd`. Optional.
`date_max` | Most recent date (inclusive) of results to be returned, in format `YYYY-mm-dd`. Optional.
`latest` | Set this to `true` to return only the most recently generated batch of correlations. Use this on its own without `date_min` and `date_max`.

## Get the strongest correlations for all attributes

```shell
curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/1/users/\$self/correlations/strongest/
```

```python
import requests

requests.get("https://exist.io/api/1/users/$self/correlations/strongest/",
    headers={'Authorization':'Token [YOUR_TOKEN]'})
```

> Returns a JSON array:

```json
{
    "count": 479, 
    "next": "https://exist.io/api/1/users/josh/correlations/attribute/steps/?page=2", 
    "previous": null, 
    "results": [
        {
            "date": "2015-05-11", 
            "period": 90, 
            "attribute": "steps", 
            "attribute2": "steps_distance", 
            "value": 0.999735821732415,
            "p": 5.43055953485446e-146,
            "percentage": 43.254411196924906,
            "stars": 2,
            "second_person": "You get more steps when you spend more time active.",
            "second_person_elements": [
                "you get more steps",
                "when",
                "you spend more time active"
            ],
            "attribute_category": null,
            "strength_description": "Quite often go together",
            "stars_description": "Certain to be related",
            "description": null,
            "occurrence": null,
            "rating": null
        }, 
        {
            "date": "2019-09-01",
            "period": 365,
            "attribute": "steps",
            "attribute2": "floors",
            "value": 0.389185953425964,
            "p": 1.2396329763201e-11,
            "percentage": 38.918595342596404,
            "stars": 5,
            "second_person": "You get more steps when you climb more floors.",
            "second_person_elements": [
                "you get more steps",
                "when",
                "you climb more floors"
            ],
            "attribute_category": null,
            "strength_description": "Quite often go together",
            "stars_description": "Certain to be related",
            "description": "If you're out and about and walking around more, you're also more likely to be climbing floors. Especially if your home or workplace is multi-level.",
            "occurrence": "Common",
            "rating": {
                "positive": false,
                "rating": "Too obvious"
            }
        }
    ]
}
```



Returns a list of the user's strongest correlations across all attributes.


### Request

`GET /api/1/users/:username/correlations/strongest/`


# Attribute ownership

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

# Updating attributes 

## Overview

If you jumped straight here looking for how to send data for your own attributes, here's a recap of the steps you may have missed:

1. Make sure you have an [OAuth2 client](#authentication-overview) set up
2. Make sure you've listed all attributes you'd like to be able to write to (set this from your [app management page](https://exist.io/account/apps/))
3. Make sure you have [acquired ownership](#attribute-ownership) of the attribute before updating (just once, not each time)
4. Now you can update.

## Updating attribute values

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

> Returns a JSON object containing successful and failed updates:

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

**This section only applies for OAuth2 clients.**

This endpoint allows services to update attribute data for the authenticated user. Data is stored on a single day granularity, so each update contains `name`, `date`, and `value`. Make sure the date is local to the user — though you do not have to worry about timezones directly, if you are using your local time instead of the user's local time, you may be a day ahead or behind!

Valid values are described by the attribute's `value_type` and `value_type_description` fields. However, values are only validated broadly by type and so care must be taken to send correct data. Do not rely on Exist to validate your values beyond
enforcing the correct type.

Check value types for each attribute in [list of supported attributes](#list-of-attributes).

### Request

`POST /api/1/attributes/update/`

### Parameters

Clients must send a JSON-encoded array of objects containing a `name`, `date`, and `value`. The array must not exceed 35 objects in length.

Name  | Description
------|--------
`name` | The attribute name, eg. `mood_note`
`date` | String of format `YYYY-mm-dd`
`value` | A valid value for this attribute type: string, integer, or float


### Response

Returns `200 OK` if all attributes were processed successfully, or `202 Accepted` if some attributes failed. The content is a JSON object containing `success` and `failed` arrays, where each item in the array is an attribute sent in the prior request. Failed attributes get `error` and `error_code` fields added. 


# Updating custom tags

There are two valid approaches here for sending tags:
 
- managing all tags by acquiring the `custom` attribute,
- using the `append` scope to write individual tags for a day, without being able to modify other tags.

Both are documented below.

## Validating tag names

```python
import re

def validate_tag(raw_tag):
    # for python 3: remove the 'u'
    regex = re.compile(ur'[\W]', re.UNICODE)
    
    tag = raw_tag.replace(' ', '_')
    tag = regex.sub('', tag)
    tag = tag.strip('_')

    if len(tag) == 0:
        raise Exception("Tag '%s' contains too many invalid characters" % raw_tag)

    return tag
```

Tags must be "slugs" that can contain only valid unicode letters, numbers, and underscores. Convert spaces to underscores, remove invalid characters, and make sure to trim leading and trailing whitespace. An example validation method is provided here in Python.   

## Acquiring and managing all tags

If you wish to manage all tags for a user, which you might do if you were writing an alternative "tag client" app, for example, you can handle this similarly to updating any other attribute. In this case, you acquire and update the `custom` attribute with a string of comma-separated tags. 

This is a short-hand means for updating each tag attribute to have a value of `1` for the day. For example, sending a string value of `tired, sex` means these tags will have a value of `1` set, and all others will be set to `0`. The zeroing of other tags gives users an easy way to remove tags for a day, but this means care must be taken to not overwrite tags sent via the [append](#appending-specific-tags) method.

### To get existing tags for a day

If you are managing all tags and don't want to overwrite existing tags, you should:

1. Get a list of all custom tag attributes
2. Get the names of all attributes with a value of `1` for the day
3. Concatenate these into one string, joined by commas
4. Use this string as the starting value for your updates


### To update tags

1. Make sure your client app has permission to acquire the `custom` attribute (set this from your [app management page](https://exist.io/account/apps/))
2. Make sure you have [acquired ownership](#attribute-ownership) of the `custom` attribute before updating (just once, not each time)
3. Send a string value for `custom` containing a list of tags.  Tags must be sent as a comma-separated list of [validated tag names](#validating-tag-names). 

To read and present tags, you can read about [custom tags values](#custom-tags).


```shell
curl https://exist.io/api/1/attributes/update/ -H "Content-Type: application/json" -H "Authorization: Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737" -X POST -d '[{"name":"custom", "date":"2015-05-20", "value":"tired, bike_ride, vitamin_d"}]'
```

```python
import requests, json

url = 'https://exist.io/api/1/attributes/update/'

attributes = [{"name":"custom", "date":"2015-05-20", "value":"tired, bike_ride, vitamin_d"}]

response = requests.post(url, headers={'Authorization':'Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737'},
    data=json.dumps(attributes))
```

> Returns a JSON object containing successful and failed updates:

```json
{ "success": [ 
    { "name":"custom",
      "date":"2015-05-20",
      "value":"tired, bike_ride, vitamin_d"
    }
  ],
  "failed": []
}
```

### Request

`POST /api/1/attributes/update/`

### Parameters

Clients must send a JSON-encoded array of objects containing a `name`, `date`, and `value`.

Name  | Description
------|--------
`name` | `custom`
`date` | String of format `YYYY-mm-dd`
`value` | A string of comma- (and optional space-) separated tags in the form `bike_ride, meditation, sex` 


### Response

Returns `200 OK` if all attributes were processed successfully, or `202 Accepted` if some attributes failed. The content is a JSON object containing `success` and `failed` arrays, where each item in the array is an attribute sent in the prior request. Failed attributes get `error` and `error_code` fields added. 


## Appending specific tags

This alternative method allows you to append tags to a day without ownership of the `custom` attribute. By using the `append` endpoint, you can send a list of tags, with optional dates, and have these tags added to the user's other tag selections for that day. This works well for event-based use cases, where a particular event might trigger a particular tag. You can easily send a single tag name, without date, to add it to the current day in the user's time zone.

To use this endpoint, your client needs to have asked for the `append` permission with the current user. When following [the OAuth2 authentication process](#authorisation-flow), substitute `scope=append`, or `scope=read+write+append` if you also need write access.  


```shell
curl https://exist.io/api/1/attributes/custom/append/ -H "Content-Type: application/json" -H "Authorization: Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737" -X POST -d '[{"value":"bike_ride", "date":"2017-05-20"}]'
```

```python
import requests, json

url = 'https://exist.io/api/1/attributes/custom/append/'

tags = [{"value":"bike_ride", "date":"2017-05-20"}]
# alternatively you could send a single value for today
tags = {"value":"bike_ride"}

response = requests.post(url, headers={'Authorization':'Bearer 96524c5ca126d87eb18ee7eff408ca0e71e94737'},
    data=json.dumps(tags))
```

> Returns a JSON object containing successful and failed updates:

```json
{ "success": [ 
    { "date":"2015-05-20",
      "value":"bike_ride"
    }
  ],
  "failed": []
}
```

### Request

`POST /api/1/attributes/custom/append/`

### Parameters

Clients must send either:

- a JSON-encoded array of objects containing a `value` and optional `date`
- a single JSON object with a `value` and optional `date`

Name  | Description
------|--------
`value` | The [validated name](#validating-tags) of the tag to append
`date` | Optional string of format `YYYY-mm-dd`. If omitted, current day is assumed
 

### Response

Returns `200 OK` if all tags were processed successfully, or `202 Accepted` if some attributes failed. The content is a JSON object containing `success` and `failed` arrays, where each item in the array is an attribute sent in the prior request. Failed attributes get `error` and `error_code` fields added. 


# API roadmap

## Changelog

* **2017-09-27:** Custom tags now available from the API
* **2015-12-01:** OAuth2 clients are available for all users
* **2015-10-05:** Removed separate value fields in insights, added rendered text field
* **2015-07-09:** Introduced attribute validation and started validating `mood` values
* **2015-05-13:** Added date filtering for insights, correlations, averages, and attribute data.

