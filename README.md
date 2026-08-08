# Google OAuth 2.0 in Postman: Mini Case Study

## Overview
This project documents how I configured and debugged a Google OAuth 2.0 flow in Postman so I could make authenticated API requests with a Bearer token instead of relying on manual credentials. The final result was a working Authorization Code flow where Google returned an authorization code to Postman, Postman exchanged it for tokens, and the collection was ready to call the API. [1][2]

## Goal
The goal was to connect Postman to a Google API using OAuth 2.0 with the correct scope, the correct redirect URI, and a valid user account that was authorized to test the app. Because OAuth is permission-based, the request only works when the client configuration, redirect URI, consent screen, and approved user all line up correctly. [1][3]

## Setup
I configured Postman to use the **Authorization Code** grant type with Google's authorization endpoint and token endpoint, then supplied the client ID, client secret, redirect URI, and requested scope. Postman's web app uses `https://oauth.pstmn.io/v1/browser-callback` as its browser callback, so that exact value must be used when testing from the web version of Postman. [2][4][5]

**Screenshot 1:** `images/01-postman-oauth-config.png` — Postman OAuth configuration screen.

## Issue 1: Redirect URI mismatch
The first failure was `redirect_uri_mismatch`, which means the redirect URI sent in the request did not exactly match an authorized redirect URI in the Google Cloud OAuth client settings. Google treats the redirect URI as part of the security boundary, so even a small mismatch prevents the authorization server from returning the code. [6][7]

**What I changed:** I opened the OAuth client in Google Cloud Console and added Postman's browser callback URI exactly as required. [2]

**Screenshot 2:** `images/02-oauth-errors-and-fix.png` — Either the `redirect_uri_mismatch` screen or the Google Cloud redirect URI setting.

## Issue 2: Access denied in testing mode
After the redirect URI was fixed, the flow moved forward, but Google then returned `access_denied` because the app was still in **Testing** mode. Google documents that External apps in testing can only be accessed by users explicitly added under **Test users** on the OAuth consent screen. [3][8]

**What I changed:** I added the Google account I wanted to use as a test user on the OAuth consent screen. [3][8]

## Working result
Once both configuration issues were fixed, the OAuth flow completed successfully: sign in with Google, approve the requested permissions, return to Postman with an authorization code, and exchange that code for tokens. At that point, Postman could attach the access token to requests and make authenticated API calls. [1][2]

**Screenshot 3:** `images/03-working-token-or-request.png` — Final success state, either the token creation screen or a successful authenticated request.

## What this shows
This exercise demonstrates more than just getting a request to work. It shows I understand how OAuth depends on exact redirect URI matching, consent screen configuration, testing restrictions, and scoped permissions before an API client can act on a user's behalf. [1][3][6]

## OAuth terms in plain English

### Authorization code grant
The authorization code grant is an OAuth 2.0 flow where the user signs in and approves access first, then the client receives a short-lived authorization code. The client exchanges that code at the token endpoint for tokens, which is safer than returning the final token directly through the browser. [1][9]

### Access token
An access token is the credential the client includes with API requests to prove it has permission to access protected resources. OAuth 2.0 defines access tokens as limited by scope and lifetime, which is why they usually expire after a short period. [1][10]

### Refresh token
A refresh token is a longer-lived credential the client can send to the authorization server to obtain a new access token after the old one expires. RFC 6749 says a refresh request cannot ask for broader scope than the user originally granted. [1][11]

### Scope
A scope is the set of permissions the client is requesting, such as read-only access to a specific API. Scope limits what the issued token can do, which is why users see the requested permissions during consent. [1][12]

## Suggested repo structure
```text
project/
├── README.md
└── images/
    ├── 01-postman-oauth-config.png
    ├── 02-oauth-errors-and-fix.png
    └── 03-working-token-or-request.png
```

## Interview value
In an interview, this project demonstrates both implementation and understanding: I can configure OAuth in a real client, diagnose common Google OAuth failures, and explain why the flow works once the redirect URI, test-user access, and requested scope are configured correctly. That makes the project stronger than a simple “it works” demo because it shows troubleshooting and security awareness. [1][3][2]
