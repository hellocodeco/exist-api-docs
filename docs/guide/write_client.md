

## Creating a client

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
