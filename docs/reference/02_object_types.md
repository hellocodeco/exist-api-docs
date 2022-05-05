---
title: Object types and terminology
---

## Users

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
    "local_time": "2020-05-08T20:56:20.417+10:00",
    "private": false,
    "imperial_units": false
}
```
Users are pretty self-explanatory.

Most of the time you will only be concerned with the currently authenticated user, so
wherever you see the `username` argument in a URL you may substitute the special keyword `$self` to request the
authenticated user.

## Attributes

```json
{
    "attribute": "steps",
    "label": "Steps",
    "group": 
        {
            "name": "activity",
            "priority": 1
        },
    "priority": 1,
    "service": "Fitbit",
    "value_type": 0,
    "value_type_description": "Integer",
    "private": false,
    "values": [
        {
            "value": 3725,
            "date": "2015-05-08"
        },
        {
            "value": 6177,
            "date": "2015-05-07"
        },
        {
            "value": 7811,
            "date": "2015-05-06"
        },
        {
            "value": 6632,
            "date": "2015-05-05"
        },
        {
            "value": 6014,
            "date": "2015-05-04"
        },
        {
            "value": 4376,
            "date": "2015-05-03"
        },
        {
            "value": 8671,
            "date": "2015-05-02"
        },
        {
            "value": 4658,
            "date": "2015-05-01"
        }
    ]
}

```

This is our name for data points or individual numbers a user can track about themselves.
These are tracked at a single day granularity — there's no per-hour or -minute data. Attributes can be
strings but are usually quantifiable, ie. integers or floats.

An attribute has many values, one for each day that it has been tracked. In some responses, you'll see the `value` property
reflected within an attribute object. At other times, particularly if you are requesting multiple days of data, the `values` property will
contain an array of `date`/`value` pairs.

If there is no data for a particular date, this will be reflected with a null value — you should expect to
receive a list of results containing every single day, rather than days without data being left out.

All datetimes are in UTC unless otherwise specified, and should have the user's timezone applied to create a local TZ-aware datetime. All dates are local to the user.

Values are always stored internally and returned in metric units. Each user object contains an `imperial_units` boolean which must be respected when formatting values for the user,
ie. if this is `true`, a `steps_distance` value must be converted from km to miles.

In situations where multiple attributes are requested, for example the "today" overview, these attributes will be returned **grouped**.
A group represents attributes that belong together, for example the "activity" group contains the attributes `steps`, `active_min`, etc.
You can see an example of this in the ['overview' endpoint](#get-current-overview-for-user).

Clients should display attributes in these groups when displaying multiple attributes.
Groups are currently fairly broad and may change as we add more supported attributes.

See [list of supported attributes](#list-of-attributes).

## Correlations

```json
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
}
```

Correlations are a measure of the relationship between two variables. A positive
correlation implies that as one variable increases (its values get higher), so
does the other. A negative correlation implies that as one variable increases,
the other decreases. We present these to users as a way to explain past trends,
and use them to predict future behaviour.

Correlation values vary between -1 and +1 with 0 implying no correlation.
Correlations of -1 or +1 imply an exact linear relationship.

P-values are a way of determining confidence in the result. A value lower than
0.05 is generally considered "statistically significant".

We create simple English sentences to represent each possible correlation as a combination of attributes.

We're also careful to represent these as correlations only, not as one attribute directly causing a change in
the other, and we ask that you do the same. **Correlation is not causation**.
There may be many hidden factors which underlie the relationship between two attributes. It is up to the user to
determine the cause.

Correlations are generated weekly, so past values are also available in order to chart their changing strength.

### Examples

**"You are moderately more likely (55%) to be productive when you listen to music."**

In this case the value is 0.55, and this is a positive correlation — when one value (*tracks played*) increases,
so does the other (*time spent productively*).

**"You are somewhat less likely (25%) to be active when it's warmer in the day."**

In this case the value is -0.25, and this is a negative correlation — when one value (*max temp*) increases,
the other decreases (*time active*).

## Insights

```json
{
    "created": "2015-05-08T05:05:46Z",
    "target_date": "2015-05-07",
    "html": "<div class=\"secondary\">Thursday marks a new productivity streak!</div>",
    "value": "3 days",
    "value2": "3",
    "comment": null,
    "type": {
        "name": "productive_min_goal_best_streak",
        "period": 1,
        "priority": 2,
        "attribute": {
            "name": "productive_min",
            "label": "Productive time",
            "group": {
                "name": "productivity",
                "priority": 2
            },
            "priority": 1
        }
    }
}
```

Insights are interesting events found within the user's data. These are not triggered by the user but generated automatically if the user's data fits an insight type's criteria. Typically these fall into a few categories: day-level events, for example, yesterday was the highest or lowest steps value in however many days; and week and month-level events, like summaries of total steps walked for the month. If an insight is relevant to a specific day it will contain a `target_date` value.

Insights have a priority where `1` is highest and means real-time, `2` is day-level, `3` is week, and `4` is month.

HTML output is provided, however each insight could be assembled from the value fields and knowledge of the insight type, if required.

### Examples

**Thursday marks a new productivity streak!** Beat your goal every day for 3 days

## Averages
```json
{
    "attribute": "floors",
    "date": "2015-05-03",
    "overall": 13,
    "monday": 13,
    "tuesday": 16,
    "wednesday": 13,
    "thursday": 12,
    "friday": 14,
    "saturday": 13,
    "sunday": 13
}
```

Averages are generated weekly and are the basis of our goal system. For attributes that don't warrant a specific related `attribute_name_goal` attribute, the average is used to create the "end value" of the attribute's progress bar in our dashboard — meaning each day, users are being shown their progress relative to their usual behaviour. We break down averages by day of the week but also record the overall average. As we keep historical data this allows us to plot "rolling averages" showing changes in attribute values.

Note: these are actually medians, but we use "average" as it's simpler to explain to users. Please also use this terminology.

## Clients and services

We use **client** to refer an application with OAuth2 client credentials. A client which writes data to attributes is termed a **service**. 


## List of attributes

All attributes we currently support. The group an attribute belongs to may change in future, but attribute names should be considered stable.

Remember all data should be sent, and is stored and returned, in metric units. Imperial conversion should occur when rendering as required.

See [attribute definition](#attributes).

Name                | Group        | Value type description         | Value type 
--------------------| ------------ | -------------------------------|-----------
`steps`             | Activity     | Integer                        | `0`
`steps_active_min`  | Activity     | Period (minutes as integer)    | `3`
`steps_elevation`   | Activity     | Float (km)                     | `1`
`floors`            | Activity     | Integer                        | `0`
`steps_distance`    | Activity     | Float (km)                     | `1`
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
`custom`            | Custom tracking | String                      | `2`
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
`mood`              | Mood         | Integer (between 1 and 9 inclusive)    | `0`
`mood_note`         | Mood         | String (max 1000 characters)    | `2`
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
`sunrise`               | Weather  | Time of day (minutes from midnight as integer) | `4`
`sunset`                | Weather  | Time of day (minutes from midday as integer) | `6`

## Attribute value types

These are the allowed types of values an attribute can store.

Value type description         | Value type 
-------------------------------|-----------
Integer                        | `0`
Float                          | `1`
String                         | `2`
Period (minutes as integer)    | `3`
Time of day (minutes from midnight as integer) | `4`
Float (percentage, 0.0 to 1.0) | `5`
Time of day (minutes from midday as integer) | `6`
Boolean                        | `7`
Scale (1-9 as integer)		   | `8`



## Custom tags

> An example of the today endpoint's output for the custom group.

```json

