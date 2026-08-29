# Chapter 11 — Password Security & Best Practices

In Chapter 10, we built the basic idea of authentication.

A user registers:

```text
email + password
       ↓
password hashing
       ↓
database
```

Then later:

```text
email + password
       ↓
find user
       ↓
verify password
       ↓
create session/token
```

It looks pretty straightforward.

But here's the uncomfortable question:

> What happens if someone gets access to your database?

Suppose your database contains:

```text
users

id | email              | password
---|--------------------|----------------
1  | rahul@example.com  | mypassword123
2  | aman@example.com   | qwerty123
```

That's basically game over.

The attacker doesn't need to break your application anymore.

They already have everyone's passwords.

And there's an even bigger problem.

People reuse passwords.

So one leaked database might give an attacker access to:

```text
Email
↓
Banking account
↓
GitHub
↓
Social media
↓
Other applications
```

This is why password security deserves its own chapter.

---

# 1. First Rule: Never Store Passwords

Let's start with the simplest rule:

> **Never store users' plaintext passwords.**

Not:

```js
{
  email: "rahul@example.com",
  password: "rahul123"
}
```

Not:

```text
password = "rahul123"
```

Not even:

```text
password = encrypted("rahul123")
```

for the purpose of normal password storage.

Instead:

```text
password
    ↓
password hashing algorithm
    ↓
password hash
    ↓
database
```

The database should contain a password verifier, not the original password.

---

# 2. But Why Hash Passwords?

Let's say Rahul chooses:

```text
my-super-secret-password
```

We pass it through a password hashing algorithm.

We get something like:

```text
$2b$12$...
```

The exact value doesn't matter here.

The important thing is:

```text
password
   ↓
hash
```

And ideally:

```text
hash
   ✕
password
```

You don't reverse the hash to get the original password.

When Rahul logs in, you don't decrypt anything.

You do:

```text
Login password
     ↓
password verification
     ↓
stored password hash
     ↓
match?
```

If it matches:

```text
Login successful
```

---

# 3. Hashing Is Not Encryption

This is one of the most common interview questions.

### Encryption

Encryption is designed to be reversible when you have the appropriate key.

```text
plaintext
   ↓
encryption
   ↓
ciphertext
   ↓
decryption
   ↓
plaintext
```

### Password hashing

Password hashing is designed to be one-way.

```text
password
   ↓
hashing
   ↓
stored hash
```

You don't need to recover the original password.

You only need to verify whether the user knows the correct password.

So:

```text
Encryption
→ protect data that you may need to recover

Password hashing
→ store passwords safely for verification
```

---

# 4. Why Not Use SHA-256?

Someone might ask:

> "Can't I just do SHA-256(password)?"

Technically, you can calculate a SHA-256 hash.

But that's not what password hashing is designed for.

SHA-256 is a general-purpose cryptographic hash function.

It's intentionally very fast.

That sounds like a good thing.

For passwords, it isn't.

Imagine an attacker has stolen your database.

They can try:

```text
password1
password2
password3
password4
...
```

If your password hashing process is extremely fast, they can test enormous numbers of guesses very quickly.

For passwords, we actually **want hashing to be deliberately expensive**.

---

# 5. Password Hashing Should Be Slow

This sounds strange.

For most backend operations:

```text
faster = better
```

For password hashing:

```text
too fast = bad
```

We want a password hashing algorithm that makes each password guess relatively expensive.

For example:

```text
Attacker
  ↓
Guess password
  ↓
Expensive password hash
  ↓
Check result
```

If one guess takes some meaningful amount of computation, billions of guesses become much more expensive.

That's the basic idea.

---

# 6. bcrypt

One common password hashing algorithm in Node.js applications is:

> **bcrypt**

For example:

```bash
npm install bcrypt
```

Then:

```ts
import bcrypt from "bcrypt";
```

To hash a password:

```ts
const passwordHash = await bcrypt.hash(password, 12);
```

You store:

```text
passwordHash
```

not:

```text
password
```

---

# 7. What Is the `12`?

You might see:

```ts
bcrypt.hash(password, 12);
```

That `12` is the bcrypt cost factor.

You don't need to memorize the exact performance characteristics.

The important idea is:

