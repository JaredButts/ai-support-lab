# Debugging Google OAuth 2.0 in Postman: A Real-World Case Study

## Overview
This case study documents my process of configuring and debugging a Google OAuth 2.0
**Authorization Code** flow inside Postman, targeting the Google Calendar API
(`calendar.readonly` scope). Rather than a clean tutorial, this walks through two real
errors I hit and fixed, because debugging is what actually proves understanding of
a protocol, not just a working request.

---

## Step 1: Configuring the OAuth 2.0 Request

![OAuth config](assets/01-oauth-config.png)

I set up a new token in Postman's Authorization tab with:
- **Grant type:** Authorization Code
- **Auth URL:** `https://accounts.google.com/o/oauth2/auth`
- **Access Token URL:** `https://accounts.google.com/o/oauth2/token`
- **Scope:** `https://www.googleapis.com/auth/calendar.readonly`
- **Client Authentication:** Send as Basic Auth header

Clicking "Get new access token" triggers Postman to open Google's Auth URL with these
parameters attached, kicking off the authorization flow.

---

## Step 2: First Failure - redirect_uri_mismatch

![Error 400](assets/02-error-redirect-uri-mismatch.png)

The first attempt failed immediately with **Error 400: redirect_uri_mismatch**. Google
rejected the request because Postman's callback URL (`https://oauth.pstmn.io/v1/browser-callback`)
wasn't registered as an authorized redirect URI on my OAuth client.

**Fix:** In Google Cloud Console -> APIs & Services -> Credentials -> my OAuth Client ID,
I added `https://oauth.pstmn.io/v1/browser-callback` under "Authorized redirect URIs."

**Why this matters:** The redirect URI isn't cosmetic, it's part of OAuth's security
model. Google will only hand an authorization code back to an endpoint it explicitly
trusts, preventing attackers from intercepting codes by pointing the flow at a
malicious callback.

---

## Step 3: Second Failure - access_denied (403)

![Error 403](assets/03-error-access-denied.png)

With the redirect URI fixed, the flow progressed to Google's account chooser, but
selecting an account produced **Error 403: access_denied**, stating the app hadn't
completed Google's verification process and could only be used by approved testers.

**Fix:** In Google Cloud Console -> OAuth consent screen -> Test users, I added my
Google account to the allow-list. Unverified apps in "Testing" publishing status
restrict access to explicitly approved testers as a safeguard.

---

## Step 4: Successful Authorization & Token Retrieval

![Account chooser](assets/04-account-chooser.png)
![Authentication complete](assets/05-auth-complete.png)
![Access token retrieved](assets/06-access-token.png)

With both fixes applied, the full flow completed:
1. Google displayed the account chooser and consent screen
2. I approved the `calendar.readonly` scope
3. Google redirected to Postman's callback with a single-use authorization code
4. Postman silently exchanged that code for an access token behind the scenes
5. The token now populates in Postman's "Current Token" field, ready to authenticate
   requests with a `Bearer` header

---

## Key OAuth 2.0 Concepts

**Authorization Code Grant**
A flow designed for apps where a human logs in and approves access. Rather than
issuing a token directly, the authorization server first returns a short-lived,
single-use *code* to the browser. The app then exchanges that code for a token via a
separate, direct backend call. This two-step handoff matters because the code passes
through the browser (less trusted), while the actual token exchange happens over a
secure, authenticated channel, so the token itself is never exposed in a URL or
browser history.

**Access Token**
The credential an app attaches to every API request (typically as a `Bearer` token in
the Authorization header) to prove it has permission to act on a user's behalf.
Access tokens are intentionally short-lived, often just an hour, to limit the damage
if one is ever leaked.

**Refresh Token**
A longer-lived credential issued alongside the access token, used to obtain a new
access token once the current one expires, without requiring the user to log in and
consent again. If the access token is a day pass, the refresh token is the membership
card that prints a new one whenever needed.

**Scope**
A permission string defining exactly what an access token can do. `calendar.readonly`
only allows reading calendar data, no writing, no deleting. Scopes enforce the
principle of least privilege, and the user sees precisely what they're granting on
the consent screen before approving.

---

## Takeaway
Two separate errors, a redirect URI mismatch and an unverified-app access block, both
stem from Google's OAuth security model treating trust as something that must be
explicitly configured, not assumed. Understanding why each error occurred (not just
copying a fix) is what separates surface-level API usage from actually understanding
the protocol.
