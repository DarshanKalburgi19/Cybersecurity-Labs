# Lab 9: Sensitive Data Exposure via Authorization Bypass in Redirect Response

## Severity
High

## Affected Component
User account endpoint (`/my-account`)

## Vulnerability Summary
The application returns user-specific sensitive information in the HTTP response body before enforcing access control. Although unauthorized requests are followed by a redirect, the sensitive data is already included in the response.


By modifying a user-controlled request parameter, an authenticated attacker can retrieve another user’s private account data, resulting in an access control bypass with sensitive data exposure.

## Business Impact
An authenticated attacker can:
* Access other users’ private account data
* Extract sensitive information such as API keys
* Harvest data silently through intercepted responses, even when redirects occur

In real-world systems, this could lead to token theft, account compromise, and privacy violations.

## Steps to Reproduce
1. Authenticate as a normal user (e.g., `wiener`).
2. Intercept the account request:
   `GET /my-account?id=wiener HTTP/2`
3. Modify the `id` parameter to reference another user:
   `GET /my-account?id=carlos HTTP/2`
4. Send the request and inspect the response body.

## Proof of Concept
The response contains `carlos’` sensitive account data, including his API key, even though the application attempts to redirect the user afterward. The sensitive information is visible in the raw response body before the browser follows the `302 Found` location header.

## Root Cause Analysis
The backend generates and returns sensitive user data prior to performing authorization checks. While the application attempts to mitigate unauthorized access using redirects, this occurs only after the sensitive data has already been included in the HTTP response.

This reflects a failure in enforcing access control at the correct stage of request processing.

## Recommendation / Mitigation
* Perform authorization checks before retrieving or rendering sensitive data.
* Never include sensitive information in responses to unauthorized requests, even temporarily.
* Ensure redirects are used only after access control has been properly enforced.
* Implement centralized authorization middleware for user-specific endpoints.