{

    "group": "custom",
    "label": "Custom tracking",
    "priority": 3,
    "items": [
        {
            "attribute": "custom",
            "label": "Custom tracking",
            "value": "accounting,",
            "service": "exist_for_android",
            "priority": 1,
            "private": true,
            "value_type": 2,
            "value_type_description": "String"
        },
        {
            "attribute": "accounting",
            "label": "Accounting",
            "value": 1,
            "service": "builtin",
            "priority": 2,
            "private": true,
            "value_type": 0,
            "value_type_description": "Integer"
        },
        {
            "attribute": "tired",
            "label": "Tired",
            "value": 0,
            "service": "builtin",
            "priority": 2,
            "private": true,
            "value_type": 0,
            "value_type_description": "Integer"
        },
        
    ]
}

```

Custom tags are represented as integer type attributes within the `custom` group, and are user-defined so unable to be listed here. Examples include tags like `meditation`, `tired`, and `sex`. 

The `custom` attribute is a string representation of the tags a user has sent, but may be truncated to the last tag to fit within 250 characters. Using this string might be helpful to you as a quick and easy way to display tags, so feel free to use it if it suits your purposes, but is not the source of truth. To correctly get all tags for a day, collect all attribute names for attributes with a priority of 2 or higher, within the `custom` group, with a value of `1`. A value of `1` represents the "tagged" state, `0` meaning the tag was not used for that day.  

Tag names are stored with spaces converted to underscores and should always be rendered with underscores converted back to spaces, to be more user-friendly.

