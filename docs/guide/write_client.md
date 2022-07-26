
In this section we'll make an API client app and write some Python scripts to send data to Exist for our attributes. Along the way we'll learn about OAuth2 clients and the authorisation process, authenticating ourselves, creating and acquiring attributes, and the main ways of interacting with the Exist API to write data. This should build a good foundation for you to use the Exist API to manage the data for an attribute, perhaps as a way of sending data on triggers from other events, or as a regular automated syncing process from another data source.

Our code examples will be in Python, and you're encouraged to copy and paste them, save them, and run them yourself! I'm using Python 3.8, but Python 3.6 or newer should be fine. You'll need to make sure you have the `requests` library [installed](https://requests.readthedocs.io/en/latest/user/install/).

You're not required to have read the [read client](/guide/read_client/) section, but here we'll only be dealing with *writing* data, so you may want to consult that section first in order to understand the structure of the data Exist provides.


## Creating a client

To be able to write data, we first need a way to interact with the API on behalf of users. For a start, that'll just be ourselves, but we may want act on behalf of many different users. Let's say we were the owners of a service called Greatreads, which tracks book-reading, and we wanted to allow our Greatreads users to connect their account to Exist and update the `pages_read` attribute based on what they'd entered in our app. In the same way that connecting a service like Fitbit to Exist requires logging in to Fitbit and approving Exist to read our data, we'd need a way to represent ourselves as the service "Greatreads" to Exist users, allowing them to give us permission to access their Exist account and write data to it. A client app is the answer, so let's make one.

