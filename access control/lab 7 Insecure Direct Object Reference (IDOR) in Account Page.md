lab 7



## Insecure Direct Object Reference (IDOR) in Account Page

Severity



High



Affected Component



User account management endpoint (/my-account)



Vulnerability Summary



The application exposes user-specific account data based solely on a client-supplied request parameter (id) without verifying that the authenticated user is authorized to access the referenced account.



By modifying the id parameter, an authenticated attacker can access other users’ account information, resulting in unauthorized data disclosure.



Business Impact



An authenticated attacker can:



Access other users’ private account data



Retrieve sensitive information such as API keys



Enumerate users and harvest data at scale



In real-world deployments, this could lead to privacy violations, account compromise, and regulatory exposure.



Steps to Reproduce



Authenticate as a normal user (e.g., wiener).



Navigate to the account page and intercept the request:



GET /my-account?id=wiener HTTP/2





Modify the id parameter to reference another user:



GET /my-account?id=carlos HTTP/2





Send the request.



Proof of Concept



The response contains carlos’ account data, including his API key, despite the attacker being authenticated as wiener.



Root Cause Analysis



The backend application uses a user-controlled request parameter to determine which account data to return, without validating that the requested user ID matches the authenticated session.



Authorization is performed implicitly through parameter values rather than enforced server-side using session context.



Recommendation / Mitigation



Derive the user identity exclusively from the authenticated session, not request parameters.



Implement object-level authorization checks for all user-specific resources.



Prevent user enumeration by avoiding predictable identifiers.



Log and monitor unauthorized access attempts.