```text
Higher cost
    ↓
More computation
    ↓
More expensive password guesses
```

But you can't just keep increasing it forever.

If password hashing becomes too expensive, your own login and registration endpoints become unnecessarily slow and consume more server resources.

So the right cost depends on your environment and should be benchmarked.

---

# 8. Don't Copy Someone's Cost Number Forever

You may find a tutorial saying:

```ts
bcrypt.hash(password, 10);
```

Another says:

```ts
bcrypt.hash(password, 12);
```

Another:

```ts
bcrypt.hash(password, 14);
```

Don't treat one number as a universal magic value.

The right configuration depends on:

```text
Hardware
Server workload
Expected traffic
Latency requirements
Security requirements
```

The general principle is:

> **Use a password hashing configuration that is deliberately expensive but still practical for your production environment.**

And review it as infrastructure changes.

---

# 9. What Is a Salt?

Let's say two users choose:

```text
password123
```

If you simply apply the same deterministic process:

```text
password123
   ↓
hash
   ↓
same result
```

Both users would have the same stored hash.

That's not ideal.

A **salt** is a unique random value incorporated into password hashing.

Conceptually:

```text
Password A + random salt A
          ↓
        hash A

Password B + random salt B
          ↓
        hash B
```

Even if:

```text
Password A = Password B
```

their stored hashes can still be different.

---

# 10. Do I Need to Generate the Salt Myself?

With bcrypt:

```ts
const hash = await bcrypt.hash(password, 12);
```

bcrypt handles the salt generation and incorporates the necessary information into the resulting hash.

You generally don't need to manually invent your own salt system.

That's another reason to use a well-established password hashing library instead of writing your own cryptography.

---

# 11. Why Does the Hash Contain So Much Information?

A bcrypt hash can look something like:

```text
$2b$12$.......................................................
```

It contains information needed by bcrypt to verify the password, including things such as:

```text
algorithm/version information
cost factor
salt
derived hash
```

So when you call:

```ts
bcrypt.compare(password, storedHash);
```

bcrypt knows how to perform the appropriate verification.

That's why you don't need a separate database column like:

```text
salt
```

for a normal bcrypt setup.

The salt information is encoded as part of the bcrypt string.

---

# 12. Password Verification

When Rahul logs in:

```ts
const valid = await bcrypt.compare(
  password,
  user.passwordHash
);
```

If the password is correct:

```text
true
```

If not:

```text
false
```

Notice that we don't do:

```ts
bcrypt.hash(password) === user.passwordHash
```

because password hashing uses a unique salt.

Use the password-hashing library's verification function.

---

# 13. What If the Database Gets Leaked?

Let's say the attacker gets:

```text
email
passwordHash
```

instead of:

```text
email
password
```

That's much better.

But don't misunderstand this.

A password hash isn't magically useless to an attacker.

An attacker can still attempt:

```text
Guess password
    ↓
Hash guess
    ↓
Compare with stolen hash
```

They can repeat that process.

That's called **password cracking** or offline password guessing.

Your goal is to make those guesses expensive.

That's why we care about:

```text
Strong password hashing
+
Salt
+
Appropriate cost
```

---

# 14. Brute-Force Attacks

Now imagine someone repeatedly sends:

```http
POST /auth/login
```

with:

```text
password1
password2
password3
password4
...
```

They're trying to guess the password.

That's a brute-force attack.

Without protection:

```text
Attacker
   ↓
100 requests
   ↓
1,000 requests
   ↓
100,000 requests
   ↓
...
```

Your login endpoint can become an attack target.

---

# 15. Rate Limiting

One basic defense is:

> **Rate limiting**

Instead of allowing:

```text
10,000 login attempts
```

from the same source in a short period, you can limit requests.

For example:

```text
5 failed attempts
       ↓
slow down / temporary block
```

The exact limits depend on your application.

You can rate-limit based on things such as:

```text
IP address
Account identifier
Device/session signals
Other risk signals
```

But be careful with simplistic IP-only limits because many legitimate users can share an IP address.

---

# 16. Rate Limiting Isn't Only for Login

You can rate-limit many sensitive endpoints:

```text
/auth/login
/auth/register
/auth/forgot-password
/auth/reset-password
```

and sometimes:

```text
/api/search
/api/comments
/api/messages
```

