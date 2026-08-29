# Chapter 10 — Authentication & Authorization

Until now, our backend has mostly treated every request the same.

For example:

```text
GET /posts
```

The server doesn't necessarily care who sent it.

But real applications can't work like that.

Imagine:

```text
GET /my-profile
```

The server needs to know:

> Who is "my"?

Or:

```text
DELETE /posts/42
```

The server needs to ask:

> Who is trying to delete this post?

> Are they allowed to delete it?

This is where **authentication** and **authorization** come in.

They sound similar.

They're not the same thing.

And understanding that difference is the first thing we need to get right.

---

# 1. Authentication vs Authorization

Let's use a simple example.

You go to an office building.

At the entrance, the security guard asks for your ID.

You show your employee card.

The guard checks:

> "Are you actually Sumit?"

That's **authentication**.

Then you walk into the engineering department.

The receptionist checks:

> "Is Sumit allowed to enter this room?"

That's **authorization**.

So:

```text
Authentication
→ Who are you?

Authorization
→ What are you allowed to do?
```

That's the simplest way to remember it.

---

# 2. Authentication

Authentication is the process of verifying someone's identity.

For example:

```text
Email:
rahul@example.com

Password:
********
```

The server checks the credentials.

If they're correct:

```text
Authentication successful
```

Now the server knows:

```text
This request belongs to Rahul.
```

Authentication can happen in many ways:

```text
Password
Session cookie
JWT
OAuth
Passkeys
API keys
```

We'll focus on the concepts most useful for backend applications.

---

# 3. Authorization

Authorization happens **after** we know who the user is.

Suppose Rahul is authenticated.

That doesn't mean Rahul can do everything.

For example:

```text
Rahul
├── Read own profile      ✓
├── Edit own profile      ✓
├── Delete someone else's post   ✗
├── Access admin panel           ✗
```

The server needs to make those decisions.

That's authorization.

---

# 4. A Real Request

Imagine:

```http
DELETE /posts/42
```

The backend might process it like this:

```text
Request
  ↓
Who is this?
  ↓
Authentication
  ↓
Rahul
  ↓
Can Rahul delete post 42?
  ↓
Authorization
  ↓
Yes / No
```

Only after those checks should the application perform the operation.

---

# 5. Why Can't We Just Trust the Frontend?

This is a very important backend rule.

Imagine your frontend has:

```text
Delete button
```

You hide it for normal users.

But that doesn't mean the endpoint is protected.

A user can still manually send:

```http
DELETE /posts/42
```

using:

* Postman
* cURL
* browser developer tools
* their own script

The backend must enforce authorization.

Never rely on:

```text
"the frontend doesn't show the button"
```

as a security mechanism.

The frontend controls the UI.

The backend controls access to protected resources.

---

# 6. The Basic Login Flow

Let's build the simplest mental model.

A user registers:

```text
POST /auth/register
```

with:

```json
{
  "email": "rahul@example.com",
  "password": "my-password"
}
```

The server:

```text
Receive password
      ↓
Hash password
      ↓
Store password hash
      ↓
Create user
```

Later, the user logs in:

```text
POST /auth/login
```

with:

```json
{
  "email": "rahul@example.com",
  "password": "my-password"
}
```

The server:

```text
Find user
    ↓
Get stored password hash
    ↓
Compare supplied password
    ↓
Correct?
    ↓
Create authenticated session/token
```

The client then uses that authentication mechanism for future requests.

---

# 7. Never Store Plaintext Passwords

This is one of the most important rules.

Don't store:

```text
email: rahul@example.com
password: my-password
```

in your database.

If your database is compromised, every user's password is immediately exposed.

Instead, store a **password hash**.

For example:

```text
password:
my-password

↓

hashing algorithm

↓

stored hash:
$2b$12$...
```

The exact hash isn't important here.

The important thing is:

> You store the result of a password hashing algorithm, not the original password.

---

# 8. Hashing vs Encryption

Beginners often mix these up.

They're different.

### Encryption

