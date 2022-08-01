---
title: Best practices
---

TL;DR:

1. Don't acquire attributes every time you write
2. Don't refresh tokens every time you write
3. Don't write more often than necessary
4. Limit your scopes to only what you need

##  Acquiring attributes

There's no need to acquire an attribute each time you want to write to it. Assume that you already own the attribute and only reactively acquire it again if you get an error that you don't.

## Refreshing tokens

OAuth2 access tokens expire in a year. Please don't refresh your token each time you make a call, as this is wasteful and unnecessary.

[Read tokens](/reference/authentication/token/) never expire and so don't need to be requested again once you have one.


## Limit your scopes

Scopes are fine-grained to read- and write-access per group so that clients can ask for only what they need, and users can give out the minimum access to their data that's necessary. Please ask for the minimum scopes you require in order for your client to function.


## Scheduling regular updates

You'll probably use something like `cron` to schedule regular calls to your program, but how you do that is outside the scope of this guide.

Use only what you need — consider how regularly you need this data, or how up-to-date it needs to be, and don't schedule your update to run any more frequently than this. **Don't make API calls too frequently.** We don't have unlimited server capacity.

Most official integrations in Exist only update attribute data every hour, and some are less often than that. Manually-entered data can update more frequently depending on how you use Exist, but a good rule of thumb is that, if you're syncing data periodically, making a request **once an hour is enough**.

!!! note
    Don't run your script more often than you need to. Once an hour at most is enough for periodic updates.

If you're using the increment endpoints to react to events, then this rule may be broken, as you'll be making a call each time the event triggers it.

Of course, if you don't need data to be up-to-date throughout the day, or you don't have useful data until the end of the day, running your update *once a day* should be enough. 

We've had some folks making automated API calls to retrieve their data on every minute of every hour, and there's just no good reason for that. Please don't do it.
