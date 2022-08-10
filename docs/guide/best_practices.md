---
title: Best practices
---

TL;DR:

1. Don't acquire attributes every time you write
2. Don't refresh tokens every time you write
3. Limit your scopes to only what you need
4. Batch your writes into fewer calls where possible
5. Don't write more often than necessary


##  Acquire attributes once

Don't forget that you cannot write to an attribute unless you own it, so you must first call to [acquire it](/reference/attribute_ownership/).

*However*, there's no need to acquire an attribute each time you want to write to it. Perform attribute creation and acquisition once when the user first connects your app, and thereafter assume that you already own the attribute. Only reactively acquire it again if you get an error that you don't.

## Refresh tokens when they expire

[OAuth2 access tokens](/reference/authentication/oauth2/) expire in a year. Please don't refresh your token each time you make a call, as this is wasteful and unnecessary.

Read tokens from our [token-based authentication](/reference/authentication/token/) never expire and so don't need to be requested again once you have one.


## Limit your scopes

[Scopes](/reference/authentication/oauth2/#scopes) are fine-grained to read- and write-access per group so that clients can ask for only what they need, and users can give out the minimum access to their data that's necessary. Please ask for the minimum scopes you require in order for your client to function.


## Batch your writes together

The [acquire, create, update, and increment write endpoints](/reference/writing_data/) all accept arrays of objects, meaning that you can avoid making multiple calls to Exist by combining multiple updates into one call. For example, if you're updating 3 days' worth of data for one attribute for a user, this can be one call to `attributes/update/` containing three updates with different dates. Likewise, you can also combine updates across different attributes as long as they're all for the same user. This is usually much faster than multiple calls and avoids hitting rate limits.


## Scheduling regular updates

You'll probably use something like `cron` to schedule regular calls to sync data to Exist, but how you do that is outside the scope of this guide.

Use only what you need — consider how regularly you need this data, or how up-to-date it needs to be, and don't schedule your update to run any more frequently than this. **Don't make API calls too frequently.** We don't have unlimited server capacity.

Most official integrations in Exist only update attribute data every hour, and some are less often than that. Manually-entered data can update more frequently depending on how you use Exist, but a good rule of thumb is that, if you're syncing data periodically, making a request **once an hour is enough**.

!!! note
    Don't run your script more often than you need to. Once an hour at most is enough for periodic updates.


Of course, if you don't need data to be up-to-date throughout the day, or you don't have useful data until the end of the day, running your update *once a day* should be enough. 

If you're using the increment endpoints to react to events, then this rule may be broken, as you'll be making a call each time the event triggers it.

If you're sending totals for each day, you should probably sync yesterday's values regularly, as well as the current value for today, to ensure that you have the most up-to-date value for the full 24-hour period.

We've had some folks making automated API calls to update their data on every minute of every hour, and there's just no good reason for that. Please don't do it.