Encryption is designed to be reversible with the appropriate key.

```text
Plaintext
   ↓
Encryption
   ↓
Encrypted data
   ↓
Decryption
   ↓
Plaintext
```

### Password hashing

Password hashing is designed to be one-way.

```text
Password
   ↓
Hash
   ↓
Stored hash
```

You don't decrypt the hash when the user logs in.

Instead, you verify whether the supplied password matches the stored hash.

We'll go much deeper into password security in Chapter 11.

For now:

```text
Passwords
→ Hash them

Don't encrypt passwords and store the encryption key somewhere.
```

---

# 9. Why Do We Need Salt?

Imagine two users both choose:

```text
password123
```

If you simply used a deterministic hash function, their hashes could be identical.

That's not ideal.

Password hashing systems use a **salt** — a unique random value associated with the password hash.

Conceptually:

```text
password
+
random salt
↓
password hashing function
↓
stored hash
```

So even if two users choose the same password, their stored hashes can differ.

Modern password-hashing libraries such as bcrypt handle salting for you.

We'll explore this properly in Chapter 11.

---

# 10. bcrypt

For password hashing in Node.js, one common choice is:

```text
bcrypt
```

You might install it with:

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

# 11. Logging In

When the user logs in:

```ts
const isValid = await bcrypt.compare(
  password,
  user.passwordHash
);
```

If:

```text
true
```

the password is correct.

If:

```text
false
```

the credentials are invalid.

Notice something important.

We aren't doing:

```text
hash(password) === storedHash
```

ourselves.

bcrypt handles the comparison using the information stored in the hash.

---

# 12. What Happens After Login?

Now we have another problem.

Suppose the user successfully logs in.

The server knows who they are.

But the next request is:

```http
GET /profile
```

HTTP itself doesn't automatically remember:

> "This is Rahul from the previous request."

Remember from Chapter 2:

> HTTP is stateless.

So we need a way for the client and server to maintain authentication across requests.

Two major approaches you'll encounter are:

```text
Sessions
Tokens
```

---

# 13. Session-Based Authentication

With sessions, the server keeps authentication state.

Imagine:

```text
User logs in
    ↓
Server creates session
    ↓
Session ID = abc123
    ↓
Browser stores session ID
```

Then future requests contain:

```text
session ID = abc123
```

The server can look up:

```text
abc123
↓
Rahul
↓
userId = 42
```

So:

```text
Browser
   ↓
session cookie
   ↓
Server
   ↓
session store
   ↓
User
```

---

# 14. What Is a Cookie?

A cookie is a small piece of data that a browser can store and send with requests to a matching website.

For example, a server can send:

```http
Set-Cookie: sessionId=abc123
```

The browser stores it.

Later, when making a request to the same site, the browser can send:

```http
Cookie: sessionId=abc123
```

This is commonly used for session-based authentication.

---

# 15. Where Is the Session Stored?

The browser might only store:

```text
sessionId = abc123
```

The actual session data can live on the server.

For example:

```text
abc123
↓
{
  userId: 42
}
```

This could be stored in:

```text
Memory
Redis
Database
```

For production systems, storing sessions only in the memory of one application server creates scaling problems.

We'll talk more about Redis and distributed systems later.

---

# 16. Token-Based Authentication

Another approach is to give the client a token representing the authenticated session/identity.

One commonly used format is:

> **JWT — JSON Web Token**

The basic flow:

```text
Login
  ↓
Server verifies credentials
  ↓
Server creates token
  ↓
Client stores token
  ↓
Client sends token with future requests
```

For example:

```http
Authorization: Bearer <token>
```

The server verifies the token and determines the identity/claims associated with it.

---

# 17. What Is a JWT?

JWT stands for:

> **JSON Web Token**

A JWT is a compact, signed representation of claims.

A typical JWT has three parts:

```text
Header.Payload.Signature
```

For example, visually:

```text
xxxxx.yyyyy.zzzzz
```

Three sections.

---

# 18. JWT Header

The header contains information about the token.

For example:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Roughly:

