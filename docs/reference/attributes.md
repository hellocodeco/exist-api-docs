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

All parameters are optional.

| Parameter | Description |
|-----------|-------------|
| `page` | Page index. Optional, default is 1. |
| `limit` | Integer defining how many results to a page |
| `include_low_priority` | Boolean flag, set to `true` to include attributes with a `priority` >= 10 |
| `groups`  | Comma-separated list of groups to filter by, e.g. `activity,workouts`|


### Response

A paged list of attribute template objects.

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

Retrieve a list of the user's attributes, without any values. Results are limited to your read scopes.

### Request

`GET /api/2/attributes/`

=== "Shell"

    ```shell
    curl "https://exist.io/api/2/attributes/" -H "Authorization: Bearer [token]"
    ```

=== "Python"
    ```python
    import requests

    requests.get("https://exist.io/api/2/attributes/",
                 headers = {'Authorization': 'Bearer [token]'})
    ```


### Parameters

All parameters are optional.

| Parameter | Description |
|-----------|-------------|
| `page` | Page index. Optional, default is 1. |
| `limit` | Integer defining how many results to a page |
| `groups`  | Comma-separated list of groups to filter by, e.g. `activity,workouts`|
| `attributes` | Comma-separated list of attributes to filter by |
| `exclude_custom` | Boolean flag, set to `true` to only show templated attributes |
| `manual` | Boolean flag, set to `true` to only show manual attributes |
| `include_inactive` | Boolean flag, set to `true` to include attributes with `active = False`, usually hidden |
| `include_low_priority` | Boolean flag, set to `true` to include attributes with a `priority` >= 10 |
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

Returns a paged list of the user's attributes and their recent values. Results are limited to your read scopes.

### Request

`GET /api/2/attributes/with-values/`

=== "Shell"

    ```shell
    curl "https://exist.io/api/2/attributes/with-values/" -H "Authorization: Bearer [token]"
    ```

=== "Python"
    ```python
    import requests

    requests.get("https://exist.io/api/2/attributes/with-values/",
                 headers={'Authorization': 'Bearer [token]'})
    ```


### Parameters

If filtering is needed, then rather than using both, you would use one or the other of `attributes` and `templates` to filter, depending on whether you care about templated attributes or not.

| Parameter | Description |
|-----------|-------------|
| `page` | Page index. Optional, default is 1. |
| `limit` | Integer defining how many results to a page |
| `days`  | Integer defining how many day values to include in `values`, max 31, default 1 |
| `date_max` | `"yyyy-mm-dd"` date string defining the maximum date in `values` |
| `groups`  | Comma-separated list of groups to filter by, e.g. `activity,workouts`|
| `attributes` | Comma-separated list of attributes to filter by |
| `templates` | Comma-separated list of attribute templates to filter by |
| `manual` | Boolean flag, set to `true` to only show manual attributes |

### Response

Note the inclusion of the `values` array, which contains date/value pairs.

```json
{
  "count": 98,
  "next": "https://exist.io/api/2/attributes/with-values/?page=2",
  "previous": null,
  "results": [
    {
      "group": {
        "name": "activity",
        "label": "Activity",
        "priority": 1
      },
      "template": "steps",
      "name": "steps",
      "label": "Steps",
      "priority": 1,
      "manual": false,
      "active": true,
      "value_type": 0,
      "value_type_description": "Integer",
      "service": {
        "name": "googlefit",
        "label": "Google Fit"
      },
      "values": [
        {
          "date": "2022-05-16",
          "value": 1533
        }
      ]
    },
    # ...snip!...
  ]
}
```


## Get a specific attribute

Returns a paged list of all values from an attribute. Results are limited to your read scopes.

### Request

`GET /api/2/attributes/values/`

=== "Shell"

    ```shell
    curl "https://exist.io/api/2/attributes/values/?attribute=[name]" -H "Authorization: Bearer [token]"
    ```

=== "Python"
    ```python
    import requests

    requests.get("https://exist.io/api/2/attributes/values/",
                 params={"attribute": "[name]"},
                 headers={'Authorization': 'Bearer [token]'})
    ```


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
