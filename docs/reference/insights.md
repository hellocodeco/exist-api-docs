---
title: Insights
---

## Get all insights

Returns a paged list of user's insights. 

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
    curl -H "Authorization: Token [YOUR_TOKEN]" https://exist.io/api/2/insights/
    ```

=== "Python"

    ```python
    import requests

    requests.get("https://exist.io/api/2/insights/",
        headers={'Authorization':'Token [YOUR_TOKEN]'})
    ```

### Response

Returns a JSON object containing a paged array:

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
