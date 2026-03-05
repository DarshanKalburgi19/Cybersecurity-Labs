LAB 6 



## Privilege Escalation via Method-Based Access Control Bypass

Severity



High



Affected Component



User role management endpoint (/admin-roles)



Vulnerability Summary



The application enforces access control for sensitive role-modification functionality based on the HTTP method rather than the requested action itself. While administrative role changes are restricted for POST requests, the same functionality remains accessible via GET requests without proper authorization checks.



As a result, a low-privileged authenticated user can escalate their privileges to administrator by invoking the role-modification endpoint using an unauthorized HTTP method.



Business Impact



An attacker with a standard user account can:



Escalate their privileges to administrator



Gain full control over user management



Perform administrative actions affecting application integrity and data security



This vulnerability enables complete account privilege takeover.



Steps to Reproduce



Authenticate as an administrator and observe the request used to upgrade a user’s role.



Log out and authenticate as a low-privileged user.



Send the following request manually:



GET /admin-roles?username=wiener\&action=upgrade HTTP/2





Observe that the request succeeds and the user wiener is upgraded to administrator, despite lacking the required privileges.



Proof of Concept



The above request successfully escalates the attacker-controlled account from a normal user to administrator without authorization.



Root Cause Analysis



The backend application applies authorization checks conditionally based on the HTTP method (POST) rather than enforcing access control at the action or resource level. The role modification logic is still executed when accessed via a GET request, allowing unauthorized users to trigger privileged functionality.



This reflects a failure to implement consistent, method-independent authorization controls.



Recommendation / Mitigation



Enforce authorization checks at the business logic level, not based on HTTP methods.



Restrict role-modification endpoints to authorized roles regardless of request type.



Avoid exposing sensitive state-changing functionality via GET requests.



Implement centralized authorization middleware for all privileged operations.

