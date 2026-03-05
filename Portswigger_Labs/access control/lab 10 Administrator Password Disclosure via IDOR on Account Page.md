# Lab 10: Administrator Password Disclosure via IDOR on Account Page

## Severity
Critical

## Affected Component
User account management endpoint (`/my-account`)

## Vulnerability Summary
The application exposes user account details based solely on a user-controlled request parameter without enforcing authorization checks. By modifying the `id` parameter, an authenticated low-privileged user can access the administrator’s account page.


The administrator password is included in the HTTP response body (masked in the UI but present in the source), resulting in direct credential disclosure and full administrative account compromise.

## Business Impact
An authenticated attacker can:
* Retrieve the administrator’s plaintext password
* Fully compromise the administrator account
* Perform all privileged actions, including user deletion
* Achieve complete application takeover

This vulnerability represents a total failure of access control and credential handling.

## Steps to Reproduce
1. Authenticate as a normal user (e.g., `wiener`).
2. Intercept the account page request:
   `GET /my-account?id=wiener HTTP/2`
3. Modify the `id` parameter to reference the administrator account:
   `GET /my-account?id=administrator HTTP/2`
4. Send the request and inspect the response source.

## Proof of Concept
The response contains the administrator’s password value in the HTML source (masked in the UI but visible in the response body). Using the extracted password, the attacker can authenticate as the administrator and delete user `carlos`.

## Root Cause Analysis
The backend application:
* Uses a client-supplied identifier to determine which account data to return
* Fails to validate ownership against the authenticated session
* Stores and returns sensitive credentials in a retrievable format

This results in an insecure direct object reference combined with insecure credential handling.

## Recommendation / Mitigation
* Never return passwords or sensitive credentials in responses.
* Store passwords using strong one-way hashing algorithms.
* Enforce strict object-level authorization for all account endpoints.
* Derive user identity exclusively from the authenticated session.
* Perform a full audit for similar credential exposure issues.
