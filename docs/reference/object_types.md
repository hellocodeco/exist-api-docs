---
title: Object types and terminology
---

## Users

This data is useful for customising the display of user data to the user's preferences. Use `timezone` to show times in the correct time zone for the user. Use the `imperial_*` flags to know which values to show in imperial units.

```json

{
    "username": "josh",
    "first_name": "Josh",
    "last_name": "Sharp",
    "avatar": "https://exist.io/media/avatars/josh_1.png",
    "timezone": "Australia/Melbourne",
    "local_time": "2022-05-13T16:55:54.302963+10:00",
    "imperial_distance": false,
    "imperial_weight": false,
    "imperial_energy": false,
    "imperial_liquid": false,
    "imperial_temperature": false,
    "trial": false,
    "delinquent": false
}
```


## Attributes


This is our name for data points or individual numbers a user can track about themselves.
These are tracked at a single day granularity — there's no per-hour or -minute data. Attributes can be
string types but are usually quantifiable, i.e. integers or floats. The `value_type` field reflects this type.

An attribute has many values, one for each day that it has been tracked. If requested, the `values` property will
contain an array of `date`/`value` pairs.

If there is no data for a particular date, this will be reflected with a null value — you should expect to
receive a list of results containing every single day, rather than days without data being omitted.

All datetimes in values are in UTC unless otherwise specified, and should have the user's timezone applied to create a local TZ-aware datetime. All dates are local to the user.

Values are always stored internally and returned in metric units. Each user object contains `imperial_*` flags which must be respected when formatting values for the user, i.e. if `imperial_distance` is `true`, a `steps_distance` value must be converted from kilometres to miles.

Attributes have a `priority` integer which is used for sorting (in ascending order). Attributes belong to groups which have a `label` and a `priority`, also used for sorting. Clients should display attributes in these groups when displaying multiple attributes.
Groups are currently fairly broad and may change as we add more supported attributes.

`manual` attributes are those which are not automatically filled with data from a supported integration, but instead have manual entry UI provided within our official clients.

