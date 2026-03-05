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
   ```http
   GET /my-account?id=wiener HTTP/2
