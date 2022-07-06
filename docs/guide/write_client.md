
In this section we'll make an API client app and write some Python scripts to send data to Exist for our attributes. Along the way we'll learn about OAuth2 clients and the authorisation process, authenticating ourselves, creating and acquiring attributes, and the main ways of interacting with the Exist API to write data. This should build a good foundation for you to use the Exist API to manage the data for an attribute, perhaps as a way of sending data on triggers from other events, or as a regular automated syncing process from another data source.

Our code examples will be in Python, and you're encouraged to copy and paste them, save them, and run them yourself! I'm using Python 3.8, but Python 3.6 or newer should be fine.

You're not required to have read the [read client](/guide/read_client/) section, but here we'll only be dealing with *writing* data, so you may want to consult that section first in order to understand the structure of the data Exist provides.


## Creating a client

To be able to write data, we first need a way to interact with the API on behalf of users. For a start, that'll just be ourselves, but we may want act on behalf of many different users. Let's say we were the owners of a service called Greatreads, which tracks book-reading, and we wanted to allow our Greatreads users to connect their account to Exist and update the `pages_read` attribute based on what they'd entered in our app. In the same way that connecting a service like Fitbit to Exist requires logging in to Fitbit and approving Exist to read our data, we'd need a way to represent ourselves as the service "Greatreads" to Exist users, allowing them to give us permission to access their Exist account and write data to it. A client app is the answer, so let's make one.

On your [account page](https://exist.io/account/) in Exist you'll see [Developer apps](https://exist.io/account/apps/) in the sidebar menu. Let's open this page. If you haven't made a client app before, you'll see an empty list, but once you have your own apps, they'll be listed here. Let's choose to [Add a new client](https://exist.io/account/apps/edit/) where we'll see the form for creating a new client app.

![The "create new app" form](/img/create_app.png)

Go ahead and fill in the details for your app. We won't be sharing this with anyone else right now, so the public details don't really matter. Name it whatever you like. The important details to enter are `http://localhost:8000/` for our Redirect URI, and "Confidential" for our client type. Submit the form and view the details for your first client app!

![The new client details](/img/new_client.png)

The client ID and secret are important values that we'll be using in our code. You'll also see a token has been created for us:

![The developer token](/img/token.png)

A user access token is a way of authenticating ourselves to Exist on behalf of a particular user. In this case, Exist has created us a personal user token to get started with, with permissions to read and write all attributes, so that we don't need to go through the OAuth2 authorisation flow. Let's not worry about this for now, because we'd like to learn about the full flow, so instead of skipping it we'll jump into it.

## Authenticating a user with OAuth2

### Choosing scopes

### The authorisation flow

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