See also [list of attribute templates](#list-of-attribute-templates) and [attribute value types](#attribute-value-types).

```json
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
            "date": "2022-05-13",
            "value": 1337
        }
    ]
}

```


## Correlations

Correlations are a measure of the relationship between two variables. A positive correlation implies that as one variable increases (its values get higher), so does the other. A negative correlation implies that as one variable increases, the other decreases. We present these to users as a way to explain past trends, and use them to predict future behaviour.

Correlation values vary between -1 and +1 with 0 implying no correlation. Correlations of -1 or +1 imply an exact linear relationship.

P-values are a way of determining confidence in the result. A value lower than 0.05 is generally considered "statistically significant".

We create simple English sentences to represent each possible correlation as a combination of attributes.

We're also careful to represent these as correlations only, not as one attribute directly causing a change in the other, and we ask that you do the same. **Correlation is not causation**. There may be many hidden factors which underlie the relationship between two attributes. It is up to the user to determine the cause.

Correlations are generated weekly on Sundays.

```json
{
  "date": "2022-05-08",
  "period": 364,
  "offset": 0,
  "attribute": "steps",
  "attribute2": "walk",
  "value": 0.6754596120404917,
  "p": 7.928064702293852e-50,
  "percentage": 67.54596120404916,
  "stars": 5,
  "second_person": "you get more steps when you tag 'walk' more.",
  "second_person_elements": [
    "you get more steps",
    "when",
    "you tag 'walk' more"
  ],
  "attribute_category": null,
  "strength_description": "Nearly always go together",
  "stars_description": "Certain to be related",
  "description": null,
  "occurrence": null,
  "rating": {
    "positive": true,
    "rating_type": 50,
    "rating": "Useful"
  }
}
```

`attribute_category` is used to distinguish "sub-correlations", that is, correlations for a subset of this attribute's data. For example, we may find a correlation between `events` in a specific calendar and `productive_min` — in this case, `attribute_category` would contain the name of the specific calendar.

`description` and `occurrence` fields are filled when both attributes are templated and we understand this relationship. For example, for the `productive_min` and `tracks` positive correlation, `occurrence` will be `"Very common"` and `description` will provide some further information about why music helps us be more productive.

`rating` provides a user-submitted rating object on the value of the correlation.


### Examples


>    **"You are more productive when you listen to music more."**
>
>   (55% strength / 5 stars / P 0.007)

In this case the value is 0.55, and this is a positive correlation — when one value (*tracks played*) increases,
so does the other (*time spent productively*).

> **"You are less active when it's a warmer day."**
>
> (25% strength / 5 stars / P 0.02 )

In this case the value is -0.25, and this is a negative correlation — when one value (*max temp*) increases,
the other decreases (*time active*).

## Insights

Insights are interesting events found within the user's data. These are not triggered by the user but generated automatically if the values for an attribute fits an insight type's criteria. Typically these fall into a few categories: day-level events, for example, yesterday was the highest or lowest steps value in however many days; and week and month-level events, like summaries of total steps walked for the month. If an insight is relevant to a specific day it will contain a `target_date` value.

Insights have a priority where `1` is highest and means real-time, `2` is day-level, `3` is week, and `4` is month.

HTML and text output is provided.

```json
{
    "created": "2022-05-13T08:17:30+01:00",
    "target_date": "2022-05-13",
    "type": {
        "name": "dow_tracks",
        "period": 1,
        "priority": 1,
        "attribute": {
            "name": "tracks",
            "label": "Tracks played",
                "group": {
                "name": "media",
                "label": "Media",
                "priority": 7
            },
            "priority": 1,
            "value_type": 0,
            "value_type_description": "Integer"
        }
    },
    "html": "<div class=\"secondary\">You listen to more music on a Friday.</div>\r\n<div class=\"num-label\">&quot;Kurt Vile&quot; has been on high rotation.</div>",
    "text": "You listen to more music on a Friday.\r\n\"Kurt Vile\" has been on high rotation."
},
```

## Averages

Averages are generated weekly and are the basis of our goal system. For attributes that don't warrant a specific related `attribute_name_goal` attribute, the average is used to create the "end value" of the attribute's progress bar in our dashboard — meaning each day, users are being shown their progress relative to their usual behaviour. We break down averages by day of the week but also record the overall average. As we keep historical data this allows us to plot "rolling averages" showing changes in attribute values. The data set for finding the average is always the last 60 days' data.

**Note:** these are actually medians, but we use "average" as it's simpler to explain to users. Please also use this terminology.

```json
{
    "user_attribute": "sleep",
    "date": "2022-05-08",
    "overall": 399,
    "monday": 382,
    "tuesday": 425,
    "wednesday": 401,
    "thursday": 394,
    "friday": 376,
    "saturday": 406,
    "sunday": 419
}
```


## Clients and services

We use **client** to refer an application with OAuth2 client credentials. A client which writes data to attributes is termed a **service**. Where we use the term **integration**, this means a service with a first-party integration with Exist.


## List of attribute templates

All the "official" attributes we currently support. Templated attributes are treated differently because we "understand" them — they receive their own insights, individual wording for correlations, and so on. You should prefer them over custom attributes where possible. 

The group an attribute belongs to may change in future, but attribute names should be considered stable.

Remember all data must be sent, and is stored and returned, in metric units. Imperial conversion should occur when rendering as required.

See [attribute definition](#attributes).

Name                | Group        | Value type description         | Value type 
--------------------| ------------ | -------------------------------|-----------
`steps`             | Activity     | Integer                        | `0`
`steps_active_min`  | Activity     | Period (minutes as integer)    | `3`
`steps_elevation`   | Activity     | Float (km)                     | `1`
`floors`            | Activity     | Integer                        | `0`
`steps_distance`    | Activity     | Float (km)                     | `1`
`stand_hours`       | Activity     | Integer                        | `0`
`cycle_min`         | Activity     | Period (minutes as integer)    | `3`
`cycle_distance`    | Activity     | Float (km)                     | `1`
`active_energy` | Activity | Float (kJ) | `1`
`workouts`          | Workouts     | Integer                        | `0`
`workouts_min`      | Workouts     | Period (minutes as integer)    | `3`
`workouts_distance` | Workouts     | Float (km)                     | `1`
`productive_min`    | Productivity | Period (minutes as integer)    | `3`
`neutral_min`       | Productivity | Period (minutes as integer)    | `3`
`distracting_min`   | Productivity | Period (minutes as integer)    | `3`
`commits`           | Productivity | Integer                        | `0`
`tasks_completed`   | Productivity | Integer                        | `0`
`words_written`     | Productivity | Integer                        | `0`
`emails_sent`       | Productivity | Integer                        | `0`
`emails_received`   | Productivity | Integer                        | `0`
`pomodoros_min` | Productivity | Period (minutes as integer) | `3`
`keystrokes` | Productivity | Period (minutes as integer) | `3`
`coffees`           | Food and drink | Integer                      | `0`
`alcoholic_drinks`  | Food and drink | Integer                      | `0`
`energy`            | Food and drink | Float (kJ)                   | `1`
`water`             | Food and drink | Integer (ml)                 | `0`
`carbohydrates`     | Food and drink | Float (g) | `1`
`fat`               | Food and drink | Float (g) | `1`
`fibre`             | Food and drink | Float (g) | `1`
`protein`           | Food and drink | Float (g) | `1`
`sugar`             | Food and drink | Float (g) | `1`
`sodium`            | Food and drink | Float (mg) | `1`
`cholesterol`       | Food and drink | Float (mg) | `1`
`caffeine`          | Food and drink | Float (mg) | `1`
`money_spent`       | Finance      | Float (user's local currency unit) | `1`
`mood`              | Mood         | Integer scale (between 1 and 9)    | `8`
`mood_note`         | Mood         | String (max 1000 characters)    | `2`
`energy_level`      | Mood         | Integer scale (between 1 and 9)    | `8`
`stress_level`      | Mood         | Integer scale (between 1 and 9)    | `8`
`sleep`             | Sleep        | Period (minutes as integer)    | `3`
`time_in_bed`       | Sleep        | Period (minutes as integer)    | `3`
`sleep_start`       | Sleep        | Time of day (minutes from midday as integer)   | `6`
`sleep_end`         | Sleep        | Time of day (minutes from midnight as integer) | `4`
`sleep_awakenings`  | Sleep        | Integer                        | `0`
`events`            | Events       | Integer                        | `0`
`events_duration`   | Events       | Period (minutes as integer)    | `3`
`weight`            | Health       | Float (kg)                     | `1`
`body_fat`          | Health       | Float (percentage, 0.0 to 1.0) | `5`
`lean_mass` | Health | Float (kg) | `1`
`heartrate`         | Health       | Integer                        | `0`
`heartrate_max` | Health | Integer | `0`
`heartrate_resting` | Health | Integer | `0`
`meditation_min`    | Health       | Period (minutes as integer)    | `3`
`menstrual_flow` | Health | Integer (`0`=none, `1`=spotting, `2`=light, `3`=medium, `4`=heavy) | `0`
`sexual_activity` | Health | Integer | `0`
`checkins`          | Location     | Integer                        | `0`
`location`          | Location     | String (`"lat,lng"` format where `lat` and `lng` are floats) | `2`
`tracks`            | Media        | Integer                        | `0`
`articles_read`     | Media        | Integer                        | `0`
`pages_read`     | Media        | Integer                        | `0`
`mobile_screen_min`     | Media        | Period (minutes as integer)                        | `3`
`gaming_min`     | Media        | Period (minutes as integer)                        | `3`
`tv_min`     | Media        | Period (minutes as integer)                        | `3`
`facebook_posts`    | Social       | Integer                        | `0`
`facebook_comments` | Social       | Integer                        | `0`
`facebook_reactions`| Social       | Integer                        | `0`
`tweets`            | Twitter      | Integer                        | `0`
`twitter_mentions`  | Twitter      | Integer                        | `0`
`twitter_username`  | Twitter      | String                         | `2`
`weather_temp_max`  | Weather      | Float (degrees Celsius)        | `1`
`weather_temp_min`  | Weather      | Float (degrees Celsius)         | `1`
`weather_precipitation` | Weather  | Float (inches of water per hour) | `1`
`weather_cloud_cover`   | Weather  | Float (percentage of sky covered, 0.0 to 1.0) | `5`
`weather_wind_speed`    | Weather  | Float (km/hr)                  | `1`
`weather_summary`       | Weather  | String                         | `2`
`weather_icon`          | Weather  | String (name of icon best representing weather values) | `2`
`day_length`               | Weather  | Period (minutes as integer) | `3`


## Attribute value types

These are the allowed types of values an attribute can store.

Value type description         | Value type 
-------------------------------|-----------
Integer                        | `0`
Float                          | `1`
String                         | `2`
Period (minutes as integer)    | `3`
Time of day (minutes from midnight as integer) | `4`
Percentage (float, 0.0 to 1.0) | `5`
Time of day (minutes from midday as integer) | `6`
Boolean                        | `7`
Scale (1-9 as integer)		   | `8`



## Custom tags


Custom tags are booleans represented as integer type attributes (`0` or `1`) within the `custom` group, and are user-defined so unable to be listed here. Examples include tags like `meditation`, `tired`, and `sex`. 

To correctly get all tags for a day, collect all attributes:

- with a value type of `7`;
- within the `custom` group;
- with a value of `1`. 

A value of `1` represents the "tagged" state, `0` or `null` meaning the tag was not used for that day.  

Tags have internal `name`s like any other attribute, but remember to display them using their `label` field.

