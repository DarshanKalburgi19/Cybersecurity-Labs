lab 13 



## Privilege Escalation via Reference-Based Access Control



Severity: High



Vulnerability Type: Reference-Based Access Control Failure



Summary



The application performs authorization decisions based on user-supplied object references (e.g., username parameter) rather than validating the privileges of the authenticated user.



As a result, a normal user can manipulate references in privileged endpoints to escalate their own privileges.



Steps to Reproduce



Authenticate as a normal user (wiener).



Intercept an administrator role-management request:



POST /admin-roles

username=carlos\&action=upgrade





Modify the request:



username=wiener\&action=upgrade





Send the request using the normal user’s session.



Impact



The normal user is successfully promoted to administrator, gaining full administrative privileges.



Root Cause



Authorization logic relies on client-controlled references rather than enforcing role checks based on the authenticated session.



Recommendation



Enforce authorization using server-side identity and role verification.



Never trust object references provided by the client.



Bind authorization decisions strictly to authenticated user context.

