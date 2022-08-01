
In this section we'll write simple Python scripts to retrieve our personal data from Exist and print it to the terminal. Along the way we'll learn about getting a token, authenticating ourselves, and some of the more common ways of interacting with the Exist API to retrieve data. This should build a good foundation for using the Exist API in a read-only fashion, perhaps to build a personal client, store and display our data on a personal website, or as the basis of creating a full read/write client.

Our code examples will be in Python, and you're encouraged to copy and paste them, save them, and run them yourself! I'm using Python 3.8, but Python 3.6 or newer should be fine. You'll need to make sure you have the `requests` library [installed](https://requests.readthedocs.io/en/latest/user/install/).

## Getting a token

To be able to read our own personal data from Exist, we need to authenticate ourselves, so Exist knows who we are. Let's start with the simplest method of authentication, which Exist calls *simple token authentication*. We'll ask for a unique token that identifies us, and then add it to each subsequent request so Exist can use it to know who we are.

Let's make a request to the simple token endpoint, using the `requests` library, exchanging our username and password for a token we can use to authenticate ourselves with the API. 

The endpoint at `https://exist.io/api/1/auth/simple-token/` takes a `username` and `password` and returns a JSON object containing a `token` field. Note that this is still at `/api/1/` although everything else we'll do will be using the new endpoints at `/api/2/`.

=== "get_token.py"

    ```python
    import requests
    from getpass import getpass

    username = input("Username: ")
    password = getpass("Password: ")

    url = 'https://exist.io/api/1/auth/simple-token/'

    response = requests.post(url, data={'username':username, 'password':password})
    
    # make sure the response was 200 OK, meaning no errors
    if response.status_code == 200: 
        # parse the json object from the response body so we can print the token
        data = response.json()
        print("Token:", data['token'])
    else: 
        print("Error!", response.content)

    ```

We can save this script as `get_token.py` (wherever you like), run it from the command line with `python3 get_token.py`, and interactively enter our username and password. The script then prints out our token, and we can save this to use in future scripts. This token won't expire so we can hard-code it into personal scripts with impunity. 

!!! note "Warning!"
    Don't share this token publicly! Anyone can use it to read your Exist data.


## Reading attribute values

Now we can authenticate ourselves, let's get some data! We'll start by just getting a list of our attributes and their values for today. To do this we make a `GET` request to `https://exist.io/api/2/attributes/with-values/`.

=== "get_attributes.py"

    ```python
    import requests
    from pprint import pprint

    TOKEN = "[your_token]"

    url = 'https://exist.io/api/2/attributes/with-values/'

    # make sure to authenticate ourselves with our token
    response = requests.get(url, headers={'Authorization': f'Token {TOKEN}'})
    
    if response.status_code == 200:
        data = response.json()
        # pretty print the json
        pprint(data)
    else:
        print("Error!", response.content)

    ```
    
You can see we're adding an `Authorization` header to our request this time, containing the token we received previously. Don't forget to put your real token into the `TOKEN` variable!

For now, let's just print what we get back. We're going to use the pretty-print module in Python to print our arrays and dictionaries with nice indents. 

Save and run this script with `python3 get_attributes.py` and we should see something like this:

```python

{
  "count": 122,
  "next": "http://exist.io/api/2/attributes/with-values/?page=2",
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
    }
  ]
}
```

So we can see this is a paged response (with 20 attributes to a page by default), a `next` field to tell us that there are more results and where to get them, and a `results` field that contains the actual attributes.

Each attribute has a lot of details about it, including its internal `name` and its user-facing `label`, the `group` it belongs to, the `service` providing its data, its `value_type` defining the type of data this attribute stores, and an array of `values` — one `value` per `date`. Because we didn't ask for a specific amount, we just got today's values.


If we wanted to get a full week of data, we could add the `days` parameter to our request, like so:

```python
response = requests.get(url, params={'days':7}, headers={'Authorization': f'Token {TOKEN}'})
```

And then the `values` array for each attribute would contain 7 objects, still starting from today:

```python3
"values": [
  {
    "date": "2022-05-16",
    "value": 1533
  },
  {
    "date": "2022-05-15",
    "value": 3001
  },
  {
    "date": "2022-05-14",
    "value": 7882
  }
]
```