depending on the application's needs.

Rate limiting is a general backend protection mechanism.

We'll discuss it again in the production/system-design part of this series.

---

# 17. Credential Stuffing

Here's another attack that is extremely important.

Suppose an attacker already has:

```text
1 million leaked email/password combinations
```

from another website.

They try those credentials against your application.

For example:

```text
rahul@example.com
password123
```

Maybe Rahul used the same password on another website.

The attacker tries it on your website.

That's **credential stuffing**.

It's different from blindly guessing passwords.

The attacker is using credentials that were already leaked elsewhere.

---

# 18. How Do You Defend Against Credential Stuffing?

There isn't one magic solution.

Useful defenses include:

```text
Strong password hashing
Rate limiting
Monitoring suspicious login activity
MFA / passkeys
Breached-password checks
Account protection mechanisms
```

And most importantly:

> Encourage users not to reuse passwords.

If you're building a serious application, authentication is not just about hashing.

It's an entire security system.

---

# 19. Password Strength

You might think:

> "I'll force everyone to have a 30-character password with 12 symbols."

That's not necessarily the best user experience.

Modern password guidance generally favors strong, unique passwords/passphrases and defenses against common or compromised passwords rather than blindly forcing complicated character combinations.

For example:

```text
correct-horse-battery-...
```

can be easier for a human to remember than:

```text
X7!k@91#Qz...
```

The bigger issue is:

```text
Is the password unique?
Is it commonly used?
Has it appeared in a breach?
```

---

# 20. Account Enumeration

Here's a subtle problem.

Imagine your login endpoint responds:

```text
Email doesn't exist
```

when the account isn't found.

But:

```text
Wrong password
```

when the account exists.

An attacker can use this to discover which email addresses have accounts.

For example:

```text
attacker@example.com
→ Account doesn't exist

rahul@example.com
→ Wrong password
```

Now the attacker knows:

> Rahul has an account.

This is called **account enumeration**.

---

# 21. Avoid Giving Attackers Useful Differences

For sensitive authentication operations, applications often use generic responses.

For example:

```text
Invalid email or password.
```

instead of:

```text
Email doesn't exist.
```

or:

```text
Password is wrong.
```

The idea is:

> Don't unnecessarily tell an attacker which part of their guess was correct.

This principle also matters for password-reset flows.

---

# 22. Forgot Password

A common feature:

```http
POST /auth/forgot-password
```

The user enters:

```json
{
  "email": "rahul@example.com"
}
```

The backend should generate a secure, temporary reset mechanism.

Conceptually:

```text
User requests reset
       ↓
Generate random reset token
       ↓
Store verifier / reset state
       ↓
Send reset link
       ↓
User clicks link
       ↓
Verify token
       ↓
Allow new password
       ↓
Invalidate reset token
```

The reset token should be:

```text
Random
Hard to guess
Short-lived
Single-use
```

---

# 23. Don't Put Password Reset Tokens in Plaintext if You Don't Need To

A useful pattern is to store a secure hash/verifier of the reset token rather than the raw token itself.

Think:

```text
Token sent to user
       ↓
hash/token verifier
       ↓
database
```

If the database is compromised, an attacker shouldn't automatically receive usable reset links.

This is similar in spirit to password storage:

> Store what you need to verify, not more sensitive information than necessary.

---

# 24. Password Reset Tokens Should Expire

Imagine generating:

```text
reset-token
```

and leaving it valid forever.

That's dangerous.

If someone gets the token six months later, they might still reset the account password.

Instead:

```text
Token created
     ↓
short expiration
     ↓
used or expired
     ↓
invalid
```

---

# 25. Password Reset Tokens Should Be Single-Use

Suppose someone uses:

```text
reset-token-123
```

successfully.

It shouldn't remain valid.

After successful use:

```text
reset-token-123
        ↓
invalid
```

This prevents replay.

---

# 26. Sessions and Token Theft

Password security doesn't stop after login.

Suppose the user's password is perfectly protected.

But an attacker steals the user's active authentication credential.

For example:

```text
session cookie
```

or:

```text
access token
```

The attacker might be able to act as the user.

This is why we also care about:

