---
title: Correlations
---

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

