---
title: Getting started
---

Hello! This guide will help you to become familiar with what's required to read and write your Exist data. We expect you to already be somewhat familiar with programming, HTTP requests and responses, and JSON data. Example code will use Python.

So firstly, the API lives at `https://exist.io/api/2/`. All requests must use HTTPS.

Your POST bodies can be sent as JSON (`application/json`) or form data (`application/x-www-form-urlencoded` or `multipart/form-data`). However the API will only return JSON.

Requests are currently rate-limited at 300/hr per user token. If you send more than 300 requests in a single 60-minute period, you'll get hit with a `429 Too Many Requests` rate-limiting response. This means you're going too hard, and you need to ease off our poor servers.

We have a few different methods of authenticating your requests so you can read and write data. We'll start with simple token authentication, which is read-only, and take a look at how to read your data. Later, we'll look at how to make a write-client that can authenticate many users with OAuth2, create attributes for them, and update their data. Read on!

[Part one: reading data :material-arrow-right:](/guide/read_client/)