On your [account page](https://exist.io/account/) in Exist you'll see [Developer apps](https://exist.io/account/apps/) in the sidebar menu. Let's open this page. If you haven't made a client app before, you'll see an empty list, but once you have your own apps, they'll be listed here. Let's choose to "Add a new client" where we'll see the form for creating a new client app.

![The "create new app" form](/img/create_app.png)

Go ahead and fill in the details for your app. We won't be sharing this with anyone else right now, so the public details don't really matter. Name it whatever you like. The important details to enter are `http://localhost:8000/` for our Redirect URI, and "Confidential" for our client type. Submit the form and view the details for your first client app!

![The new client details](/img/new_client.png)

The client ID and secret are important values that we'll be using in our code. You'll also see a token has been created for us:

![The developer token](/img/token.png)

A user access token is a way of authenticating ourselves to Exist on behalf of a particular user. In this case, Exist has created us a personal user token to get started with, with permissions to read and write all attributes, so that we don't need to go through the OAuth2 authorisation flow before getting our own data. Let's not worry about this for now, because we'd like to learn about the full flow, so instead of skipping it we'll jump into it.

## Authenticating a user with OAuth2

Users give client apps access to their accounts by completing the OAuth2 authorisation flow. For clients, this involves sending the user to a URL on the Exist site filled with some relevant client details, receiving a code that signals user assent, and exchanging this code for a long-lived user token. For users, this involves making sure they're logged in to Exist, following this URL, and then approving the permissions the client has asked for by clicking "allow". At the end, users can use the client to read and/or write the attributes it has access to, and clients can perform these read/write operations on users' behalf.

### Choosing scopes

A *scope* represents a certain permission or area of access to data. In the case of Exist, we have *read scopes* and *write scopes* that correspond to the groups attributes live in. For example, if we ask Exist for the `activity_read` scope, we're asking for the ability to read attribute, correlation, average, and insight data related to any attribute the user has in the Activity group. If we asked for the `activity_write` scope, we wouldn't be able to read any of this data, but we could create, own, and write new values for any attribute the user has in Activity.

Scopes are broken down in this way so that clients can ask only for the access they need, and users can be safe in the knowledge that clients aren't reading or writing any other attributes. So a responsible client should ask **only** for the scopes it really needs in order to function correctly. For example, for Greatreads, we'd ask solely for the `media_write` scope so we can acquire and write to the `pages_read` attribute. The list of requested scopes is presented on the authorisation page for the user to agree to, so the user would see these permissions listed:

* Read your profile details (not including email address)
* Write Media group: Create new attributes and write data to attributes in the group

All clients can see a user's profile details without asking for this scope specifically, so that they can address the user and present user data in the correct units and so on. So these are the two permissions the user agrees to.

You can find a full list of scopes [in the OAuth2 reference](/references/authentication/oauth2/#scopes).

### The authorisation flow

Okay, now we start getting into some code. Let's go over the list of steps required to authorise ourselves with a user:

1. We send the user to the "request authorisation" page at `/oauth2/authorize` on the Exist site
2. The user reviews the permissions and clicks "Allow", which redirects them back to a URL we specify
3. We take the code we received in step 2 and exchange it for a user token by making a call to `/oauth2/access_token`

That's it! So for the first step, what we need is the ability to create the correct URL to give the user. We know the page, but we need to include a lot of parameters:

*  `response_type=code` to request an auth code in return   
*  `redirect_uri` with the URL to which Exist returns the user
*  `scope` with the list of scopes we're asking for
*  `client_id` which is our OAuth2 client ID

Let's keep going with our Greatreads example and ask for the `media_write` scope only. Remember that when we created our client app, we set our redirect URI to `http://localhost:8000/`, so we'll need that same value here.

=== "request_authorisation.py"

    ```python
    from urllib.parse import urlencode

    CLIENT_ID = "[your_id]"
    URL = 'https://exist.io/oauth2/authorize'

    # the parameters we'll be sending
    params = {'client_id': CLIENT_ID, 
              'response_type':'code', 
              'redirect_uri':'http://localhost:8000/',
              'scope':'media_write',
             }
    
    # let's encode them appropriately for a URL
    querystring = urlencode(params)
    # and finally print the complete result to the terminal
    print(f"{URL}?{querystring}")
    ```

Copy this, save it to `request_authorisation.py`, and add your own client ID to `CLIENT_ID`. Run it with `python3 request_authorisation.py` and you'll see the URL in your terminal (something like, but not the same as, this example):

```shell
https://exist.io/oauth2/authorize?client_id=1234abcdef&response_type=code&redirect_uri=http%3A%2F%2Flocalhost%3A8000%2F&scope=media_write
```

This is the URL we'd give to any user who wanted to connect our Greatreads app to their Exist account. If we follow this URL in our own browser (if you can't click it, copy and paste it in), we'll be presented with the authorisation page:

![The authorisation page](/img/authorisation.png)

Keep this tab open but don't click "Allow" yet! Exist wants to send a request back to us at `http://localhost:8000/`, and we need to be listening on this port to receive it. That's what we'll write next.

=== "receive_authorisation.py"

    ```python
    from http.server import HTTPServer, BaseHTTPRequestHandler
    from urllib.parse import urlparse, parse_qs
    import requests

    CLIENT_ID = ''
    CLIENT_SECRET = ''
    REDIRECT_URI = ''

    # create a class to handle our http request
    class Handler(BaseHTTPRequestHandler):
        def do_GET(self):
            # parse the path of this request into its parts
            parts = urlparse(self.path)
            # then parse the query parameters into a dict
            query = parse_qs(parts.query)
            # and then get the code we need out of the dict
            code = query['code'][0]
            
            # tell the browser it worked
            self.send_response(200)
            self.wfile.write(b'OK!\n')
            # then get our access token
            self.get_token(code)

        def get_token(self, code):
            # make our request using our new code, and some other client details
            response = requests.post('https://exist.io/oauth2/access_token', {
                'grant_type':'authorization_code',
                'code': code,
                'client_id': CLIENT_ID,
                'client_secret': CLIENT_SECRET,
                'redirect_uri': REDIRECT_URI,
            })
            # parse the response into json
            data = response.json()
            print('Access token:', data['access_token'])
            print('Refresh token:', data['refresh_token'])

    
    # create a http server and listen for one request only
    server_address = ('127.0.0.1', 8000)
    httpd = HTTPServer(server_address, Handler)
    httpd.handle_request()
    httpd.server_close()

    ```

Save this as `receive_authorisation.py`, making sure you add your own values for the client ID, secret, and redirect URI. Then run it with `python3 receive_authorisation.py`, which will start listening for HTTP requests on port 8000 on our local machine. Remember, this is where we told Exist to send the user when we defined our `redirect_uri`. 

Listening for requests means we're ready to go, so hit "Authorise" in the browser. Your browser gets redirected to `http://localhost:8000/?code=[code]`, and our python script receives this request from the browser, grabs that code, and sends it to Exist to exchange for an access token. 

So if all goes well, you'll see "OK!" in the browser, and your tokens in the terminal:

```
Access token: 8f65bce0f9fdde00b88c7cf25f29b06be43f6ee5
Refresh token: 8c7cf25f29b06be43f6ee58f65bce0f9fdde00b8
```

We've done it. We send the user to the authorisation page on Exist, received the request containing the code, and exchanged the code for a token. Now, once our code was running on a publicly accessible server (not `localhost`), we could successfully authenticate any user.


### Refreshing a token


## Acquiring attributes

Let's look at the flow of acquiring and using `template` vs `name` a little more, because the rules here are tricky. 

As an example, let's imagine we're writing an official integration for a step tracker app we invented, so we'd like to provide `steps` data. If we want to acquire the templated attribute `steps`, for example, we would use the `template` key, because what we want is explicitly the attribute that's an integer quantity of steps walked each day. If the user doesn't have this attribute, Exist will create it for the user and give its ownership to us. But imagine that we had erroneously used the `name` key, for example `"name": "steps"`, and the user had instead already created a manual attribute they've called `steps` which is a percentage type, for some reason. We would expect to start filling it with integer counts for each day, but we can't — it's the wrong type! This endpoint gives us an explicit choice about what we want so we can prevent this.

On the other hand, let's imagine we're writing a client for updating manual attributes, whatever they are. We know the user has created an attribute called `steps`, and we don't care what it is, because we just want to make a client interface for every manual attribute of any type. In this case, we'd acquire it with `name`, to make explicit that we're okay with non-templated attributes.

## Creating a new attribute


## Writing data

### Writing totals

### Reacting to events with the increment endpoint

## Scheduling regular updates


[Part three: sharing your integration :material-arrow-right:](/guide/integration/)