```text
alg
→ signing algorithm

typ
→ token type
```

Don't treat the header as secret information.

It's encoded, not encrypted.

---

# 19. JWT Payload

The payload contains **claims**.

For example:

```json
{
  "sub": "42",
  "role": "user",
  "exp": 1788000000
}
```

You might have claims such as:

```text
sub
→ subject / user identifier

exp
→ expiration time

role
→ application-specific role
```

The exact claims depend on your application.

---

# 20. JWT Signature

The signature is what lets the server verify that the token was created/signed by a trusted party and hasn't been modified.

Conceptually:

```text
Header
+
Payload
+
Secret/private key
↓
Signature
```

When the server receives the token, it verifies the signature.

If someone changes:

```json
{
  "role": "user"
}
```

to:

```json
{
  "role": "admin"
}
```

the signature won't match if the token hasn't been legitimately re-signed.

The server should reject it.

---

# 21. JWT Is Not Encryption

This is a very common interview question.

A normal JWT is **encoded and signed**, not encrypted.

That means you should not put sensitive secrets into the payload assuming they're hidden.

For example, don't put:

```json
{
  "password": "my-secret-password"
}
```

in a normal JWT.

Someone who possesses the token can generally decode its header and payload.

The signature protects integrity/authenticity.

It does not automatically provide confidentiality.

---

# 22. Base64URL Is Not Encryption

JWT parts are represented using Base64URL encoding.

Encoding isn't encryption.

For example:

```text
Hello
```

can be encoded into another representation.

Anyone can decode it.

So:

```text
encoded
≠
secret
```

This distinction is important.

---

# 23. Access Tokens

An access token is used to access protected resources.

For example:

```http
GET /profile
Authorization: Bearer eyJ...
```

The backend verifies the token.

Then:

```text
Token valid
   ↓
User identified
   ↓
Continue
```

If invalid or expired:

```text
401 Unauthorized
```

---

# 24. Authentication Middleware

This is where the concepts from Chapter 3 come back.

We can create authentication middleware.

Conceptually:

```ts
async function authenticate(req, res, next) {
  // get token
  // verify token
  // identify user
  // attach user to request
  // continue
}
```

Then:

```ts
app.get(
  "/profile",
  authenticate,
  getProfile
);
```

The flow becomes:

```text
Request
  ↓
authenticate middleware
  ↓
valid?
  ↓
attach user
  ↓
controller
```

This is a very common backend pattern.

---

# 25. Authentication Middleware Doesn't Mean Authorization

This distinction is worth repeating.

Suppose:

```text
authenticate()
```

runs successfully.

Now we know:

```text
userId = 42
```

But that doesn't mean the user can delete any post.

We still need authorization.

For example:

```ts
if (post.userId !== req.user.id) {
  return res.status(403).json({
    message: "Forbidden"
  });
}
```

Now we're asking:

> Is this authenticated user allowed to perform this action?

---

# 26. 401 vs 403

This is a common interview question.

### 401 Unauthorized

The request does not have valid authentication credentials.

Examples:

```text
Missing token
Invalid token
Expired token
Invalid session
```

Think:

> **I don't know who you are.**

### 403 Forbidden

The server knows who you are, but you aren't allowed to perform the action.

Think:

> **I know who you are, but you can't do this.**

So:

```text
401
→ Authentication problem

403
→ Authorization problem
```

---

# 27. Role-Based Authorization

Suppose our application has:

```text
user
admin
moderator
```

We can define permissions.

For example:

```text
User
→ create posts

Moderator
→ delete inappropriate posts

Admin
→ manage users
```

Now authentication gives us:

```text
userId = 42
```

Authorization can determine:

```text
role = admin
```

and then decide whether the operation is allowed.

---

# 28. Role-Based Access Control

This is commonly called:

> **RBAC — Role-Based Access Control**

Conceptually:

```text
User
 ↓
Role
 ↓
Permissions
```

For example:

```text
Rahul
 ↓
Admin
 ↓
delete users
manage posts
view reports
```

