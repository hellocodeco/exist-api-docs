
## URLs and endpoints

The new API lives at `https://exist.io/api/2/` — note the 2 in the URL. URLs for resources have also changed, and providing the username or `$self` in the URL is now unnecessary. Most API endpoints that return lists now return paged responses.

## Scopes

API v2 has new [fine-grained scopes](/reference/authentication/oauth2/#scopes) for reading and writing individual groups. The `read`, `write`, and `append` scopes are not valid for any v2 endpoints, and existing access tokens with only these scopes will not be able to read or write data using the v2 endpoints. Users must reauthorise your app through the OAuth2 flow in order to generate a token with the correct scopes.


## Custom tags

Previously in API v1, custom tags were updated in total by writing a string to the virtual attribute `custom`. We also provided an `attributes/custom/append/` endpoint for adding a single tag to a day without managing the state for all other tags. These processes have changed.

In API v2, custom tags are managed as individual boolean attributes. Tags must be acquired and written to individually, the same as any other attribute. Remember that the [update endpoint](/reference/writing_data/#update-attribute-values) accepts multiple updates, so writing a value for many tags at once can be done with a single call. Reading tags requires the `custom_read` scope and writing requires `custom_write`.

To write a positive value for a single tag, emulating the append behaviour, you can write a value of `1` for the tag via the `update` endpoint.

To write a set of tag values for a day, use the `update` endpoint to send an array of updates, where each `name` is the tag name and each value is `1` or `0` as required.


## Incrementing

The `append` endpoint for custom tags in API v1 has been generalised to the `attributes/increment/` endpoint in API v2. Now, rather than storing and sending the total value for an attribute for each day, you can [send incremental values](/reference/writing_data/#increment-attribute-values) that will be applied to the current value. This is useful for reacting to events, for example sending a `1` increment each time an event occurs, or sending the duration of a recently completed workout for `workout_min` without needing to find the total of all workouts for the day. [Read more in the guide.](/guide/write_client/#reacting-to-events-with-the-increment-endpoint)
