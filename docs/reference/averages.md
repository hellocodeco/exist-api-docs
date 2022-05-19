---
title: Averages
---

## Get averages


Returns the most recent average values for each attribute, one set per week. Can be used to retrieve historical averages for attributes also. Results are limited to your read scopes.

### Request

`GET /api/2/averages/`

=== "Shell"

    ```shell
    curl "https://exist.io/api/2/averages/" -H "Authorization: Bearer [YOUR_TOKEN]"
    ```

=== "Python"

    ```python
    import requests

    requests.get("https://exist.io/api/2/averages/",
        headers={'Authorization':'Bearer [YOUR_TOKEN]'})
    ```

### Parameters

By default, without any parameters, you'll receive the most recent set of averages across all attributes belonging to your read scopes. By using `include_historical=1` along with `attributes=[name]`, you can retrieve a list of multiple records per attribute, newest first.

| Parameter | Description |
|-----------|-------------|
| `page` | Integer for paging the results |
| `limit` | Integer defining how many results to a page |
| `date_min` | `"yyyy-mm-dd"` date string defining the minimum (oldest) date |
| `date_max` | `"yyyy-mm-dd"` date string defining the maximum (most recent) date |
| `groups`  | Comma-separated list of groups to filter by, e.g. `activity,workouts`|
| `attributes` | Comma-separated list of attributes to filter by |
| `include_historical` | Optional boolean, set `true` to receive historical records also


### Response 

Returns JSON object containing a paged array:

```json
{
  "count": 2,
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
}
```

