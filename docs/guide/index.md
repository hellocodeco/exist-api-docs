---
title: Getting started
---


The API lives at `https://exist.io/api/1/`. All requests must use HTTPS.

POST bodies can be sent as `application/json`, `application/x-www-form-urlencoded`, or `multipart/form-data`.
However the API will only return JSON.

Requests are currently rate-limited at 300/hr per user token. Given user data does not change frequently within
an hourly period (if at all) this should be more than adequate.


OAuth2 is the main means of authorising requests to the API, but if you're building a personal read-only client you might like to use simple token auth.
[Read the authentication overview](#authentication-overview) to see which one you need.

To get stuck in and retrieve some personal data, you should start by [creating a new client app](https://exist.io/account/apps/),
then [get today's overview](#get-current-overview-for-user).

