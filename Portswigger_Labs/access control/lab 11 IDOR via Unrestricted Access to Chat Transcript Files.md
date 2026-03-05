# Lab 11: IDOR via Unrestricted Access to Chat Transcript Files

## Severity
High

## Affected Component
Chat transcript download endpoint (`/download-transcript/`)

## Vulnerability Summary
The application stores user chat transcripts as static files on the server and allows them to be retrieved via predictable URLs. The endpoint does not enforce authorization checks to ensure that the requesting user owns the requested transcript.


By modifying the transcript filename in the request, an attacker can access other users’ private chat logs, resulting in unauthorized data disclosure.

## Business Impact
An authenticated attacker can:
* Access other users’ private chat conversations
* Retrieve sensitive information such as passwords
* Perform account takeover using leaked credentials
* Enumerate and scrape chat data at scale

This exposes highly sensitive user data and undermines user privacy.

## Steps to Reproduce
1. Authenticate as a normal user.
2. Use the live chat feature and click **View transcript**, observing a request similar to:
   `GET /download-transcript/5.txt HTTP/2`
3. Send the request to Burp Repeater.
4. Modify the filename parameter:
   `GET /download-transcript/1.txt HTTP/2`
5. Send the request.

## Proof of Concept
The response returns another user’s chat transcript containing `carlos’` password. Using the disclosed credentials, the attacker can authenticate as `carlos` and fully access his account.

## Root Cause Analysis
The backend retrieves transcript files directly from the filesystem based on a user-controlled identifier without verifying ownership or enforcing access control. 

Static file naming combined with missing authorization enables direct object reference attacks against stored user data.

## Recommendation / Mitigation
* Enforce strict authorization checks before serving user-specific files.
* Use non-guessable, access-controlled storage paths.
* Bind transcript access to the authenticated user’s session.
* Avoid storing sensitive data such as passwords in chat logs.
* Monitor and log abnormal access patterns to transcript files.