The exact permission model depends on the application.

Don't automatically put `role: "admin"` into a JWT and assume that's enough.

Your authorization design needs to account for changes to roles and permissions.

---

# 29. Sessions vs JWT

Now the classic interview question.

### Sessions

```text
Client
 ↓
Session ID
 ↓
Server
 ↓
Session store
```

The server maintains session state.

### JWT

```text
Client
 ↓
JWT
 ↓
Server verifies token
```

The token contains claims and is cryptographically verified.

A simplified comparison:

|                           | Sessions                                           | JWT                                              |
| ------------------------- | -------------------------------------------------- | ------------------------------------------------ |
| Server-side session state | Usually yes                                        | Often less/none for access-token validation      |
| Client sends              | Session ID                                         | Token                                            |
| Revocation                | Usually straightforward                            | Can be more complicated if tokens are long-lived |
| Scaling                   | Need shared session storage or appropriate routing | Can simplify some stateless validation patterns  |
| Token size                | Small session ID                                   | Larger                                           |
| Complexity                | Often simpler for browser apps                     | Useful in some distributed/API architectures     |

There isn't a universal winner.

---

# 30. JWT Is Not Automatically Better

This is something I really want beginners to avoid.

You may see tutorials saying:

> "JWT is modern, sessions are old."

That's not good engineering advice.

Sessions are still a very useful authentication approach.

JWTs are useful in certain architectures.

The right choice depends on things like:

```text
Application type
Client type
Session requirements
Revocation requirements
Scaling architecture
Security model
```

Choose based on the problem.

---

# 31. Access Token vs Refresh Token

You will often hear about two token types.

### Access token

Used to access APIs.

Usually short-lived.

```text
Access token
→ 5 minutes
→ 15 minutes
→ 1 hour
```

The exact lifetime depends on the system.

### Refresh token

Used to obtain a new access token after the access token expires.

Conceptually:

```text
Login
 ↓
Access token + Refresh token
 ↓
Access token expires
 ↓
Send refresh token
 ↓
Server validates it
 ↓
New access token
```

This lets access tokens remain relatively short-lived.

---

# 32. Why Not Make Access Tokens Last Forever?

Imagine:

```text
Access token lifetime
→ 30 days
```

If someone steals that token, it may remain usable for a long time.

Shorter-lived access tokens reduce the window in which a stolen token can be used.

But this creates another problem:

> How do we get a new token without asking the user to log in again?

That's one reason refresh tokens exist.

---

# 33. Where Should Tokens Be Stored?

This is a more nuanced topic than:

> "Always use localStorage."

Don't blindly follow that advice.

For browser applications, authentication storage needs to consider threats such as XSS and CSRF.

A common secure pattern is to use cookies with appropriate attributes.

For example:

```text
HttpOnly
Secure
SameSite
```

### HttpOnly

JavaScript running in the browser can't directly read the cookie.

This can reduce the impact of some token-stealing scenarios involving XSS.

### Secure

The cookie should only be sent over HTTPS.

### SameSite

Controls when the browser sends the cookie in cross-site contexts and can help mitigate some CSRF scenarios.

The exact configuration depends on your application.

---

# 34. Cookies vs localStorage

A common beginner comparison:

```text
Cookie
vs
localStorage
```

Don't reduce it to:

> "Cookies good, localStorage bad."

The real question is:

> **What threats does my authentication design need to handle?**

If authentication is cookie-based, you need to think carefully about CSRF.

If tokens are accessible to JavaScript, you need to think carefully about XSS and token theft.

Security is about the complete design, not one storage API.

---

# 35. What Is OAuth?

Now let's talk about:

> **Login with Google**

You've probably seen:

```text
Continue with Google
Continue with GitHub
Continue with Microsoft
```

That's commonly built using **OAuth 2.0** and, for user authentication/identity, often **OpenID Connect (OIDC)** on top of OAuth.

OAuth itself is primarily an authorization framework.

This distinction matters.

OAuth can be used to allow an application to access resources on behalf of a user.

