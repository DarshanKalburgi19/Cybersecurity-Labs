# Lab 13: Privilege Escalation via Reference-Based Access Control

## Severity
High

## Affected Component
Administrative role management endpoint (`/admin-roles`)

## Vulnerability Summary
The application performs authorization decisions based on user-supplied object references (e.g., username parameter) rather than validating the privileges of the authenticated user. 


As a result, a normal user can manipulate references in privileged endpoints to escalate their own privileges.

## Business Impact
An authenticated attacker can:
* Promote their own account to administrator status
* Gain unauthorized access to administrative functions
* Compromise the integrity of the application's user management system

## Steps to Reproduce
1. Authenticate as a normal user (e.g., `wiener`).
2. Intercept an administrator role-management request:
   `POST /admin-roles`
   `username=carlos&action=upgrade`
3. Modify the request to target your own username:
   `username=wiener&action=upgrade`
4. Send the request using the normal user’s session.

## Proof of Concept
The normal user is successfully promoted to administrator, gaining full administrative privileges despite the request being initiated from a low-privileged session.

## Root Cause Analysis
Authorization logic relies on client-controlled references rather than enforcing role checks based on the authenticated session. This results in a failure to verify if the requester has the authority to perform the action specified in the request.

## Recommendation / Mitigation
* Enforce authorization using server-side identity and role verification.
* Never trust object references provided by the client for sensitive operations.
* Bind authorization decisions strictly to the authenticated user context.
* Implement centralized access control checks for all privileged endpoints.
