# Lab 5: Improper Access Control via X-Original-URL Header Injection

## Severity
High

## Affected Component
URL-based access control mechanism protecting administrative endpoints (`/admin`)

## Vulnerability Summary
The application implements URL-based access control at the frontend layer to restrict unauthenticated access to administrative endpoints such as `/admin`. However, the backend server incorrectly trusts the `X-Original-URL` HTTP header to determine the requested resource.


By supplying a crafted `X-Original-URL` header, an unauthenticated attacker can bypass access controls and directly invoke protected backend routes, resulting in full administrative access.

## Business Impact
An unauthenticated attacker can:
* Access the administrative control panel
* Perform privileged actions such as deleting user accounts
* Fully compromise application integrity and user trust

In a real-world environment, this could lead to account takeover, data loss, or service disruption.

## Steps to Reproduce
1. Intercept a request to the application’s home page using Burp Suite.
2. Modify the request to include the following header:
   `X-Original-URL: /admin`
3. Observe that the response returns `200 OK` and renders the administrative interface, despite the user being unauthenticated.
4. Inspect the admin panel source and identify a deletion endpoint:
   `/admin/delete?username=carlos`
5. Craft the following request:
   `GET /?username=carlos HTTP/2`
   `X-Original-URL: /admin/delete`
6. Send the request.

## Proof of Concept
The above request successfully deletes the user `carlos` without authentication, demonstrating a complete access control bypass.

## Root Cause Analysis
The frontend enforces access restrictions based on the visible URL path, while the backend processes the request based on the `X-Original-URL` header. This inconsistency allows attackers to supply a trusted internal path via headers while appearing to request a benign endpoint externally.

The backend fails to revalidate authorization for the resolved internal route.

## Recommendation / Mitigation
* Do not trust client-supplied headers such as `X-Original-URL` or `X-Rewrite-URL` for access control decisions.
* Enforce authorization checks server-side, independent of routing logic.
* Strip or overwrite sensitive routing headers at the edge (reverse proxy / load balancer).
* Ensure consistent access control logic across all request-processing layers.
