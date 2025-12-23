# Username Enumeration via Response Timing – PortSwigger Lab

## Lab Objective
The goal of this lab was to exploit a response timing vulnerability to:
1. Identify a valid username
2. Brute-force the corresponding password
3. Log in to the target account

The application also had IP-based brute force protection, which needed to be bypassed.

---

## Vulnerability Explanation
The application responds differently based on whether the username exists.

- Invalid username + any password → very fast response
- Valid username + wrong password → noticeably slower response

This happens because the server performs additional password checks only when the username is valid.  
This difference in response time allows **username enumeration**.

---

## Brute Force Protection Flaw
The application blocks repeated login attempts based on the client IP.

However, it **trusts HTTP headers like `X-Forwarded-For`** to identify the client.  
By modifying this header, the server assumes each request is coming from a new user, effectively bypassing the IP-based protection.

---

## My Initial Mistake
At first, I:
- Knew I had to change the `X-Forwarded-For` header
- But didn’t understand *why* it worked
- Assumed changing the header randomly was enough

I also didn’t clearly understand how timing differences alone could confirm a valid username.

---

## Correct Exploitation Logic

### Step 1: Username Enumeration
- Sent the login request to Intruder
- Added `X-Forwarded-For: 1.2.3.X` to bypass IP blocking
- Used a number range payload for the header
- Used the provided username list as payload

One username showed a **consistently higher response time**, indicating it was valid.

---

### Step 2: Password Brute Force
- Cleared previous Intruder settings
- Fixed the valid username
- Used the password list as payload
- Continued modifying `X-Forwarded-For`

One password resulted in a **302 redirect**, confirming correct credentials.

---

## Result
Successfully logged in using the discovered credentials and completed the lab.

---

## Key Takeaways
- Response timing alone can leak sensitive information
- Authentication logic must behave consistently for all failure cases
- Trusting client-controlled headers is dangerous
- Understanding *why* an attack works is more important than just solving the lab
