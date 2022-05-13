---
title: Attributes
---


## Get attribute templates

Retrieve a list of the attribute templates Exist supports. See [attribute template definition and list](/reference/object_types/#list-of-attribute-templates).

### Request

`GET /api/2/attributes/templates/`

=== "Shell"

    ```shell
    curl "https://exist.io/api/2/attributes/templates/" -H "Authorization: Bearer [token]"
    ```

=== "Python"
    ```python
    import requests

    requests.get("https://exist.io/api/2/attributes/templates/",
                 headers = {'Authorization': 'Bearer [token]'})
    ```


### Parameters

| Parameter | Description |
|-----------|-------------|
| `page` | Integer used for paging the results |
| `limit` | Integer used to define how many results to a page |
| `include_low_priority` | Boolean flag, set to `true` to include attributes with a `priority` > 10 |
| `groups`  | Comma-separated list of groups to filter by, e.g. `activity,workouts`|


### Response

A paged list of attribute objects.

```json
{
  "count": 81,
  "next": "https://exist.io/api/2/attributes/templates/?page=2",
  "previous": null,
  "results": [
    {
      "name": "steps",
      "label": "Steps",
      "group": {
        "name": "activity",
        "label": "Activity",
        "priority": 1
      },
      "priority": 1,
      "value_type": 0,
      "value_type_description": "Integer"
    },
    {
      "name": "steps_active_min",
      "label": "Active minutes",
      "group": {
        "name": "activity",
        "label": "Activity",
        "priority": 1
      },
      "priority": 2,
      "value_type": 3,
      "value_type_description": "Period (min)"
    },
    # ...snip!...
  ]
}
```


## Get a user's attributes

Retrieve a list of the user's attributes, without any values.

### Request

`GET /api/2/attributes/`

### Parameters

| Parameter | Description |
|-----------|-------------|
| `page` | Integer used for paging the results |
| `limit` | Integer used to define how many results to a page |
| `groups`  | Comma-separated list of groups to filter by, e.g. `activity,workouts`|
| `attributes` | Comma-separated list of attributes to filter by |
| `exclude_custom` | Boolean flag, set to `true` to only show templated attributes |
| `manual` | Boolean flag, set to `true` to only show manual attributes |
| `include_inactive` | Boolean flag, set to `true` to include attributes with `active = False`, usually hidden |
| `include_low_priority` | Boolean flag, set to `true` to include attributes with a `priority` > 10 |
| `owned` | Boolean flag, set to `true` to omit attributes not owned by this client |

### Response

A paged list of attribute objects belonging to this user. `available_services` shows the services a user has connected which have indicated they can provide data for this attribute.

```json
{
  "count": 134,
  "next": "https://exist.io/api/2/attributes/?page=2",
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
        "name": "googlefit",
        "label": "Google Fit"
      },
      "active": true,
      "priority": 1,
      "manual": false,
      "value_type": 0,
      "value_type_description": "Integer",
      "available_services": [
        {
          "name": "googlefit",
          "label": "Google Fit"
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
    # ...snip!...
  ]
}
```

## Get attributes with values

### Request

### Parameters

### Response


## Get a specific attribute

Returns a paged list of all values from an attribute.

### Request

`GET /api/2/attributes/values/`

### Parameters

Name  | Description
------|--------
`attribute` | Non-optional, string name of the attribute to provide values for 
`limit` | Number of values to return per page. Optional, max is 100.
`page`  | Page index. Optional, default is 1.
`date_max` | Most recent date (inclusive) of results to be returned, in format `YYYY-mm-dd`. Optional.

### Response

```json
{
  "count": 3217,
  "next": "https://exist.io/api/2/attributes/values/?attribute=steps&page=2",
  "previous": null,
  "results": [
    {
      "date": "2022-05-13",
      "value": 1765
    },
    {
      "date": "2022-05-12",
      "value": 6792
    },
    {
      "date": "2022-05-11",
      "value": 4632
    },
    {
      "date": "2022-05-10",
      "value": 2121
    },
    {
      "date": "2022-05-09",
      "value": 5141
    },
    # ...snip!...
  ]
}
```
