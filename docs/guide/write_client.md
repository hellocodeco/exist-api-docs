
## Acquiring attributes

Let's look at the flow of acquiring and using `allow_user_defined` a little more, because the rules here are tricky. 

As an example, let's imagine we're writing an official integration for a step tracker app we invented, so we'd like to provide `steps` data. If we want to acquire the templated attribute `steps`, for example, we wouldn't include `allow_user_defined`, because what we want is explicitly the attribute that's an integer quantity of steps walked each day. If the user doesn't have this attribute, Exist will create it for the user and give its ownership to us. But imagine that we had erroneously included `allow_user_defined=true`, and the user had instead already created a manual attribute they've called `steps` which is a percentage type, for some reason. We would expect to start filling it with integer counts for each day, but we can't — it's the wrong type! So by default this endpoint limits us to templated attributes to prevent this.

On the other hand, let's imagine we're writing a client for updating manual attributes, whatever they are. We know the user has created an attribute called `steps`, and we don't care what it is, because we just want to make a client interface for every manual attribute of any type. In this case, we'd acquire it with `allow_user_defined=true`, to make explicit that we're okay with non-templated attributes.



[Part three: sharing your integration :material-arrow-right:](/guide/integration/)
