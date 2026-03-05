lab 12



## Privilege Escalation via Missing Access Control in Multi-Step Workflow

Severity



High



Affected Component



Administrative role management workflow (/admin-roles)



Vulnerability Summary



The application implements a multi-step workflow for modifying user roles. While access control is enforced on the initial step, subsequent confirmation steps do not properly verify the requester’s privileges.



By directly invoking the confirmation endpoint, a low-privileged authenticated user can bypass authorization checks and promote themselves to administrator.



Business Impact



An authenticated attacker can:



Escalate privileges to administrator



Gain full control over user management



Perform all administrative actions



This results in complete compromise of the application’s authorization model.



Steps to Reproduce



Authenticate as a normal user (e.g., wiener).



Intercept a role-change confirmation request:



POST /admin-roles

action=upgrade\&confirmed=true\&username=carlos





Modify the username parameter:



action=upgrade\&confirmed=true\&username=wiener





Send the request.



Proof of Concept



The request succeeds and upgrades wiener to administrator, despite the user lacking administrative privileges.



Root Cause Analysis



Authorization checks are applied only during the initial role-selection step of the workflow. The final confirmation endpoint assumes that prior steps were completed by an authorized user and fails to re-validate privileges.



This allows attackers to bypass access control by directly invoking later workflow stages.



Recommendation / Mitigation



Enforce authorization checks on every step of multi-step workflows.



Do not rely on previous steps for security decisions.



Validate user permissions at the final state-changing action.



Centralize access control logic to prevent partial enforcement.