```text
HTTPS
Secure cookies
HttpOnly cookies
SameSite settings
Token expiration
Session invalidation
XSS protection
CSRF protection
```

Authentication security is bigger than password hashing.

---

# 27. HTTPS Is Not Optional

Never send passwords over plain HTTP.

You want:

```text
HTTPS
```

because it protects communication between the client and server against network attackers who might otherwise observe or modify traffic.

So:

```text
Browser
   ↓
HTTPS
   ↓
Backend
```

not:

```text
Browser
   ↓
HTTP
   ↓
Backend
```

especially for authentication.

---

# 28. Secure Cookies

If you're using cookies for authentication, understand these attributes:

```text
HttpOnly
Secure
SameSite
```

### HttpOnly

JavaScript can't directly access the cookie.

This can reduce some risks if malicious JavaScript is running in the page.

### Secure

Cookie should only be sent over HTTPS.

### SameSite

Controls cross-site cookie sending behavior and can help defend against certain CSRF attacks.

These aren't magic switches.

You still need a complete security design.

---

# 29. XSS

XSS stands for:

> **Cross-Site Scripting**

Imagine your application accidentally allows an attacker to inject malicious JavaScript into a page.

That JavaScript may attempt to perform actions as the user or access data available to JavaScript.

This is one reason authentication credentials that are directly accessible to JavaScript require careful consideration.

Using `HttpOnly` cookies can reduce the ability of injected JavaScript to directly read the authentication cookie.

But:

> HttpOnly does not make XSS harmless.

Malicious JavaScript can still potentially make requests from the user's browser.

You still need to prevent XSS itself.

---

# 30. CSRF

CSRF stands for:

> **Cross-Site Request Forgery**

It matters particularly when browsers automatically attach authentication cookies to requests.

Imagine:

```text
User logged into bank.com
```

Then they visit a malicious website.

That website attempts to cause the browser to send a request to:

```text
bank.com/transfer
```

If the browser automatically includes the authentication cookie and the application doesn't have appropriate CSRF defenses, the request could potentially be accepted.

This is one reason cookie-based authentication needs careful CSRF protection.

---

# 31. SameSite and CSRF

Cookie configuration can help.

For example:

```text
SameSite=Lax
```

or:

```text
SameSite=Strict
```

can restrict certain cross-site cookie behavior.

But don't treat one setting as a complete CSRF solution.

Depending on the application, you may also use:

```text
CSRF tokens
Origin checks
SameSite cookies
Other request validation
```

The correct approach depends on the application architecture.

---

# 32. Pepper

We've talked about salt.

Now another term you may hear:

> **Pepper**

A pepper is an additional secret value used during password hashing/verification.

The difference:

```text
Salt
→ unique per password
→ generally stored with the password hash

Pepper
→ secret shared value
→ stored separately from the database
```

Conceptually:

```text
password
   +
salt
   +
pepper
   ↓
password hashing
   ↓
stored verifier
```

A pepper can add another layer of protection in certain architectures.

But it also introduces key-management complexity.

You need to protect it like a secret.

Don't hardcode it into your Git repository.

---

# 33. Where Should Secrets Live?

Not:

```js
const PEPPER = "my-super-secret-pepper";
```

in your source code.

Instead use secure configuration/secrets management appropriate to your environment.

For development:

```env
PASSWORD_PEPPER=...
```

For production:

```text
Secret manager
```

We'll discuss environment configuration and secrets properly in Chapter 19.

---

# 34. Password Hashing Algorithms

You'll encounter several names.

### bcrypt

Widely used and well supported.

### Argon2

A modern password hashing algorithm designed specifically for password hashing and commonly recommended for new systems when available and properly configured.

### scrypt

Another password-based key derivation function designed to make certain attacks expensive, including through memory usage.

You may also encounter:

```text
PBKDF2
```

which is widely standardized and used in many systems.

The important thing isn't:

> "Which algorithm is trendy?"

The important thing is:

> **Use a well-established password hashing/KDF algorithm with a secure configuration.**

And don't invent your own.

---

# 35. Don't Use MD5 or Plain SHA-256 for Password Storage

You may see code like:

```js
crypto.createHash("md5")
```

or:

```js
crypto.createHash("sha256")
```

for passwords.

Don't do that for a modern password-storage design.

General-purpose hashes are designed to be fast.