And so on.

Okay, now we know what the JSON looks like, let's modify our code to instead print a list of attributes and their values.

=== "get_attributes.py"

    ```python
    import requests
    import datetime

    TOKEN = "[your_token]"

    url = 'https://exist.io/api/2/attributes/with-values/'

    response = requests.get(url, headers={'Authorization': f'Token {TOKEN}'})
    
    attributes = {}

    if response.status_code == 200:
        # parse the response body as json
        data = response.json()
        
        # collect the data we want into a dict
        for attribute in data['results']:
            # grab the fields we want from the json
            label = attribute['label']
            value = attribute['values'][0]['value']
            # and store them as key/value
            attributes[label] = value
        
        # print today's date
        print(datetime.date.today().strftime("%A %d %B").upper())
        # now print our attribute labels and values
        for label, value in attributes.items():
            print(f"{label}: {value}")
    else:
        print("Error!", response.content)
    ```

If we save and run our updated script, we should see something like this:

```
FRIDAY 10 JUNE
Steps: 4691
Active minutes: 31
Steps goal: 3531
Productive time: 83
Distracting time: 9
Neutral time: 9
Productive time goal: 80
alcohol: 0
bad sleep: 0
baking: 0
```

Helpfully, we get attributes from the API already grouped and sorted by group and priority. So in our example here we have some activity, productivity, and then custom tag attributes, each with their value for today. Custom tags are booleans represented as a `0` or `1`, so we can see a few unused tags here. Any attributes without data that don't have a default value, like time of day or scale types, will print `None`. Zero is the default value for other types like quantities.

But what we're printing is only the first page, so it's not every attribute. Let's update our script again to keep making calls with an incrementing `page` parameter until we have all attributes. We'll move the request into its own function so we can call it many times easily.

!!! note
    Be mindful that every new request counts against your rate limiting, so if you end up making 300 calls in an hour, you'll get rate-limited until the next hour. A rate-limited response has a `status_code` of `429` and no data.

=== "get_attributes.py"

    ```python
    import requests
    import datetime

    TOKEN = "[your_token]"
  
    url = 'https://exist.io/api/2/attributes/with-values/'
    
    attributes = {}

    def get_page(page):
      # we're passing in the page param, and also asking for 100 items per page
      response = requests.get(url, params={'page':page, 'limit':100}, headers={'Authorization': f'Token {TOKEN}'})
    
      if response.status_code == 200:
          # parse the response body as json
          data = response.json()
          
          # collect the data we want into a dict
          for attribute in data['results']:
              # grab the fields we want from the json
              label = attribute['label']
              value = attribute['values'][0]['value']
              # and store them
              attributes[label] = value

          # if there's a value for 'next', then let's get the next page
          if data['next'] is not None:
            # call this function again with the next page number
            get_page(page+1)

      else:
        print("Error!", response.content)

    # start here by requesting the first page
    get_page(1)

    # print today's date
    print(datetime.date.today().strftime("%A %d %B").upper())
    # now print our attribute labels and values
    for label, value in attributes.items():
        print(f"{label}: {value}")
  
    ```

Run the updated script and you'll see it takes a bit longer to complete, but then prints a long list of all our attributes:

```
FRIDAY 10 JUNE
Steps: 4691
Active minutes: 31

(lots of attributes omitted...)

Location name: Melbourne
Tracks played: 14
Time gaming: 0
Tweets: 0
Twitter mentions: 0
Max temp: 13.8
Min temp: 9.0
Precipitation: 0.09
Air pressure: 1017
Cloud cover: 0.82
Humidity: 0.82
Wind speed: 4.97
Day length: 576
Weather summary: Mostly cloudy throughout the day.
Weather icon: rain
```

Now we're printing every attribute and its value for today. We have a very basic terminal client for Exist! Each time we run it, it will print the current values for the current day for all of our attributes.

### Getting all tags for a day

Let's up the stakes and say we'd like to display a list of tags that were used, for a particular date we request. We've already seen that we can retrieve tags and their values, so this shouldn't be much more complicated.

We'll use the same API endpoint but this time we'll add some filters to the output. We'll ask for just the `custom` group, where all tags reside, and we'll set the `date_max` to a date of our choice, meaning the values we get back start at that (newest) date. With this parameter and the default limit of one day of data, we'll get a single `value` for each attribute that will match the date we ask for.

