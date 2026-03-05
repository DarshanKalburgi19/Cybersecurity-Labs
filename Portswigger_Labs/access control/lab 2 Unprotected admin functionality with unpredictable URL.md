###### **LAB : Unprotected admin functionality with unpredictable URL.**

A Portswigger LAB.



# **Unprotected Admin Functionality via client-Side Exposure** 



###### **Executive Summary**

&nbsp;	An unprotected administrative interface was discovered by inspecting client-side JavaScript files. The application exposed a sensitive admin endpoint that lacked proper access control, allowing an unauthenticated user to perform administrative actions, including deleting users accounts.





###### **Vulnerability type :**

Broken Access Control 

Unprotected administrative functionality 





###### **Technical description:**

&nbsp;	The application exposed a administrative endpoint within client-side JavaScript code. Although the URL was not directly linked in user interface but, it was accessible to any user who discovered it. The server failed to enforce authorization checks on this endpoint, allowing unauthorized users to access administrative functionalities.



###### **Affected Functionality :**

	Endpoint : /admin-mbklof

&nbsp;	Authentication: Not required 
	Authorization: Missing



###### **Steps to reproduce :**

1 .Visit the target web application.

2 . Inspect the page source and review the JavaScript files.

3 .Identify a refrence to an administrative endpoint in client-side code.

4 .Navigate directly to the discovered admin URL.

5 .Observe that administrative actions are directly available/accessible with out any authentication.

6 .Using the availability functionality to delete a user account.



**Impact** :
---

&nbsp;	An attacker could gain full administrative access to the application without any authentication. This could result in unauthorized data control, modification, user accounts takeover, deletion or complete compromise of the application.





###### **Root Cause :**

&nbsp; The application relies on obscurity to protect administrative functionality and does not implement server-side authentication authorised checks for sensitive endpoints.