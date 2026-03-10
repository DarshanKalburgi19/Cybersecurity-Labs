# Broken Brute-Force Protection via Multiple Credentials in Single JSON Request

> Platform: PortSwigger Web Security Academy  
> Lab: Broken brute-force protection, multiple credentials per request  
> Difficulty: Expert  

---

# Summary

The login endpoint applies brute-force protection **per HTTP request**, not per credential. By manually crafting a malformed JSON request containing multiple password fields, authentication succeeds if the correct password is included anywhere in the request. This allows an attacker to bypass rate limiting and account lockout protections.

The vulnerability occurs because the backend incorrectly handles duplicate JSON keys and fails to validate the credential structure before performing authentication.

---

# Initial Observations

The login form submits credentials as **JSON via JavaScript**.

Attempting a standard brute-force attack using **Burp Suite Intruder (Sniper)** quickly resulted in **rate limiting** after multiple login attempts.

This suggested that brute-force protection was applied **per request**, rather than per credential attempt.

---

# Frontend Code Analysis

## Login Form (HTML)

```html
<form class="login-form" method="POST" action="/login">

    <label>Username</label>
    <input required type="username" name="username">

    <label>Password</label>
    <input required type="password" name="password">

    <button onclick="event.preventDefault(); jsonSubmit('/login')">Log in</button>

</form>
```

## JavaScript Function

```javascript
function jsonSubmit(loginPath) {

    const formToJSON = elements => [].reduce.call(elements, (data, element) => {

        if (element.name && element.name.length > 0) {
            data[element.name] = element.value;
        }

        return data;

    }, {});

    const jsonObject = formToJSON(document.getElementsByClassName("login-form")[0].elements)

    const formData = JSON.stringify(jsonObject);

    fetch(loginPath, {

        method: "POST",
        body: formData,

        headers: {
            "Content-Type": "application/json"
        }

    })

}
```

### Key Takeaway

The frontend always sends **exactly one password field**. Duplicate credentials **cannot originate from the client**, meaning any handling of multiple passwords is purely a **backend issue**.

---

# Attack Methodology

## Step 1: Standard Brute Force (Failed)

Initial attempts used **Burp Suite Intruder Sniper attack** to brute force passwords.

Result:

- Rate limited after multiple requests
- Temporary login block occurred

Conclusion:

Brute-force protection was functioning **per request**, not per credential attempt.

---

## Step 2: Manual JSON Manipulation

The login request was intercepted using **Burp Suite**, and the JSON body was manually modified to include **multiple password fields**.

### Example Payload

```http
POST /login HTTP/2
Content-Type: application/json

{
  "username": "carlos",
  "password": "admin",
  "password": "123456",
  "password": "password",
  "password": "qwerty"
}
```

This payload contains multiple password guesses within **a single request**.

---

# Testing and Observations

## Initial Hypothesis (Incorrect)

At first, it was assumed that:

> The backend tests passwords sequentially until the correct one is found.

However, testing disproved this assumption because:

- No increase in response time
- No evidence of looping
- No observable per-password evaluation

This hypothesis was discarded after deeper analysis.

---

# Testing Attempts

| Attempt | Password Count | Result |
|-------|-------|-------|
| Test 1 | 10 passwords | ❌ Login failed |
| Test 2 | 20 passwords | ❌ Login failed |
| Test 3 | 50 passwords | ❌ Login failed |
| Test 4 | Full password list | ✅ Login succeeded |

Additional observations:

- The **last password was not the correct one**
- The **correct password appeared somewhere in the middle**
- Password order did **not consistently affect the outcome**

This ruled out simple **“last key wins” behavior**.

---

# Final Understanding

## Backend Behavior

The backend:

1. Accepts malformed JSON containing **duplicate password keys**
2. Parses duplicate values into a **collection-like structure**
3. Performs authentication incorrectly
4. Validates credentials if **any password in the structure is correct**

This allows **multiple credential guesses inside a single request**.

---

# Why This Bypasses Brute-Force Protection

The brute-force defense counts **HTTP requests**, not **credential attempts**.

Therefore:

```
1 request = unlimited password guesses
```

This bypasses:

- Rate limiting
- Account lockout mechanisms
- Request-based brute-force protections

---

# Proof of Exploit

Example payload containing multiple password attempts:

```json
{
  "username": "carlos",
  "password": "wrong1",
  "password": "correct_password",
  "password": "wrong2"
}
```

Result:

```
HTTP/1.1 302 Found
Location: /my-account
```

Successful authentication occurs because the correct password exists **somewhere within the request payload**.

---

# Impact

This vulnerability allows attackers to bypass brute-force protections and attempt multiple credentials within a single request.

Potential impacts include:

- Authentication bypass
- Account takeover
- Undetected password brute forcing
- Circumvention of rate limiting
- Weakening of login security controls

In real-world systems, this vulnerability could allow attackers to attempt **large password dictionaries within a single request**.

---

# Root Cause

The vulnerability occurs due to multiple backend security flaws:

- Failure to validate JSON structure
- Acceptance of duplicate credential keys
- Type confusion between expected **string values** and **collection-like structures**
- Brute-force protection applied **per request instead of per credential attempt**

---

# Key Lessons Learned

- Backend behavior should not be assumed solely from results.
- Black-box testing requires eliminating alternative explanations.
- Frontend source code can reveal important design assumptions.
- Subtle parsing issues can break authentication logic completely.
- Security mechanisms must validate **data structure and input format**, not just values.

---

# Correct Vulnerability Statement

The login endpoint mishandles multiple password values supplied in a single JSON request. Authentication succeeds if the correct password exists anywhere in the payload. Because brute-force protection is applied per request rather than per credential attempt, attackers can bypass rate limiting and account lockout mechanisms.

---

# Conclusion

This lab demonstrates how subtle input parsing issues can compromise authentication security. The vulnerability is not a traditional brute-force flaw but a logic issue caused by malformed JSON input and improper validation of credential structures.

Proper input validation and strict request schema enforcement are essential to prevent such authentication bypass vulnerabilities.