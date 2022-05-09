---
title: Overview
---

There are two authentication methods — simple tokens, and OAuth2 clients. So which one do you need?

**Simple token authentication** is read-only and exists as a basic means for users to access their own data from Exist. This method is available to everyone
by exchanging a username and password for a token that doesn't expire. This is only recommended for quickly building a single-user, read-only client,
and may be deprecated in future.

**OAuth2 clients** are superior to simple-token authentication as they can acquire control of attributes and write values for attributes.
One client application can create and use access tokens for many users. One user may have many clients authorised to access their Exist account, each with a separate token that can be revoked.

OAuth2 clients can be created by all users. You can create one from your [app management page](https://exist.io/account/apps/) within your Exist account.