Password hashing needs different properties.

Use an appropriate password hashing/KDF algorithm.

---

# 36. Don't Build Your Own Password Hashing Algorithm

This is one of the easiest security rules to remember:

> **Don't invent cryptography.**

Don't create:

```text
password
+
username
+
random string
+
SHA256
+
another SHA256
+
MD5
```

and call it secure.

Security algorithms have been studied for years.

Use established libraries and established algorithms.

---

# 37. Login Rate Limiting Needs Thought

Let's say you implement:

```text
5 login attempts per IP per hour
```

Sounds good.

But what if:

```text
10,000 users
```

are behind the same corporate NAT?

One IP might represent many legitimate users.

Likewise, an attacker can rotate IP addresses.

So rate limiting should be designed around the actual threat model.

You might combine:

```text
IP-based limits
+
account-based limits
+
progressive delays
+
risk signals
+
monitoring
```

Again, there isn't one magic number.

---

# 38. Progressive Delays

Instead of immediately blocking someone forever:

```text
failed attempt
→ 1 second

failed attempt
→ 2 seconds

failed attempt
→ 5 seconds

failed attempt
→ 30 seconds
```

you can increase the cost of repeated failures.

This can slow attackers while still allowing legitimate users to recover from a typo.

The exact strategy depends on your application.

---

# 39. Don't Lock Accounts Too Easily

You might think:

> "After 5 failed attempts, permanently lock the account."

That creates another problem.

An attacker can intentionally trigger lockouts for other users.

That's a denial-of-service technique against accounts.

Security controls should consider how they can be abused themselves.

---

# 40. Multi-Factor Authentication

Passwords have weaknesses.

Users reuse them.

They can be phished.

They can be leaked.

One way to improve account security is:

> **MFA — Multi-Factor Authentication**

Instead of:

```text
Password
```

you might require:

```text
Password
+
Authenticator code
```

or:

```text
Password
+
Security key
```

or use:

```text
Passkey
```

MFA is especially useful for sensitive applications and privileged accounts.

---

# 41. Passkeys

You may have noticed modern websites offering:

```text
Sign in with a passkey
```

Passkeys use public-key cryptography and are designed to reduce reliance on passwords and resist phishing better than traditional passwords.

You don't need to implement passkeys in this repository yet.

But it's worth knowing the term because authentication is moving beyond passwords.

---

# 42. Authentication Is a System

This is the biggest lesson from this chapter.

A secure authentication system isn't:

```text
bcrypt
+
JWT
=
secure
```

It's more like:

```text
Password hashing
        +
Secure session/token handling
        +
HTTPS
        +
Rate limiting
        +
Input validation
        +
Account protection
        +
Password reset security
        +
CSRF/XSS defenses
        +
Monitoring
        +
Good secret management
```

Security comes from the whole system.

---

# 43. A Safer Registration Flow

Let's put everything together.

```text
POST /auth/register
        ↓
Validate input
        ↓
Normalize appropriate fields
        ↓
Check account rules
        ↓
Hash password
        ↓
Store password hash
        ↓
Create user
        ↓
Return safe response
```

Notice:

```text
password
```

doesn't get stored.

---

# 44. A Safer Login Flow

```text
POST /auth/login
        ↓
Validate input
        ↓
Rate-limit / abuse checks
        ↓
Find account
        ↓
Verify password
        ↓
Create authentication state
        ↓
Set secure cookie / return appropriate token
        ↓
Return response
```

And avoid unnecessarily revealing:

```text
"email doesn't exist"
```

versus:

```text
"password is wrong"
```

---

# 45. A Safer Password Reset Flow

```text
POST /auth/forgot-password
        ↓
Generic response
        ↓
Generate secure random token
        ↓
Store verifier + expiration
        ↓
Send reset link
        ↓
User opens link
        ↓
Verify token
        ↓
Set new password
        ↓
Invalidate reset token
        ↓
Invalidate relevant old sessions
```

That last step is easy to forget.

If the account was compromised, changing the password should be part of recovering control of the account.

---

# 46. Common Mistakes

## Mistake 1 — Plaintext passwords

Never.

---

## Mistake 2 — SHA-256 password storage

Don't use a fast general-purpose hash as your password-storage scheme.