OpenID Connect adds an identity layer for authentication.

---

# 36. The Basic OAuth/OIDC Idea

Imagine your application:

```text
My App
```

and Google:

```text
Identity Provider
```

Instead of asking the user for their Google password, your application sends them to Google's authorization flow.

Conceptually:

```text
Your App
   ↓
Google
   ↓
User authenticates
   ↓
User grants requested access
   ↓
Google redirects back
   ↓
Your App receives authorization result
```

Your application never needs the user's Google password.

That's the important idea.

---

# 37. Why OAuth Exists

Imagine an application asks:

> "Give me your Google password so I can access your Google account."

That's obviously a terrible design.

OAuth lets the user authorize an application without giving the application their provider password.

Modern identity flows are more nuanced than the simple diagram above, but that's the core idea.

---

# 38. Don't Say "OAuth = Login With Google"

That's another common interview mistake.

A better answer:

> "OAuth 2.0 is an authorization framework. For login and identity, applications commonly use OpenID Connect on top of OAuth 2.0."

That's a much stronger answer.

---

# 39. Authentication Flow for Our Backend

Let's put everything together.

A simple email/password system might look like:

```text
                REGISTER
                   │
                   ↓
             Email + Password
                   │
                   ↓
             Validate input
                   │
                   ↓
             Hash password
                   │
                   ↓
              PostgreSQL
                   │
                   ↓
                User
```

Then login:

```text
                 LOGIN
                   │
                   ↓
             Email + Password
                   │
                   ↓
              Find user
                   │
                   ↓
          Compare password hash
                   │
              ┌────┴────┐
              │         │
            Invalid    Valid
              │         │
             401        ↓
                    Create session
                    or token
                         │
                         ↓
                       Client
```

Then protected request:

```text
GET /profile
      │
      ↓
Authentication
      │
      ↓
Identify user
      │
      ↓
Authorization
      │
      ↓
Business logic
      │
      ↓
Database
      │
      ↓
Response
```

This is the mental model I want you to keep.

---

# 40. Building Authentication Into Our Forum

Let's return to our forum project.

We already have:

```text
users
posts
comments
votes
```

Now we'll add authentication.

Our user model might contain:

```text
users
--------------------
id
username
email
password_hash
created_at
```

Notice:

```text
password_hash
```

not:

```text
password
```

---

# 41. Registration Endpoint

We can create:

```http
POST /auth/register
```

Request:

```json
{
  "username": "rahul",
  "email": "rahul@example.com",
  "password": "some-password"
}
```

Backend flow:

```text
Request
  ↓
Validate input
  ↓
Check email
  ↓
Hash password
  ↓
Create user
  ↓
Return safe user data
```

Notice that we don't return:

```json
{
  "passwordHash": "..."
}
```

to the client.

Password hashes are sensitive information too.

---

# 42. Login Endpoint

Create:

```http
POST /auth/login
```

Request:

```json
{
  "email": "rahul@example.com",
  "password": "some-password"
}
```

Flow:

```text
Request
  ↓
Validate input
  ↓
Find user
  ↓
Compare password
  ↓
Create session/token
  ↓
Return authentication result
```

---

# 43. Protected Route

Now:

```http
GET /profile
```

requires authentication.

Conceptually:

```ts
app.get(
  "/profile",
  authenticate,
  getProfile
);
```

If no valid authentication:

```http
401 Unauthorized
```

If authenticated:

```text
req.user
```

might contain:

```js
{
  id: 42,
  email: "rahul@example.com"
}
```

Then your controller can use that identity.

---

# 44. Authorization Example

Suppose:

```http
DELETE /posts/42
```

The authentication middleware identifies:

```text
userId = 10
```

The database says:

```text
post.userId = 20
```

Now:

```text
10 !== 20
```

The user doesn't own the post.

So:

```http
403 Forbidden
```

The request is authenticated.

But it's not authorized.

That's the difference in a real example.

---

# 45. Don't Put Authorization Only in the Route Name

This isn't security:

