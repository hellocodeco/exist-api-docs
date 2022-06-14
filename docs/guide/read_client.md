
In this section of the guide we'll build a read-only client, capable of retrieving personal data from an Exist account and printing it to the terminal. I'm using Python 3.8, but Python 3.6 or newer should be fine.

## Getting a token

To be able to read our own personal data from Exist, we need to authenticate ourselves, so Exist knows who we are. Let's start with the simplest method of authentication, which Exist calls *simple token authentication*. We'll ask for a unique token that identifies us, and then add it to each subsequent request so Exist can use it to know who we are.

Let's make a request to the simple token endpoint, using the `requests` library, exchanging our username and password for a token we can use to authenticate ourselves with the API. 

The endpoint at `https://exist.io/api/1/auth/simple-token/` takes a `username` and `password` and returns a JSON object containing a `token` field.

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

We can save this script as `get_token.py`, run it with `python get_token.py`, and interactively enter our username and password. The script then prints out our token, and we can save this to use in future scripts. This token won't expire so we can hard-code it into personal scripts with impunity. 

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

So by this point we know how to print a list of attributes and their values, a list of tags used for a day, and how to request a specific date!

## Getting averages

### Showing long-term trends

## Showing correlations

### Finding a particular correlation

## Scheduling regular updates

[Part two: writing data :material-arrow-right:](/guide/write_client/)