Every tag that has a `value` of `1` was used on this day, so we just need to collect the labels of every tag with this value and we're done.

=== "get_tags.py"

    ```python
    import requests
    import datetime

    TOKEN = "[your_token]"
  
    url = 'https://exist.io/api/2/attributes/with-values/'
    
    attributes = [] # we only need an array this time

    def get_page(date, page):
        # we received a date object, but the API takes a string of the format YYYY-mm-dd
        # so let's convert our date to the right format
        date_string = date.strftime("%Y-%m-%d")

        # note the new parameter filtering the group and the maximum date
        response = requests.get(url, params={'page':page, 'limit':100, 'groups':'custom', 'date_max':date_string}, headers={'Authorization': f'Token {TOKEN}'})
      
        if response.status_code == 200:
            # parse the response body as json
            data = response.json()
            
            # collect the data we want into a dict
            for attribute in data['results']:
                # there may not be a value, so let's try it and ignore any errors
                try:
                    # grab the value so we can check it
                    value = attribute['values'][0]['value']
                    # check that the tag was used
                    if value == 1:
                        # get and store the label
                        label = attribute['label']
                        attributes.append(label)
                except:
                    continue

            # if there's a value for 'next', then let's get the next page
            if data['next'] is not None:
                # call this function again with the next page number
                get_page(date, page+1)

        else:
            print("Error!", response.content)

    def get_date():

        # ask for a date string
        date_input = input("Date (format yyyy-mm-dd): ")
        try:
            # try converting the string into a date object
            date = datetime.datetime.strptime(date_input, "%Y-%m-%d").date()
            return date
        except:
            return

    # this is where our code starts executing when we run the script
    date = get_date()

    if date is None:
        print("Bad date input")
    else:
        # request the first page
        get_page(date, 1)

        # print a nicely formatted version of the date
        print(date.strftime("%A %d %B %Y").upper())
        # now print our attributes as a comma-delimited list
        print(", ".join(attributes))
    ```

If we save that (adding our token), run it, and enter a date, we'll see a list of tags for that day:

```
Date (format yyyy-mm-dd): 2022-06-12
SUNDAY 12 JUNE
bad sleep, camera, guitar, late to sleep, mask, melatonin, socialising, walk
```

Again, we get these attributes back in the correct order "for free".

So by this point we know how to print a list of attributes and their values, a list of tags used for a day, and how to request a specific date! We've learned a few more approaches for working with attributes.

## Getting averages

Every week Exist saves updated averages for each numeric attribute, one "overall" average as well as an average for each day of the week. One place this is used is as the "goal value" for progress bars in Exist client apps. To show the goal for the `steps` attribute on a Monday, for example, we'd get `steps`'s averages and use the `monday` value.

We can get averages by making a `GET` request to `https://exist.io/api/2/averages/`. If we do this with no other parameters, we see a paged list of JSON objects representing each attribute's averages:

```json
{
    "count": 71,
    "next": null,
    "previous": null,
    "results": [
        {
            "user_attribute": "steps",
            "date": "2022-06-12",
            "overall": 3531.0,
            "monday": 5155.0,
            "tuesday": 2403.0,
            "wednesday": 2637.0,
            "thursday": 4219.0,
            "friday": 3531.0,
            "saturday": 4062.0,
            "sunday": 2886.0
        },
        {
            "user_attribute": "tracks",
            "date": "2022-06-12",
            "overall": 9.0,
            "monday": 6.0,
            "tuesday": 9.0,
            "wednesday": 2.0,
            "thursday": 21.0,
            "friday": 14.0,
            "saturday": 0.0,
            "sunday": 0.0
        },
```

So now that we know what the format looks like, we can put this together with our previous work on reading attribute values to show the progess for one attribute for today. We'll ask the user the `name` of an attribute, get its current value, get its average, and then print their progress for the day. We'll use the `attributes` parameter to filter this endpoint to specific list of attributes — in this case, the single name we ask for.