```text
/admin/delete-user
```

just because it contains `/admin`.

Anyone can send the request.

Authorization must be enforced by backend logic.

For example:

```text
Request
  ↓
Authenticate
  ↓
Identify user
  ↓
Check permissions
  ↓
Allow / deny
```

---

# 46. Common Authentication Mistakes

## Mistake 1 — Storing plaintext passwords

Never do this.

Store password hashes using an appropriate password hashing algorithm.

---

## Mistake 2 — Returning password hashes

Even hashed passwords should not casually be returned in API responses.

---

## Mistake 3 — Trusting the frontend

Hiding buttons isn't authorization.

The backend must enforce permissions.

---

## Mistake 4 — Thinking JWT payload is secret

It's not.

A normal JWT payload is readable.

Don't put secrets in it.

---

## Mistake 5 — Making JWTs live forever

Long-lived access tokens increase the impact of token theft.

Use an appropriate expiration strategy.

---

## Mistake 6 — Confusing authentication with authorization

Remember:

```text
Authentication
→ Who are you?

Authorization
→ Can you do this?
```

---

## Mistake 7 — Treating OAuth as "a login API"

OAuth is an authorization framework.

OIDC is commonly used when you need identity/authentication on top of OAuth.

---

## Mistake 8 — Building auth without thinking about attacks

Authentication is security-sensitive.

Think about:

```text
Brute force
Credential stuffing
XSS
CSRF
Token theft
Session theft
Password reuse
Account enumeration
```

We'll go deeper into several of these in Chapter 11.

---

# 47. Mini Project — Add Authentication to the Forum API

Now let's make our forum feel like a real application.

Add:

```text
POST /auth/register
POST /auth/login
POST /auth/logout
GET  /profile
```

Then protect:

```text
POST   /posts
PATCH  /posts/:id
DELETE /posts/:id
POST   /posts/:id/comments
```

The flow should be:

```text
Register
   ↓
Login
   ↓
Authenticated
   ↓
Create post
   ↓
Edit own post
   ↓
Try editing someone else's post
   ↓
403
```

That's a much better exercise than simply copying a JWT tutorial.

---

# 48. Authentication Checklist

Before calling your authentication system "done", ask:

```text
□ Are passwords hashed?

□ Are password hashes excluded from responses?

□ Is input validated?

□ Are authentication failures handled?

□ Are protected routes actually protected?

□ Is authorization checked on sensitive actions?

□ Are tokens/session identifiers protected?

□ Are authentication credentials transmitted over HTTPS?

□ Do authentication credentials expire appropriately?

□ Can users log out?

□ Can compromised sessions/tokens be revoked where necessary?

□ Is brute-force protection considered?

□ Are secrets stored outside source code?
```

You don't need to solve every item perfectly in your first project.

But you should know that they exist.

---

# 49. Interview Questions

## 1. Authentication vs authorization?

**Authentication** verifies who a user is.

**Authorization** determines what that authenticated user is allowed to do.

---

## 2. Why shouldn't passwords be stored directly?

Because if the database is compromised, plaintext passwords would immediately be exposed.

Passwords should be stored using a suitable password hashing algorithm.

---

## 3. Hashing vs encryption?

Encryption is designed to be reversible with a key.

Password hashing is designed to be one-way and is used for securely storing password verifiers.

---

## 4. What is a salt?

A salt is a unique random value used with password hashing so identical passwords don't result in identical stored hashes and to make certain precomputed attacks harder.

---

## 5. What is JWT?

JWT is a compact, signed representation of claims that can be used in token-based authentication and authorization systems.

---

## 6. What are the three parts of a JWT?

```text
Header
Payload
Signature
```

They are separated by dots.

---

## 7. Is JWT encrypted?

Not normally.

JWTs are commonly encoded and signed, not encrypted.

The payload should therefore not contain sensitive secrets.

---

## 8. 401 vs 403?

```text
401
→ authentication is missing/invalid

403
→ authenticated but not allowed
```

---

## 9. Session vs JWT?

