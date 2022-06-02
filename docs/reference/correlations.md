---
title: Correlations
---

## Get all correlations

Returns a paged list of all correlations generated in the last week, optionally filtered by attribute, strength, or confidence. Results limited to your read scopes.


### Request

`GET /api/2/correlations/`

=== "Shell"

    ```shell
    curl "https://exist.io/api/2/correlations/" -H "Authorization: Bearer [YOUR_TOKEN]" 
    ```

=== "Python"

    ```python
    import requests

    requests.get("https://exist.io/api/2/correlations/",
        headers={'Authorization':'Token [YOUR_TOKEN]'})
    ```


### Parameters

Name  | Description
------|--------
`limit` | Number of values to return per page. Optional, max is 100.
`page`  | Page index. Optional, default is 1.
`strong` | Boolean flag, set to `1` to return only correlations above a certain relationship strength
`confident` | Boolean flag, set to `1` to return only correlations with a five-star confidence
`attribute` | Pass the name of an attribute to filter correlations to this attribute only


### Response

Returns a JSON object containing an array of correlation results:


```json
{
  "count": 479, 
  "next": "https://exist.io/api/2/correlations/?page=2", 
  "previous": null, 
  "results": [
    {
      "date": "2022-05-16",
      "period": 309,
      "offset": 0,
      "attribute": "sleep",
      "attribute2": "sleep_start",
      "value": -0.5577851552276848,
      "p": 1.1564227190131434e-26,
      "percentage": 55.77851552276848,
      "stars": 5,
      "second_person": "you spend more time asleep when you go to bed earlier.",
      "second_person_elements": [
        "you spend more time asleep",
        "when",
        "you go to bed earlier"
      ],
      "attribute_category": null,
      "strength_description": "Quite often go together",
      "stars_description": "Certain to be related",
      "description": null,
      "occurrence": null,
      "rating": null
    },
    # ...snip!...
  ]
}
```


## Find a specific correlation combination

Takes parameters for the two related attributes and returns one or zero correlations, depending on whether Exist has found something. Will only return a result if at least one of the attributes is in your read scopes.

### Request

`GET /api/2/correlations/combo/`


=== "Shell"

    ```shell
    curl "https://exist.io/api/2/correlations/combo/?attribute=sleep&attribute2=workouts_min" -H "Authorization: Bearer [YOUR_TOKEN]" 
    ```

=== "Python"

    ```python
    import requests

    requests.get("https://exist.io/api/2/correlations/combo/",
        params={"attribute":"sleep", "attribute2":"workouts_min"},
        headers={'Authorization':'Bearer [YOUR_TOKEN]'})
    ```


### Parameters

 Name | Description
 -----|------------
`attribute` | String name of first attribute
`attribute2` | String name of second attribute

These can be in any order.

### Response

Returns a single correlation object or a HTTP status `404` result with an error object if none is found.

```json
{
  "date": "2022-05-16",
  "period": 305,
  "offset": 0,
  "attribute": "sleep",
  "attribute2": "workouts_min",
  "value": 0.12521231439002578,
  "p": 0.028788436765440892,
  "percentage": 12.521231439002579,
  "stars": 4,
  "second_person": "you spend more time working out when you spend more time asleep.",
  "second_person_elements": [
    "you spend more time working out",
    "when",
    "you spend more time asleep"
  ],
  "attribute_category": null,
  "strength_description": "Rarely go together",
  "stars_description": "Almost certainly related",
  "description": null,
  "occurrence": null,
  "rating": {
    "positive": true,
    "rating_type": 50,
    "rating": "Useful"
  }
}
```