=== "show_progress.py"

    ```python
    import requests
    import datetime

    TOKEN = "[your_token]"
  
    def get_average(name):
        """
        Gets the average for the current day of the week.
        """

        today = datetime.date.today()  # get today's date
        weekday_code = today.weekday()  # get today's day of the week, where monday = 0

        # make an array of the weekday keys, where monday is also 0, and so on
        weekdays = ["monday","tuesday","wednesday","thursday","friday","saturday","sunday"]

        # get the value we'll use to look up today's average by indexing the array with the current weekday
        weekday = weekdays[weekday_code]

        url = 'https://exist.io/api/2/averages/'

        response = requests.get(url, params={'attributes':name}, headers={'Authorization':f'Token {TOKEN}'})
        if response.status_code == 200:
            try:
                data = response.json()
                # get the first object, our one attribute's average
                average = data['results'][0]  
                # use our key to get the right value for today
                return average[weekday]
            except:
                # we assume all the fields we need are present
                # but if they're not, an exception will be thrown
                # so let's handle it very generally.
                print("Couldn't get average")
        else:
            print("Error!", response.content)

    def get_attribute(name):
        """
        Gets the attribute's label and current value for today.
        """

        result = {} # make an empty dictionary to store what we'll return

        # we're using the same attribute data call from the previous step
        url = 'https://exist.io/api/2/attributes/with-values/'

        response = requests.get(url, params={'attributes':name}, headers={'Authorization':f'Token {TOKEN}'})
        if response.status_code == 200:
            try:
                data = response.json()
                attribute = data['results'][0]
                result['label'] = attribute['label']
                result['value'] = attribute['values'][0]['value']
                return result
            except:
                # we're expecting some fields to be there, but they may not be, 
                # which would raise an exception
                # so let's just handle any failure very generally
                print("Couldn't get attribute")
        else:
            print("Error!", response.content)

    # running the script starts execution here
    name = input("Attribute name: ")

    # call our functions
    attribute = get_attribute(name)
    average = get_average(name)

    # make sure both calls succeeded
    if attribute is not None and average is not None:

        # we can only do this if we have a value for today
        if attribute['value'] is not None:
            # make a progress percentage
            percent = round((attribute['value'] / average) * 100)
        else:
            # our fallback is 0%
            percent = 0

        # get today's date to use for a header in our output
        today = datetime.date.today()

        # print it all
        print(today.strftime("%A %d %B %Y").upper())
        print(f"{attribute['label']}: {attribute['value']} / {average} ({percent}%)")

    ```

Save this script as `get_progress.py` (make sure to insert your correct token!), run it with `python3 get_progress.py`, and you'll see an output something like this:

```
Attribute name: steps
FRIDAY 17 JUNE 2022
Steps: 619 / 3531.0 (18%)
```

It's Friday for me today, so looking back at the example averages output a bit further up, you'll see that we correctly chose the `friday` value as the goal for today.

This won't format values nicely, so if you use a duration or a percentage, for example, these will show the raw integer or float values. But otherwise, we now have a nice way of using a current average to show our progress for today against our recent activity for the same day of the week.


### Showing long-term trends

In Exist, these average values are used to show graphs of long-term trends. Because averages are saved weekly, we can plot each `overall` value on a graph and show how the average has changed over time. Drawing a proper graph is outside the scope of this tutorial, but we can at least tabulate this data and draw a simple bar graph in the terminal.

We'll use the same averages endpoint as before, again filtering for just the one attribute, but this time we'll add the `include_historical` parameter to retrieve a list of past averages for the attribute. Instead of only returning this week's averages, we'll get back a paged list of all averages, back to the first one ever created for us when we created this attribute. We don't need all of those, though, so we'll use the `limit` parameter and only fetch the first page of results.