---

## Mistake 3 — Rolling your own crypto

Use established algorithms and libraries.

---

## Mistake 4 — No rate limiting

A login endpoint without abuse protection is an easy target for automated attacks.

---

## Mistake 5 — Revealing whether an account exists

Be careful with authentication and password-reset responses.

---

## Mistake 6 — Password reset tokens that never expire

Reset tokens should be temporary and single-use.

---

## Mistake 7 — Forgetting session invalidation

Changing a password doesn't necessarily invalidate already-issued sessions/tokens unless your system is designed to do that.

---

## Mistake 8 — Putting secrets in Git

Don't commit:

```text
JWT_SECRET
database password
pepper
API keys
```

to your repository.

---

## Mistake 9 — Treating JWT as security

JWT is just one mechanism.

A badly designed JWT-based authentication system can still be insecure.

---

## Mistake 10 — Trusting the frontend

Authorization must be enforced by the backend.

---

# 47. Mini Project — Harden Our Forum Authentication

We already built authentication for the forum.

Now improve it.

Your API:

```text
POST /auth/register
POST /auth/login
POST /auth/logout
POST /auth/forgot-password
POST /auth/reset-password
GET  /profile
```

Add:

```text
✓ Password hashing
✓ Password verification
✓ Rate limiting
✓ Generic login errors
✓ Secure reset tokens
✓ Reset token expiration
✓ Single-use reset tokens
✓ Secure cookies if using cookie auth
✓ HTTPS in deployment
✓ Proper authentication middleware
✓ Authorization checks
```

Then test attacks.

---

# 48. Try to Break Your Own Login

This is where learning becomes interesting.

Don't just test:

```text
correct email
correct password
```

Try:

### Test 1

Wrong password 10 times.

What happens?

---

### Test 2

Try a nonexistent email.

What response do you get?

Does it reveal whether the account exists?

---

### Test 3

Try the same reset token twice.

Does the second attempt fail?

---

### Test 4

Try an expired reset token.

Does it fail?

---

### Test 5

Try accessing:

```text
GET /profile
```

without authentication.

---

### Test 6

Authenticate as User A.

Try deleting User B's post.

Does the backend reject it?

---

### Test 7

Inspect your API responses.

Are you accidentally returning:

```text
passwordHash
```

or other sensitive fields?

---

# 49. Interview Questions

## 1. Why shouldn't passwords be encrypted instead of hashed?

Because the application doesn't need to recover the original password. Password hashing provides a one-way verification mechanism and is designed for secure password storage.

---

## 2. Why shouldn't we use SHA-256 directly for passwords?

SHA-256 is a fast general-purpose cryptographic hash. Password storage benefits from deliberately expensive password hashing/KDF algorithms that make large-scale guessing attacks more costly.

---

## 3. What is a salt?

A unique random value used as part of password hashing so identical passwords can produce different stored hashes and precomputed attacks are made harder.

---

## 4. What is a pepper?

A secret value used in addition to the password and salt during password verification. Unlike a salt, it must be protected separately from the database.

---

## 5. What is brute-force protection?

Mechanisms that make repeated authentication attempts more difficult or expensive, such as rate limiting, progressive delays, and other abuse controls.

---

## 6. What is credential stuffing?

An attack where an attacker uses username/password combinations leaked from another service to attempt logins against your application.

---

## 7. What is account enumeration?

When an attacker can determine whether an account exists by observing differences in application responses.

---

## 8. Why should password reset tokens expire?

So that a stolen or leaked token can't be used indefinitely.

---

## 9. Why should password reset tokens be single-use?

To prevent someone from replaying a previously used token.

---

## 10. What is the difference between salt and pepper?

A salt is normally unique per password and can be stored alongside the password hash.

A pepper is a secret shared value that must be stored separately from the database.

---

# 50. Interview Scenario

Here's a practical backend interview question:

> Your company discovers that the users table was leaked. It contains email addresses and bcrypt password hashes. What should you do?

Don't answer:

> "We're safe because bcrypt."

Think through the incident.

You would want to consider:

