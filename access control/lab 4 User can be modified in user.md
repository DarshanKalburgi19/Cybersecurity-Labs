### **Lab -4 User can be modified in user profile.**



**Vulnerability Type** 

**Broken Access Control**



severity: IDK (if any one can help please tell me how to give a bug a severity rate)


**User can be modified in user profile**
===

###### 

###### **Description:**

&nbsp; The application restricts access to the admin panel at /admin to users with roleid:2. Although normal users should not be able to modify their role, the application exposes role information to the normal user through an API response and fails to enforce protection and proper authorization checks when processing profile update requests.



###### **steps to reproduce:**

**step 1:**

login with the given credentials

username: wiener

password: peter



**step 2:**

Attempt to access the admin panel directly 

GET /admin



Access is denied means , let's check if role based restrictions are in place.



**step 3:**

Attempt parameter manipulation using common role values(eg. admin,administrator) no success.



**step 4:**

Update the email address via the user profile functionality (eg teresa@ca )and intercept the response/request.



**step 5:**

Observe that the response contains sensitive authorization data:

{

&nbsp; "user": "wiener",

&nbsp; "apikey": "…",

&nbsp; "roleid": 1

}



**step 6:**

send the email update request to the repeater.



**step 7:**

Modify the json code adding role based parameter 

{

&nbsp;"user": "wiener",

&nbsp;"roleid": 2

}



**step 8:**

Forward the modified request to the server.



**step 9:**

Access the admin panel successfully.

GET /admin



**step 10:** 

Delete the user carlos to complete the lab.



###### **Technical Analysis :**

&nbsp; The application exposes internal authorization (roleid) in API response code and fails to restrict which fields can be modified by the user during profile updates.



Authorization decisions must never be based on the user-modifiable data.





###### **Impact:**

**An attacker can :**

&nbsp;	1.Can escalate privilages from a normal user to administrator.

&nbsp;	2.Access restricted to administrative endpoints.

&nbsp;	3.Perform high-impact actions such as deleting user accounts.


**This could lead to:**

&nbsp;	1.Full account takeover.

&nbsp;	2.Data manipulation or deletion.

&nbsp;	3.Complete application compromise.



###### **Key take-way:**

&nbsp; Exposing internal authorization attributes and accepting them from user input leads to critical privilege escalation vulnerabilities.	