=== "show_trend.py"

    ```python
    import requests
    import datetime

    TOKEN = "[your_token]"

    def get_averages(name):
        """
        Gets a set of recent averages for an attribute.
        """
        url = 'https://exist.io/api/2/averages/'

        # we'll ask for 26 results (26 weeks = half a year)
        response = requests.get(url, params={'attributes':name, 'include_historical':1, 'limit':26}, 
                                headers={'Authorization':f'Token {TOKEN}'})
        if response.status_code == 200:
            try:
                data = response.json()
                averages = {} # make a dictionary to hold our results
                
                for obj in data['results']: # loop over each average object
                    # make a date object from the date string we receive
                    date = datetime.datetime.strptime(obj['date'], "%Y-%m-%d").date()
                    overall = obj['overall']
                    # store the fields we care about
                    averages[date] = overall
                
                return averages
            except:
                # we assume all the fields we need are present
                # but if they're not, an exception will be thrown
                # so let's handle it very generally.
                print("Couldn't get average")
        else:
            print("Error!", response.content)

    def make_bar_graph(value, maximum):
        """
        Makes an ascii bar graph bar scaled relative to the max value.
        """
        bar_len = 40
        filled_len = int(round(bar_len * value / maximum))
        # make a string of equals signs for the value
        # and fill the rest of the bar with hyphens
        bar = '=' * filled_len + '-' * (bar_len - filled_len)
        return bar

    
    # execution begins here
    name = input("Attribute name: ")
    averages = get_averages(name)
    
    if averages: # only print if we didn't get an error
        max_average = max(averages.values())

        for raw_date, value in averages.items():
            date = raw_date.strftime("%d %b %Y")
            bar = make_bar_graph(value, max_average)
            # print the formatted date, value, and graph with tabs to align them
            print(f"{date}:\t{value}\t{bar}")
    else:
        print("No averages to display.")
    ```


As usual, save this script as `show_trend.py` (make sure to insert your correct token!), run it with `python3 show_trend.py`, and you'll see something like this for your chosen attribute. I've chosen number of tracks played:

```
Attribute name: tracks
19 Jun 2022:    4.0     ===============-------------------------
12 Jun 2022:    3.0     ===========-----------------------------
05 Jun 2022:    1.0     ====------------------------------------
29 May 2022:    2.0     =======---------------------------------
22 May 2022:    3.0     ===========-----------------------------
15 May 2022:    5.0     ==================----------------------
08 May 2022:    5.0     ==================----------------------
01 May 2022:    10.0    ====================================----
24 Apr 2022:    9.0     =================================-------
17 Apr 2022:    10.0    ====================================----
10 Apr 2022:    9.0     =================================-------
03 Apr 2022:    9.0     =================================-------
27 Mar 2022:    11.0    ========================================
20 Mar 2022:    10.0    ====================================----
13 Mar 2022:    8.0     =============================-----------
06 Mar 2022:    6.0     ======================------------------
27 Feb 2022:    6.0     ======================------------------
20 Feb 2022:    6.0     ======================------------------
13 Feb 2022:    6.0     ======================------------------
06 Feb 2022:    6.0     ======================------------------
30 Jan 2022:    6.0     ======================------------------
23 Jan 2022:    6.0     ======================------------------
16 Jan 2022:    6.0     ======================------------------
09 Jan 2022:    6.0     ======================------------------
02 Jan 2022:    8.0     =============================-----------
26 Dec 2021:    9.0     =================================-------
```

Nice, right? Now we know how to request both current and historical averages and the sort of things they're good for.

## Showing correlations

Correlations describe a relationship between two different attributes and how they go together. An example might be "You have a better day when you walk more", where this is a positive relationship between `mood` and `steps`. Correlations get generated weekly based on up to a year of past data.

We can retrieve all of the most recent correlations by sending a `GET` request to `https://exist.io/api/2/correlations/`. We'll get another paged result that looks like this:

```json
{
    "count": 394,
    "next": "https://exist.io/api/2/correlations/?page=2",
    "previous": null,
    "results": [
        {
            "date": "2022-06-20",
            "period": 300,
            "offset": -1,
            "attribute": "bad_sleep",
            "attribute2": "coffee",
            "value": 0.7059233380455165,
            "p": 1.5026407410821477e-46,
            "percentage": 70.59233380455166,
            "stars": 5,
            "second_person": "you tag 'coffee' more on days after you tag 'bad sleep' more.",
            "second_person_elements": [
                "you tag 'coffee' more",
                "on days after",
                "you tag 'bad sleep' more"
            ],
            "attribute_category": null,
            "strength_description": "Nearly always go together",
            "stars_description": "Certain to be related",
            "description": null,
            "occurrence": null,
            "rating": null
        }
    ]
}
```

We can see each correlation has a lot of different properties including the sentence describing it, the `attribute`s in question, and some values for the strength as a `percentage`, and the confidence in `stars`, where 5 stars is the most confident.