A session typically uses a session identifier that references server-side session state.

A JWT carries signed claims that can be validated by the server.

Neither approach is universally better.

---

## 10. What is OAuth?

OAuth 2.0 is an authorization framework that allows applications to obtain limited access to resources without requiring the user's credentials to be shared with the application.

For authentication/identity, OpenID Connect is commonly used on top of OAuth 2.0.

---

# 50. Interview Scenario

Here's a more realistic one.

> A user logs in successfully. You issue a JWT. Five minutes later, the user's account is disabled by an administrator. The JWT hasn't expired yet. Can the user still access your API?

There isn't one universal answer.

It depends on your authentication architecture.

If your API only validates a self-contained access token and doesn't check current account status, the token may continue working until it expires.

That creates a tradeoff.

You could:

```text
Short-lived access tokens
+
Refresh token controls
```

or introduce server-side checks/revocation mechanisms depending on your requirements.

This is why authentication isn't simply:

```text
JWT = solved
```

Real systems have to think about:

```text
Expiration
Revocation
Session state
Account status
Permissions
Token theft
```

---

# 51. Another Interview Scenario

> A user can edit another user's post by changing `/posts/10` to `/posts/11`. What's wrong?

The server is probably authenticating the user but not properly checking authorization.

The backend needs to verify something like:

```text
Current user owns post
OR
Current user has permission to edit it
```

before performing the update.

This is an authorization bug.

The frontend isn't the security boundary.

---

# 52. The Authentication Mental Model

When you see a protected endpoint:

```text
POST /posts
```

think:

```text
Request
   ↓
Who is making this request?
   ↓
Authentication
   ↓
User identified
   ↓
Is this user allowed to create a post?
   ↓
Authorization
   ↓
Validate request
   ↓
Business logic
   ↓
Database
   ↓
Response
```

Authentication and authorization are just two pieces of the larger request-processing pipeline.

---

# 53. What You Should Be Able To Explain Now

You should be able to explain:

```text
Authentication
→ Who are you?

Authorization
→ What can you do?

Password hashing
→ How we safely store password verifiers

Session
→ Server-side authentication state referenced by a session ID

JWT
→ Signed token containing claims

Cookie
→ Browser mechanism for storing/sending small pieces of data

Access token
→ Credential used to access protected resources

Refresh token
→ Credential used to obtain new access tokens

401
→ Authentication problem

403
→ Authorization problem

OAuth
→ Authorization framework

OIDC
→ Identity/authentication layer built on OAuth 2.0
```

More importantly, you should understand how these pieces fit together.

---

# 54. Where We Are Now

Our backend has evolved again.

Previously:

```text
Client
   ↓
HTTP
   ↓
Web Server
   ↓
Router
   ↓
Database
   ↓
Response
```

Now:

```text
Client
   ↓
HTTP
   ↓
Web Server
   ↓
Authentication
   ↓
Authorization
   ↓
Router
   ↓
Business Logic
   ↓
Prisma
   ↓
PostgreSQL
   ↓
Response
```

We're getting much closer to a real application.

But there's still one big problem.

We've learned how to authenticate users.

We've learned that passwords should be hashed.

But authentication is security-sensitive enough that we need to go deeper.

What exactly makes a password hash secure?

Why bcrypt instead of SHA-256?

What's the point of a salt?

What is a pepper?

How do attackers brute-force passwords?

What is credential stuffing?

How should login endpoints be rate-limited?

How do we prevent account enumeration?

And what happens if someone keeps trying:

```text
POST /auth/login
```

10,000 times?

That's next.

---

# Next — Chapter 11: Password Security & Best Practices

We'll focus specifically on protecting authentication systems:

```text
Password hashing
bcrypt
Salt
Pepper
Brute-force attacks
Credential stuffing
Rate limiting
Account enumeration
Secure password reset
Session/token security
HTTPS
Common authentication mistakes
```

The goal isn't just:

> "Use bcrypt."

The goal is to understand **why** these protections exist and what problem each one solves.