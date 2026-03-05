### Lab : User role controlled by request parameter 

A portswigger lab.

# User role controlled with request parameter 


###### **vulnerability type:**

	**Broken access control/Client side authorization bypass.**


&nbsp;	severity: IDK (if any one can help please tell me how to get severity rate)



###### **Summary:**

&nbsp;	The LAB application exposes at admin panel at /admin. Although normal users should not have access to the panel, authorization is controlled using a parameter that can be changed/modified by the client. 



###### **Steps to reproduce:**



**step 1:**

Login to the application using the provided credentials:

username : wiener

password : peter



**step2:**

browse the site normally and catch the request in burpsuite 



**step3:**

Identify the role based parameter which is request based. 

Admin=False



**step4:**

Send the request to Burp repeater.



**step5:**

Modify the parameter value.

Admin=True



**step6:**

Modify the request to the server.



**step7:**

Observe that the application grants access to the /admin panel.



**step8:**

From the admin interface delete the user carlos to successfully complete the lab.





###### **Technical Analysis:**

&nbsp;	The application relies on a client supplied request parameter  to determine wheater a user has administrative previlages.Since this parameter can be freely modified by the user, an attacker can escalate privilages by changing its value.



Authorization decisions should be enforced at server-side not the user side.





###### **Impact:**

**An acctacker can:**

&nbsp;	Escalate privilages from normal users to administrative level.

Access restricted admin functionality 

Can perform destructive actions such as deleting user accounts



**In real-world scenario this could lead to:**

Account takeover

Data loss

Full system compromise 



###### **Key take-way :**

&nbsp;	Authorization must be based on trusted server-side logic, not on user-modifiable request parameters.