```text
1. Contain the breach
        ↓
2. Determine what data was exposed
        ↓
3. Assess whether password hashes are usable for offline guessing
        ↓
4. Force password resets if appropriate
        ↓
5. Invalidate sessions/tokens where appropriate
        ↓
6. Investigate the attack path
        ↓
7. Rotate compromised secrets
        ↓
8. Monitor suspicious activity
        ↓
9. Notify affected users/regulators as required
        ↓
10. Fix the vulnerability
```

Notice something important.

Even with good password hashing, a database breach is still a serious security incident.

Security isn't about making attacks impossible.

It's about reducing the probability and impact of attacks and having a good response when something goes wrong.

---

# 51. Another Interview Scenario

> Your login API is being hit 50,000 times per minute with different passwords. CPU usage is extremely high. What could be happening?

Think:

```text
Login endpoint
      ↓
Many password verification operations
      ↓
Password hashing is intentionally expensive
      ↓
CPU gets consumed
```

This might be an automated brute-force or credential-stuffing attack.

Possible defenses:

```text
Rate limiting
+
Abuse detection
+
Progressive delays
+
IP/account/device signals
+
Monitoring
```

And this is an interesting backend lesson:

> **The security feature that protects your passwords can itself become expensive under attack.**

That's why authentication endpoints need abuse protection.

---

# 52. The Mental Model

When you're designing authentication, think in layers.

```text
                     Authentication Security
                              │
          ┌───────────────────┼───────────────────┐
          ↓                   ↓                   ↓
       Passwords           Sessions            Requests
          │                   │                   │
       Hashing             Cookies             HTTPS
       Salt                Tokens              CSRF
       Cost                Expiry              Rate limits
       MFA                 Revocation           Validation
          │                   │                   │
          └───────────────────┼───────────────────┘
                              ↓
                         Monitoring
```

Don't think:

```text
"Which JWT library should I use?"
```

Think:

> "What are all the ways an attacker could abuse this authentication system?"

That's the more useful engineering question.

---

# 53. What You Should Know After Chapter 11

You should now understand:

```text
Password
   ↓
Hash
   ↓
Salt
   ↓
Password verification
```

And:

```text
Brute force
   ↓
Rate limiting / abuse protection
```

And:

```text
Credential stuffing
   ↓
Rate limiting + monitoring + MFA/passkeys + good password practices
```

And:

```text
Password reset
   ↓
Secure random token
   ↓
Expiration
   ↓
Single use
```

And:

```text
Cookie authentication
   ↓
HTTPS
   ↓
Secure
HttpOnly
SameSite
   ↓
CSRF considerations
```

And most importantly:

> **Authentication is a complete security system, not a JWT implementation.**

---

# 54. Where We Are Now

Our backend now looks something like:

```text
                    Client
                       │
                       ↓
                     HTTPS
                       │
                       ↓
                  Web Server
                       │
                       ↓
                    Router
                       │
                       ↓
               Authentication
                       │
                       ↓
                Authorization
                       │
                       ↓
                 Business Logic
                       │
                       ↓
                    Prisma
                       │
                       ↓
                  PostgreSQL
```

We've covered the foundation of our backend:

```text
Chapter 1
How the Internet Works

Chapter 2
HTTP

Chapter 3
Web Servers

Chapter 4
Routing & URL Design

Chapter 5
JSON & Data Formats

Chapter 6
API Testing

Chapter 7
Databases

Chapter 8
SQL

Chapter 9
ORMs & Database Tools

Chapter 10
Authentication & Authorization

Chapter 11
Password Security
```

That's already enough knowledge to build a decent backend application.

But there's still a problem.

Imagine our API has:

```text
GET /jobs
```

and the database contains:

```text
10,000,000 jobs
```

We can't reasonably return all 10 million records.

Users also want:

```text
Search
Filter
Sort
Page through results
```

And clients may depend on our API for years.

So now we need to learn how to design APIs that behave well when the amount of data and number of clients starts growing.

That's where we're going next.

---

# Next — Chapter 12: APIs in the Real World

We'll take everything we've learned and make our APIs more practical:

```text
Pagination
Filtering
Sorting
Search
API versioning
Cursor pagination
Offset pagination
Response design
Error responses
Query parameters
Large datasets
API compatibility
```

And we'll answer a question that comes up in real backend work:

> **How do you design an API that still works nicely when you have millions of records and multiple clients depending on it?**