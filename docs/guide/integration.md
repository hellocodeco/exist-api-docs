---
title: Sharing your integration
---

If you've written an integration that others can use to read or write data from Exist, congratulations and thank you! Now you have a few different choices to make as to how to make it available.

Unfortunately we do not have the resources to receive and maintain your code, provide hosting for external integrations, or provide support for your integrations on your behalf.

## Options

### Hosting it yourself

The smoothest experience for your users is not-coincidentally the most effort for you — hosting a web app on your own servers. This allows users to easily connect your app by visiting its URL, authorising it with Exist, and being redirected back to your site to complete the process. Your server-side setup also manages regular syncing to Exist without requiring any further work from the user. Of course, this setup requires that you manage your own infrastructure and provide an ongoing service to users, who will expect everything to keep working and come to you for support if things go wrong.

### Providing a client-side app

Providing an app the user can run on their phone or computer is a nice compromise option that requires less infrastructure for you and still doesn't ask too much of the user. Users can still connect your app to Exist via the regular OAuth2 flow, being redirected back to your app via a custom URL scheme, and thereafter everything should just work. The user's device is responsible for running your app's syncing code regularly in the background, which doesn't require you to execute this step on your own servers, but may be an issue on some platforms like iOS. Consider the tradeoffs if users are required to open the app manually to trigger a sync — this is a poor experience and many users will forget this step.


### Providing the source code

The lowest-effort option is to simply write some scripts in the language of your choice and then make the source code available for users to download and run themselves. This option requires a lot more of the user, who must create their own OAuth2 client in Exist, follow your steps for authenticating, and manage the the process of regular syncing, refreshing tokens, etc., themselves. This sets the bar for technical knowledge much higher, moving much of the onus to the user. However, this is still a valid option in that *any* integration is better than *no* integration. If you're not prepared to commit to the level of support required for one of the other options (which is totally fair!), then this is still a useful way of providing something that a small number of people with the required technical knowledge may be able to benefit from.


## Tell us about it

We love to hear about what you're building with Exist, and our developer forum is the best way to show what you've built to other Exist users too. So please [share your work in our forum](https://forum.exist.io/c/developers/6)!
