lab 8 



## IDOR via Indirect Disclosure of Unpredictable User IDs

Severity



High



Affected Component



User account page (/my-account)



Vulnerability Summary



The application relies on unpredictable user identifiers (GUIDs) to protect access to user-specific account pages. However, these identifiers are indirectly exposed through publicly accessible content such as blog posts.



By extracting another user’s GUID and supplying it as a request parameter, an authenticated attacker can access unauthorized account data, resulting in horizontal privilege escalation.



Business Impact



An authenticated attacker can:



Access other users’ private account data



Retrieve sensitive information such as API keys



Enumerate users via publicly exposed relationships (e.g., blog authors)



The use of unpredictable identifiers does not prevent exploitation once the identifier is disclosed.



Steps to Reproduce



Authenticate as a normal user (e.g., wiener).



Browse publicly available blog posts.



Identify a post authored by another user (e.g., carlos).



Click the author’s username and observe the leaked user identifier (GUID).



Intercept a request to the account page:



GET /my-account?id=<wiener\_guid> HTTP/2





Replace the id parameter with the extracted GUID:



GET /my-account?id=759e1617-3341-49dd-9e20-f13988bd3b32 HTTP/2





Send the request.



Proof of Concept



The response returns carlos’ account information, including his API key, despite the attacker being authenticated as wiener.



Root Cause Analysis



The backend application authorizes access to account data solely based on a user-controlled identifier supplied in the request. Although the identifier is a GUID, it is still exposed indirectly through other application features.



Authorization is not enforced by validating ownership against the authenticated session.



Recommendation / Mitigation



Enforce object-level authorization using session context rather than request parameters.



Avoid exposing internal user identifiers in public content.



Treat GUIDs as identifiers, not security controls.



Implement consistent authorization checks for all user-specific resources.