Let's say that, given the name of an attribute, we'd like to retrieve and display a set of its most confident correlations — that is, those least likely to be incorrect or just random chance. We can filter this endpoint by using `confident=1` and `attribute=[name]` to get a list of correlations that fit what we're after.

=== "show_correlations.py"

    ```python
    import requests
    import datetime

    TOKEN = "[your_token]"

    def get_correlations(name):
        url = "https://exist.io/api/2/correlations/"

        response = requests.get(url, params={'confident':1, 'attribute':name, 'limit':10}, 
                                headers={'Authorization':f'Token {TOKEN}'})

        if response.status_code == 200:
            try:
                data = response.json()
                correlations = [] # make an array to hold our results
                
                for obj in data['results']: # loop over each correlation object
                    correlation = {} # make a new object to hold only the fields we want
                    correlation['text'] = obj['second_person']
                    correlation['percentage'] = int(obj['percentage']) # round this to an integer
                    correlations.append(correlation)
                
                return correlations
            except:
                # we assume all the fields we need are present
                # but if they're not, an exception will be thrown
                # so let's handle it very generally.
                print("Couldn't get correlations")
        else:
            print("Error!", response.content)
    

    # when we run the script, execution begins here 
    name = input("Attribute: ")

    correlations = get_correlations(name)
    if correlations:
        for c in correlations:
            print(f"{c['text']} ({c['percentage']}%)")
    ```

If we save that script and *[audience chants the catch phrase]* Insert! Our! Token!, and run it with `python3 show_correlations.py`, we'll get something like this:

```
Attribute: caffeinated_drink
you go to bed later on days after you tag 'caffeinated drink' more. (29%)
you tag 'late to sleep' more on days after you tag 'caffeinated drink' more. (28%)
you tag 'caffeinated drink' more when it's the weekend. (19%)
you tag 'bad sleep' more on days after you tag 'caffeinated drink' more. (17%)
you tag 'mowing' more when you tag 'caffeinated drink' more. (14%)
you have a better day when you tag 'caffeinated drink' more. (12%)
```

You can see that I really need to avoid caffeine if I want to get to sleep at a decent hour. But it does make my mood better, so what's a guy to do?

So now we have a way of finding and displaying a list correlations for an attribute.


### Finding a particular correlation

Let's say we'd like to find the relationship between two known attributes — that is, we're looking for one specific correlation. It might not exist, because not everything we track is related to everything else. For example, there may be no relationship between how many tracks you listen to and how long you're asleep, because you listen to a similar amount every day. But that's something we can uncover by asking for a correlation that's a specific combination of attributes, and seeing what we get back.

In this instance we'll `GET` the `https://exist.io/api/2/correlations/combo/` endpoint, passing `attribute` and `attribute2` parameters (in any order). If there's no result, we'll get a  HTTP 404 (Not Found) status code, so we'll need to handle that too.

=== "find_correlation.py"

    ```python
    import requests

    TOKEN = "[your_token]"

    def get_correlation(name1, name2):
        url = "https://exist.io/api/2/correlations/combo/"

        response = requests.get(url, params={'attribute':name1, 'attribute2':name2},
                                headers={'Authorization':f'Token {TOKEN}'})
        if response.status_code == 404:
            return None
        
        if response.status_code == 200:
            return response.json() # our json is just the correlation object
        else:
            print("Error!", response.content)

    
    # execution begins here
    name1 = input("Attribute 1: ")
    name2 = input("Attribute 2: ")

    correlation = get_correlation(name1, name2)
    if correlation:
        # we got a result
        text = correlation['second_person']
        percentage = int(correlation['percentage']) # round this to an integer
        print(f"{text} ({percentage}%)")
    else:
        # we didn't
        print("No correlation found.")
    ```

This is pretty simple, right? We can save this file, insert our own token for `TOKEN`, run it with `python3 find_correlation.py`, and we'll see an interface for finding a correlation:

```
Attribute 1: mood
Attribute 2: walk
you have a better day when you tag 'walk' more. (24%)
```


At this point, we've covered all of the essentials for reading core data types from Exist using a personal token. I hope you've found it useful!

[Part two: making a write client :material-arrow-right:](/guide/write_client/)
