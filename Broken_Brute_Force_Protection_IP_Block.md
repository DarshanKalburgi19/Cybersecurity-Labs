# Broken Brute-Force Protection – IP Block

## Lab Objective
The website blocks an IP after three failed login attempts.

The goal of this lab was to exploit a logic flaw in the brute-force protection and obtain valid credentials for the target user.

---

## Vulnerability Explanation
The application tracks failed login attempts per IP address.

However, the **failed-attempt counter resets after any successful login**, even if the login belongs to a different user.

So the logic becomes:
- Fail login → counter increases
- Successful login (any user) → counter resets

This breaks the brute-force protection.

---

## Key Insight
Instead of triggering an IP block by repeatedly failing login attempts, we can:
- Insert a valid login attempt in between failed attempts
- Continuously reset the failed-attempt counter
- Avoid IP blocking entirely

---

## Exploitation Strategy

### Known Valid Credentials
The lab provides valid credentials:
- `wiener : peter`

These credentials are used only to **reset the failed-attempt counter**.

---

### Attack Logic
Instead of brute-forcing like this:
carlos + wrong
carlos + wrong
carlos + wrong → IP blocked

css
Copy code

We use this pattern:
carlos + wrong
wiener + peter → resets counter
carlos + wrong
wiener + peter → resets counter

yaml
Copy code

This allows unlimited brute-force attempts without triggering the IP block.

---

## Tool Usage (Intruder)

- Sent the login request to Intruder
- Selected **two payload positions**:
  - Username
  - Password
- Used **Pitchfork attack**
- Carefully arranged payloads so:
  - `wiener:peter` appears regularly between failed attempts
- Set **maximum concurrent requests = 1**
  - This is important because out-of-order requests can break the logic

---

## Identifying Correct Credentials
- Observed HTTP status codes
- A successful login returned **302**
- Used response differences and error messages to identify valid credentials

Eventually, the correct credentials for the target user were found and the lab was solved.

---

## My Initial Confusion
At first, I didn’t clearly understand:
- Why logging in as another user would affect brute-force protection
- How the failed-attempt counter behaved internally

Once I understood that **any successful login resets the counter**, the attack became clear.

---

## Key Takeaways
- Brute-force protection must be tied to the specific user, not just the IP
- Authentication logic must not reset security counters globally
- Understanding application logic is more important than tool usage
- Small logic flaws can completely bypass security controls
