---
title: Important values and URLs
---


Name | Value
-----------------| ------
**API base URL**     | `https://exist.io/api/2/`
**OAuth2 base URL**  | `https://exist.io/oauth2/`
**Response type**| `application/json`
**Rate-limiting**| 300 requests/hr per user token
**POSTing data** | `application/x-www-form-urlencoded` (the usual) or send the body as `application/json`
**OAuth2 auth header** | `Authorization: Bearer [tokenxyz]`
**Simple token auth header** | `Authorization: Token [tokenabc]`
**Getting a simple token** | `POST  'username' and 'password' to https://exist.io/api/1/auth/simple-token/`
**Testing your token** | `GET https://exist.io/api/2/accounts/profile/`
**See some JSON in your browser** | [Your attributes with today's values](https://exist.io/api/2/attributes/with-values/)

