---
title: Insights
---

## Get all insights

Returns a paged list of the authenticated user's insights. Results are limited to your read scopes.

### Request

`GET /api/2/insights/`

### Parameters

Name  | Description
------|--------
`limit` | Number of values to return per page, starting with today. Optional, max is 100.
`page`  | Page index. Optional, default is 1.
`date_min` | Oldest date (inclusive) of results to be returned, in format `YYYY-mm-dd`. Optional.
`date_max` | Most recent date (inclusive) of results to be returned, in format `YYYY-mm-dd`. Optional.
`priority` | Filter by insight priority, where `1` = today and `4` = last month. Optional.


=== "Shell"

    ```shell
    curl "https://exist.io/api/2/insights/" -H "Authorization: Bearer [YOUR_TOKEN]"
    ```

=== "Python"

    ```python
    import requests

    requests.get("https://exist.io/api/2/insights/",
        headers={'Authorization':'Bearer [YOUR_TOKEN]'})
    ```

### Response

Returns a JSON object containing a paged array:

```json
{
  "count": 12,
  "next": null,
  "previous": null,
  "results": [
    {
      "created": "2022-05-16T13:17:03+08:00",
      "target_date": "2022-05-16",
      "type": {
        "name": "dow_sleep",
        "period": 1,
        "priority": 1,
        "attribute": {
          "name": "sleep",
          "label": "Time asleep",
          "group": {
            "name": "sleep",
            "label": "Sleep",
            "priority": 3
          },
          "priority": 1,
          "value_type": 3,
          "value_type_description": "Period (min)"
        }
      },
      "html": "<div class=\"secondary\">Last night&#x27;s sleep was surprisingly long.</div>\r\n<div class=\"num-label\">Monday is usually your shortest sleep of the week.</div>",
      "text": "Last night's sleep was surprisingly long.\r\nMonday is usually your shortest sleep of the week."
    },
    # ...snip!...
  ]
}
```
