# Chapter 18 — Testing Your Backend

Let's be honest.

When you're building a backend for the first time, testing can feel like extra work.

You write an API:

```http
POST /api/users
```

You open Postman.

Send a request.

It works.

You think:

> "Done."

Then you change something three days later.

You change authentication.

Something else breaks.

You don't notice.

You deploy.

Now a user tells you:

> "I can't log in."

You fix that.

Then someone says:

> "Creating posts stopped working."

And now you're manually testing everything again.

This is where testing becomes important.

Testing isn't about proving your code is perfect.

It is about giving you **confidence that your code behaves the way you expect, especially when you change it.**

---

# 1. What Is Testing?

At the simplest level:

> **Testing means checking whether your software behaves as expected.**

Suppose we have:

```http
GET /api/users/123
```

We expect:

```text
User exists
   ↓
200 OK
   ↓
User data
```

A test can verify exactly that.

We can also test:

```text
User doesn't exist
   ↓
404 Not Found
```

And:

```text
Invalid ID
   ↓
400 Bad Request
```

Testing isn't just:

> "Does the happy path work?"

It's also:

> "What happens when things go wrong?"

---

# 2. Why Do We Need Tests?

Imagine your backend has:

```text
Authentication
Users
Posts
Comments
Payments
File uploads
Queues
```

You change this:

```text
authentication middleware
```

That change could accidentally affect:

```text
Posts
Comments
Payments
Files
```

Without tests, you have to manually check everything.

With tests:

```text
Change code
   ↓
Run tests
   ↓
Tests fail
   ↓
Something broke
```

This is one of the biggest benefits of automated testing.

---

# 3. Tests Are Not Just For Bugs

A common misunderstanding is:

> "We write tests after bugs happen."

Not exactly.

Tests help you:

```text
Catch bugs
Prevent regressions
Refactor safely
Document expected behavior
Understand existing code
Deploy with more confidence
```

---

# 4. What Is a Regression?

Suppose this works today:

```text
POST /api/users
```

Then you add a new feature.

After the change:

```text
POST /api/users
```

stops working.

You didn't intentionally break it.

That's a:

> **Regression**

Automated tests can catch regressions.

---

# 5. A Very Simple Example

Suppose we have:

```js
function add(a, b) {
  return a + b;
}
```

A test could be:

```js
expect(add(2, 3)).toBe(5);
```

We're saying:

> "When I give this function 2 and 3, I expect 5."

That's the basic idea behind every automated test.

The systems get more complicated.

The idea doesn't.

---

# 6. What Makes a Good Test?

A good test usually has:

```text
Arrange
Act
Assert
```

Let's understand that.

### Arrange

Prepare what you need.

```js
const user = {
  name: "Rahul"
};
```

### Act

Perform the operation.

```js
const result = createUser(user);
```

### Assert

Check the result.

```js
expect(result.name).toBe("Rahul");
```

So:

```text
Arrange
   ↓
Act
   ↓
Assert
```

You'll see this pattern everywhere.

---

# 7. Unit Tests

Let's start with the smallest type:

> **Unit test**

A unit test tests a small isolated piece of code.

Usually:

```text
function
class
small module
```

For example:

```js
function calculateTotal(price, quantity) {
  return price * quantity;
}
```

Test:

```js
test("calculates total price", () => {
  const result = calculateTotal(100, 3);

  expect(result).toBe(300);
});
```

We're not testing:

```text
HTTP
Database
Redis
Network
```

Just:

```text
calculateTotal()
```

---

# 8. Why Unit Tests Are Useful

Unit tests are usually:

```text
Fast
Small
Focused
Easy to run
```

If you have:

```text
1,000 unit tests
```

you don't want every test to:

```text
Start database
Start Redis
Call external APIs
Wait for network
```

That would be painfully slow.

Unit tests are useful for testing business logic in isolation.

---

# 9. What Should You Unit Test?

Good candidates:

```text
Price calculations
Validation rules
Permission logic
Data transformations
Utility functions
Business rules
Formatting
Algorithms
```

For example:

```js
function canDeletePost(user, post) {
  return user.id === post.authorId || user.role === "admin";
}
```

This is excellent unit-test material.

You can test:

```text
Author → allowed
Admin → allowed
Random user → denied
```

---

# 10. Integration Tests

Now things get more interesting.

An:

> **Integration test**

checks that multiple pieces work together.

For example:

```text
Service
   ↓
PostgreSQL
```

You might test:

```text
Create user
   ↓
Database
   ↓
Read user
```

Now you're testing the interaction between your application and the database.

---

# 11. Unit vs Integration

Think:

### Unit

```text
Function
 ↓
Result
```

### Integration

```text
Application
 ↓
Database
 ↓
Result
```

A unit test might say:

```text
"Does this function calculate the total correctly?"
```

An integration test might say:

```text
"Can my application actually create and retrieve an order from PostgreSQL?"
```

Both are valuable.

---

# 12. API Tests

Backend developers often spend a lot of time testing APIs.

For example:

```http
POST /api/login
```

You might test:

### Valid credentials

```text
200 OK
```

### Wrong password

```text
401 Unauthorized
```

### Missing email

```text
400 Bad Request
```

### Unknown user

```text
401/404
```

depending on your API design.

This tests the API behavior from the HTTP layer.

---

# 13. Supertest

If you're using Node.js and Express, a common tool is:

> **Supertest**

It allows you to make HTTP requests against your application during tests.

For example:

```js
const response = await request(app)
  .get("/api/users");

expect(response.status).toBe(200);
```

Now you're testing your actual HTTP routes.

---

# 14. Why Not Just Use Postman?

Postman is great.

We covered it in Chapter 6.

But imagine having:

```text
100 API endpoints
```

and manually testing every endpoint after every code change.

Not fun.

Automated tests let you do:

```bash
npm test
```

and check hundreds or thousands of scenarios automatically.

Postman is useful for exploration and manual testing.

Automated tests are useful for repeatable verification.

---

# 15. End-to-End Tests

An:

> **End-to-End (E2E) test**

tests a larger part of the system from the perspective of a real workflow.

For example:

```text
Register
   ↓
Login
   ↓
Create post
   ↓
Read post
   ↓
Update post
   ↓
Delete post
```

That's an entire user journey.

You're testing multiple components together.

---

# 16. Testing Levels

You can visualize the different levels like this:

```text
          E2E
       ─────────
      Integration
    ───────────────
        Unit
────────────────────
```

Unit tests:

```text
small + fast
```

Integration tests:

```text
larger + slower
```

E2E tests:

```text
largest + usually slowest
```

This leads to the famous:

> **Testing Pyramid**

---

# 17. The Testing Pyramid

A simplified testing pyramid looks like:

```text
             /\
            /  \
           / E2E\
          /──────\
         /        \
        /Integration\
       /────────────\
      /              \
     /   Unit Tests   \
    /──────────────────\
```

The idea is:

```text
Lots of unit tests
Some integration tests
Fewer expensive E2E tests
```

Why?

Because unit tests are generally faster and cheaper to run.

But don't interpret the pyramid as:

> "Never write E2E tests."

That's not the point.

You need different types of tests for different risks.

---

# 18. A Better Mental Model

Don't ask:

> "How many unit tests should I have?"

Ask:

> "What risks do I need confidence about?"

For example:

```text
Business calculation
→ unit test

Database behavior
→ integration test

Authentication endpoint
→ API/integration test

Complete checkout flow
→ E2E test
```

Choose the test type based on what you're trying to verify.

---

# 19. Testing Authentication

Authentication is an excellent thing to test.

Suppose:

```http
POST /api/login
```

Test:

```text
Correct email + password
→ success
```

Wrong password:

```text
→ failure
```

Unknown user:

```text
→ failure
```

Missing credentials:

```text
→ validation error
```

Then test protected routes:

```text
No token
→ 401
```

Invalid token:

```text
→ 401
```

Valid token:

```text
→ allowed
```

---

# 20. Testing Authorization

Authentication asks:

> "Who are you?"

Authorization asks:

> "Are you allowed to do this?"

Suppose:

```text
User A
owns Post 123
```

User B tries:

```http
DELETE /posts/123
```

Expected:

```text
403 Forbidden
```

Test it.

This catches security regressions.

---

# 21. Testing Database Code

Suppose:

```js
await db.user.create(...)
```

You have a few choices.

### Option 1

Mock the database.

### Option 2

Use a real test database.

### Option 3

Use an isolated temporary database/container.

Which one is better?

It depends on what you're testing.

---

# 22. Mocking

A:

> **Mock**

is a fake version of a dependency used during a test.

Imagine:

```text
Your service
   ↓
Payment API
```

You don't want your test to actually charge a credit card.

So you create a fake payment service:

```text
Your service
   ↓
Fake Payment API
```

The fake can say:

```text
"Payment succeeded"
```

or:

```text
"Payment failed"
```

on demand.

---

# 23. Why Use Mocks?

Mocks can make tests:

```text
Fast
Predictable
Isolated
```

Suppose your external payment API is down.

You don't want your test suite to fail because a third-party service is having a bad day.

Instead:

```text
Mock
 ↓
simulate success/failure
```

---

# 24. But Don't Mock Everything

This is a very common mistake.

Imagine your entire test suite is:

```text
Mock database
Mock Redis
Mock queue
Mock payment API
Mock filesystem
Mock everything
```

Tests pass.

But production breaks.

Why?

Because you've never tested the real integrations.

Mocks answer:

> "Does my code behave correctly given this fake dependency?"

They don't necessarily answer:

> "Does my application actually work with PostgreSQL?"

That's why integration tests matter.

---

# 25. Mock vs Stub

You'll hear both terms.

Very simply:

### Stub

Provides predetermined data.

```text
getUser()
→ returns fake user
```

### Mock

Often also verifies how something was used.

For example:

```text
Was sendEmail() called?
How many times?
With what arguments?
```

People sometimes use these words loosely, so don't get stuck on terminology.

The bigger concept is:

> **Replace a real dependency with a controlled test double.**

---

# 26. Fake

A fake is another type of test double.

For example:

```text
Real database
```

might be replaced with:

```text
In-memory fake database
```

A fake usually has some working behavior rather than simply returning one hard-coded result.

Again, the terminology can vary between teams.

Understand the purpose rather than obsessing over labels.

---

# 27. Testing External APIs

Suppose your application calls:

```text
Stripe
SendGrid
GitHub API
Google Maps
```

You usually don't want every test to make real network requests.

Instead:

```text
Test
 ↓
Mock HTTP client
 ↓
Fake response
```

Test scenarios such as:

```text
200 success
400 error
401 error
429 rate limit
500 server error
Timeout
Slow response
Malformed response
```

This is much more useful than only testing success.

---

# 28. Testing Failure Is More Important Than You Think

Imagine:

```text
Database works
Redis works
Payment works
Everything works
```

Your tests pass.

Production:

```text
Redis goes down
```

What happens?

You should have a test for that if your application has a defined fallback behavior.

For example:

```text
Redis unavailable
   ↓
Log warning
   ↓
Fallback to database
```

Test it.

---

# 29. Testing Timeouts

External service:

```text
Payment API
```

doesn't respond.

Your code should eventually stop waiting.

Test:

```text
Payment API
   ↓
timeout
```

and verify your application:

```text
returns appropriate error
logs failure
doesn't hang forever
```

---

# 30. Testing Retries

Suppose your code retries three times.

Test:

```text
Attempt 1 → fail
Attempt 2 → fail
Attempt 3 → success
```

Expected:

```text
success
```

Also:

```text
Attempt 1 → fail
Attempt 2 → fail
Attempt 3 → fail
```

Expected:

```text
final failure
```

You should also verify that retries don't continue forever.

---

# 31. Testing Rate Limiting

From our previous chapters:

```text
Rate limiting
```

protects your API from excessive requests.

Test:

```text
Request 1 → 200
Request 2 → 200
Request 3 → 200
...
Limit reached
→ 429 Too Many Requests
```

Then test what happens after the rate-limit window resets.

---

# 32. Testing File Uploads

From Chapter 16:

Suppose your endpoint allows:

```text
JPEG
PNG
max 5 MB
```

Test:

```text
valid JPEG
→ success
```

Then:

```text
10 MB JPEG
→ reject
```

Then:

```text
video.mp4
→ reject
```

Then:

```text
malformed file
→ reject
```

Then:

```text
no file
→ appropriate error
```

Security-related validation deserves tests.

---

# 33. Testing Queues

Suppose:

```text
POST /emails
```

doesn't send an email immediately.

Instead:

```text
API
 ↓
Queue
 ↓
Worker
 ↓
Email Provider
```

Your test might verify:

```text
API request
 ↓
job added to queue
```

Then another test verifies:

```text
Worker
 ↓
processes job
 ↓
email provider called
```

You don't necessarily need one giant test for everything.

Test responsibilities separately.

---

# 34. Testing WebSockets

From Chapter 15:

Suppose a chat server sends:

```text
message
```

to a room.

Test:

```text
Client A joins room
Client B joins room

Client A sends message

Client B receives message
```

Then test:

```text
Client C
not in room
```

should not receive the private room message.

That's both functionality and authorization testing.

---

# 35. Test Isolation

A very important concept:

> **One test shouldn't accidentally affect another test.**

Bad:

```text
Test 1 creates user "rahul@example.com"

Test 2 expects no user with that email
```

Now Test 2 depends on Test 1.

That's fragile.

Tests should be isolated.

---

# 36. Database Test Isolation

You might use:

```text
Transaction rollback
Database reset
Separate test database
Temporary database
Containers
```

The exact approach depends on your stack.

The goal:

```text
Test A
 ↓
changes database
 ↓
cleanup

Test B
 ↓
starts clean
```

---

# 37. Test Data

You need test data.

For example:

```js
const user = {
  email: "test@example.com",
  password: "password123"
};
```

But as your application grows, manually creating objects everywhere becomes annoying.

You might use:

> **Factories**

For example:

```js
const user = createUserFixture();
```

which gives you valid default data.

Then customize what you need.

---

# 38. Fixtures

A:

> **Fixture**

is predefined test data or setup used by tests.

Example:

```js
const userFixture = {
  id: "user_123",
  email: "test@example.com",
  role: "user"
};
```

Fixtures can make tests easier to read.

But don't create giant fixtures containing 50 unrelated fields if your test only needs two.

Keep test data focused.

---

# 39. Test Naming

This:

```js
test("works", () => {});
```

isn't useful.

Better:

```js
test("returns 404 when user does not exist", () => {});
```

Even better:

```js
test("GET /users/:id returns 404 when the user does not exist", () => {});
```

A good test name tells you:

```text
What happened?
Under what condition?
What should happen?
```

---

# 40. A Test Should Have One Main Reason to Fail

Imagine:

```text
test("everything works")
```

and it checks:

```text
login
create post
update post
delete post
send email
```

When it fails, what broke?

You don't know.

Better:

```text
login succeeds with valid credentials
```

and:

```text
login fails with invalid password
```

and:

```text
creating a post requires authentication
```

Smaller tests are easier to understand.

---

# 41. Don't Test Implementation Details Too Much

Suppose your function currently uses:

```js
array.map()
```

Your test shouldn't care.

If you change:

```js
map()
```

to:

```js
for...of
```

the behavior should remain the same.

Tests should generally verify:

> **What the code does**

rather than:

> **Exactly how the code is written internally**

This makes refactoring safer.

---

# 42. Example

Suppose:

```js
function isAdult(age) {
  return age >= 18;
}
```

Good:

```js
expect(isAdult(20)).toBe(true);
expect(isAdult(15)).toBe(false);
```

You don't need a test saying:

```text
"function uses >= operator"
```

That's an implementation detail.

---

# 43. Edge Cases

Good engineers think about edge cases.

Suppose:

```js
calculateDiscount(price, percentage)
```

Don't only test:

```text
100, 10 → 90
```

Also think:

```text
0
negative numbers
100%
greater than 100%
decimal values
missing values
very large numbers
```

The exact expected behavior should be defined by your business rules.

---

# 44. Boundary Testing

A very useful technique is testing around boundaries.

Suppose:

```text
Maximum upload size = 5 MB
```

Don't only test:

```text
1 MB
```

Test:

```text
4.99 MB
5 MB
5.01 MB
```

Because bugs often live at boundaries.

Same for:

```text
Rate limits
Pagination
Age restrictions
Character limits
Amounts
Dates
```

---

# 45. Property-Based Thinking

You don't need a property-testing library to understand the idea.

Instead of asking:

```text
Does 2 + 3 = 5?
```

you can think:

> Addition should be commutative.

Meaning:

```text
a + b = b + a
```

For many inputs.

This way of thinking can reveal classes of bugs instead of one example.

It's an advanced technique, but useful to know.

---

# 46. Test Coverage

You'll hear:

> **Code coverage**

It measures how much of your code was executed by tests.

For example:

```text
80% coverage
```

roughly means tests executed 80% of the measured code.

But:

> **80% coverage does NOT mean your application is 80% correct.**

You can have:

```text
100% coverage
```

with terrible tests.

---

# 47. How 100% Coverage Can Still Be Bad

Suppose:

```js
function login(password) {
  if (password === "secret") {
    return true;
  }

  return false;
}
```

A test could execute both branches.

Coverage:

```text
100%
```

But your tests might still fail to check:

```text
empty password
very long password
unexpected input
security behavior
```

Coverage tells you:

> "Which code ran?"

It doesn't tell you:

> "Did we test the right behavior?"

---

# 48. Don't Chase 100% Blindly

If a team says:

> "Every line must be covered."

you might end up writing meaningless tests just to make the number increase.

Instead:

> Test important behavior and important failure modes.

Coverage is a useful signal.

It's not the goal.

---

# 49. Test the Risky Stuff First

Suppose your application has:

```text
Authentication
Payments
Permissions
Money calculations
File access
Data deletion
```

These deserve serious testing.

A tiny formatting helper probably doesn't carry the same risk.

Testing effort should roughly follow:

```text
Risk
Impact
Complexity
```

Not just:

```text
lines of code
```

---

# 50. Test Doubles

You'll hear a few terms:

```text
Mock
Stub
Fake
Spy
```

A:

### Stub

Returns controlled data.

### Mock

Can verify expected interactions.

### Fake

Simplified working implementation.

### Spy

Records how something was called.

For example:

```text
Was sendEmail() called?
How many times?
With which arguments?
```

Again, terminology varies between libraries.

The important concept is:

> **Test doubles let you control dependencies during tests.**

---

# 51. Testing Time

Time causes surprising bugs.

Suppose:

```text
Token expires in 15 minutes
```

Don't make your test actually wait 15 minutes.

Use a fake/controlled clock where your testing tools support it.

Then test:

```text
Token valid
Token expires
Token rejected
```

without waiting.

---

# 52. Testing Randomness

Suppose your application generates:

```text
random IDs
```

Tests shouldn't depend on one exact random result.

Instead test properties:

```text
ID exists
ID has expected format
IDs are not accidentally reused
```

If needed, control randomness during tests.

---

# 53. Testing Authentication Tokens

Suppose you use JWT.

Test:

```text
Valid token
→ success
```

Expired:

```text
→ reject
```

Modified payload:

```text
→ reject
```

Missing token:

```text
→ reject
```

Wrong signing secret:

```text
→ reject
```

This is much more valuable than testing only:

```text
JWT exists
```

---

# 54. Testing Passwords

Suppose your authentication system uses bcrypt.

You might test behavior:

```text
Correct password
→ authentication succeeds
```

Wrong password:

```text
→ authentication fails
```

But you generally don't want tests depending on the exact bcrypt hash string because implementation details can change.

Test the behavior.

---

# 55. Testing Transactions

Suppose creating an order requires:

```text
Create order
 ↓
Create order items
 ↓
Decrease inventory
```

All of this should happen in a transaction.

Test failure in the middle.

For example:

```text
Create order ✓
Create items ✓
Decrease inventory ✗
```

Expected:

```text
Everything rolled back
```

If your test doesn't verify this, you might accidentally introduce partial data.

---

# 56. Testing Race Conditions

Some bugs only appear under concurrency.

Imagine:

```text
Inventory = 1
```

Two users buy at the same time.

Both requests read:

```text
inventory = 1
```

Both attempt to purchase.

Now:

```text
inventory = -1
```

or you've sold more than you had.

A normal single-request test won't catch this.

For concurrency-sensitive code, you may need tests that execute operations concurrently and verify your database constraints/transactions/locking strategy.

---

# 57. Testing Idempotency

From previous chapters:

```http
POST /payments
Idempotency-Key: abc123
```

Test:

```text
Request 1
→ payment created
```

Then:

```text
Request 2
same idempotency key
→ no duplicate payment
```

This is a perfect example of an important real-world behavior that deserves an automated test.

---

# 58. Testing Pagination

Suppose:

```http
GET /posts?page=2&limit=10
```

Test:

```text
empty database
1 record
10 records
11 records
large dataset
invalid page
invalid limit
maximum limit
```

Also verify:

```text
No duplicate records across pages
No missing records
Correct total/count metadata
```

Pagination bugs are often subtle.

---

# 59. Testing Caching

From Chapter 13:

Suppose:

```text
GET /products/123
```

First request:

```text
Cache miss
 ↓
Database
 ↓
Cache
```

Second request:

```text
Cache hit
 ↓
No database query
```

Your tests can verify this behavior.

Then:

```text
Product updated
 ↓
Cache invalidated
```

Next request:

```text
Cache miss
 ↓
Fresh database value
```

This is much more valuable than only testing that Redis is "connected."

---

# 60. Testing Background Jobs

Suppose:

```text
POST /welcome-email
```

returns immediately:

```text
202 Accepted
```

and creates a queue job.

Test:

```text
API request
 ↓
job created
```

Then separately:

```text
worker
 ↓
job processed
 ↓
email sent
```

Also test:

```text
email provider fails
 ↓
job retries
```

and eventually:

```text
max retries reached
 ↓
dead-letter/failed state
```

if your system uses one.

---

# 61. Testing File Uploads

You should test:

```text
Valid file
Invalid extension
Wrong MIME type
Too large
Empty file
Missing file
Too many files
Unauthorized user
Private file access
```

And if your system scans files:

```text
Scan passes
Scan fails
```

---

# 62. Testing WebSockets

Test:

```text
Connect
Disconnect
Join room
Leave room
Send message
Receive message
Unauthorized room access
```

Also think about:

```text
Connection dropped
Reconnect
Duplicate messages
```

depending on your application.

---

# 63. Test Your Error Responses

Your API should have predictable behavior.

For example:

```text
GET /users/does-not-exist
```

should consistently return:

```http
404
```

with your expected error structure.

Don't only test successful responses.

A mature test suite spends significant effort on failure paths.

---

# 64. Testing Environment

Never casually run destructive tests against your production database.

Have a separate:

```text
development
test
staging
production
```

environment where appropriate.

Your tests should use:

```text
TEST_DATABASE_URL
```

rather than your production connection string.

This sounds obvious.

People have still accidentally deleted production data.

Don't be that person.

---

# 65. Test Secrets

Don't hardcode real secrets in tests.

Bad:

```text
AWS_SECRET_ACCESS_KEY=real-production-secret
```

Tests should use:

```text
fake/test credentials
```

and mocked services where appropriate.

---

# 66. Flaky Tests

One of the most frustrating things:

```text
Run 1 → pass
Run 2 → fail
Run 3 → pass
```

That's a:

> **Flaky test**

The test isn't deterministic.

Common causes:

```text
Timing
Race conditions
Shared state
Real network calls
Random data
Timezone differences
Uncontrolled clocks
Tests depending on execution order
```

Flaky tests are dangerous because developers eventually stop trusting the test suite.

---

# 67. Fix Flaky Tests

Don't simply:

```text
retry the test 5 times
```

and call it fixed.

Find the reason.

For example:

```text
Bad:
sleep(2000)
```

Maybe the test should instead wait for a specific condition.

Or:

```text
Bad:
depends on previous test data
```

Fix the test isolation.

---

# 68. Parallel Tests

Large test suites may run tests in parallel to become faster.

That's good.

But it can expose hidden dependencies.

For example:

```text
Test A modifies shared database row
Test B modifies same row
```

Running sequentially:

```text
works
```

Running in parallel:

```text
fails
```

This can reveal poor test isolation.

---

# 69. Testing in CI

Now connect this to Chapter 23 later.

You push code:

```text
git push
   ↓
GitHub Actions
   ↓
Install dependencies
   ↓
Run tests
   ↓
Tests pass?
```

If they fail:

```text
❌ Pull request blocked
```

If they pass:

```text
✅ Continue pipeline
```

This is how automated testing becomes part of software delivery.

---

# 70. Tests Are Part of the Development Workflow

A healthy workflow can look like:

```text
Write code
   ↓
Write/update tests
   ↓
Run tests
   ↓
Fix
   ↓
Commit
   ↓
CI
   ↓
Deploy
```

Testing isn't something you do once at the end.

It becomes part of development.

---

# 71. Unit Test Example

Let's make a small service.

```js
export function calculateTotal(price, quantity) {
  if (price < 0 || quantity < 0) {
    throw new Error("Invalid values");
  }

  return price * quantity;
}
```

Tests:

```js
describe("calculateTotal", () => {
  test("calculates total", () => {
    expect(calculateTotal(100, 3)).toBe(300);
  });

  test("returns zero when quantity is zero", () => {
    expect(calculateTotal(100, 0)).toBe(0);
  });

  test("rejects negative price", () => {
    expect(() => {
      calculateTotal(-100, 2);
    }).toThrow();
  });
});
```

Notice:

```text
Happy path
Boundary
Failure
```

That's already better than testing only:

```text
100 × 3
```

---

# 72. API Test Example

Imagine:

```http
GET /api/users/123
```

Test:

```js
const response = await request(app)
  .get("/api/users/123");

expect(response.status).toBe(200);
expect(response.body.id).toBe("123");
```

Now test missing user:

```js
const response = await request(app)
  .get("/api/users/999");

expect(response.status).toBe(404);
```

Now you're testing actual HTTP behavior.

---

# 73. Authentication API Test

For:

```http
POST /api/login
```

test:

```js
const response = await request(app)
  .post("/api/login")
  .send({
    email: "test@example.com",
    password: "correct-password"
  });

expect(response.status).toBe(200);
expect(response.body.token).toBeDefined();
```

Then:

```text
Wrong password
→ 401
```

and:

```text
Missing password
→ 400
```

---

# 74. Testing Protected Routes

Suppose:

```http
GET /api/profile
```

requires authentication.

Test without token:

```js
const response = await request(app)
  .get("/api/profile");

expect(response.status).toBe(401);
```

Then with a valid token:

```text
→ 200
```

Then with another user's credentials:

```text
→ appropriate authorization response
```

Security should be tested like any other feature.

---

# 75. A Practical Test Strategy

For your backend projects, you don't need to write 10,000 tests.

Start with important behavior.

For a Todo API:

```text
Unit:
- validation
- business rules

Integration:
- database operations

API:
- create todo
- get todos
- update todo
- delete todo
- authentication
- authorization

Failure:
- invalid input
- missing todo
- unauthorized request
- database failure
```

That's already a meaningful test suite.

---

# 76. What Not to Test

Don't waste time testing framework behavior.

For example, if Express guarantees that:

```text
res.json()
```

sends JSON, you don't need a test proving Express itself works.

Test your code.

Not every dependency.

---

# 77. Don't Test the Same Thing at Every Level

Suppose you test:

```text
calculateTotal()
```

at unit level.

Then your E2E test also indirectly tests:

```text
calculateTotal()
```

That's okay.

But don't create five nearly identical tests just because you feel you need more coverage.

Different test levels should give different confidence.

---

# 78. The Real Goal

Here's something worth remembering:

> **The goal of testing isn't maximum number of tests.**

The goal is:

> **Maximum useful confidence for reasonable effort.**

You want to be able to say:

```text
I changed authentication.

Tests passed.

Database integration tests passed.

Protected-route tests passed.

CI passed.

I'm reasonably confident I didn't break the application.
```

That's the value.

---

# 79. Common Mistakes

## Mistake 1 — Only testing happy paths

Real systems fail.

Test failures too.

---

## Mistake 2 — Mocking everything

Your tests can pass while your real integrations are broken.

---

## Mistake 3 — No test isolation

One test modifies state and breaks another.

---

## Mistake 4 — Chasing 100% coverage

Coverage is a signal, not the goal.

---

## Mistake 5 — Testing implementation details

Refactoring shouldn't require rewriting behavior tests.

---

## Mistake 6 — Using production data

Never use production databases casually for tests.

---

## Mistake 7 — Making real external API calls

Tests become slow, expensive, unreliable, and potentially dangerous.

---

## Mistake 8 — Ignoring concurrency

Some bugs only appear when multiple operations happen at the same time.

---

## Mistake 9 — Ignoring flaky tests

A test suite people don't trust is almost useless.

---

## Mistake 10 — Testing only functions

Backend behavior also lives in:

```text
HTTP
Database
Authentication
Queues
External services
```

Test the important boundaries.

---

# 80. Interview Questions

## 1. What is unit testing?

Testing a small isolated piece of code, usually without real external dependencies.

---

## 2. What is integration testing?

Testing how multiple components work together, such as an application and database.

---

## 3. What is E2E testing?

Testing a complete workflow across multiple parts of the system from a user/business perspective.

---

## 4. What is the testing pyramid?

A testing strategy that generally favors many fast unit tests, fewer integration tests, and a smaller number of expensive E2E tests.

---

## 5. What is mocking?

Replacing a real dependency with a controlled test double.

---

## 6. Why shouldn't you mock everything?

Because mocks can hide problems in real integrations. You still need integration tests for important boundaries.

---

## 7. What is test isolation?

Ensuring one test doesn't depend on or corrupt the state of another test.

---

## 8. What is a flaky test?

A test that sometimes passes and sometimes fails without a relevant code change.

---

## 9. What causes flaky tests?

Common causes include:

```text
Timing
Race conditions
Shared state
Real network calls
Randomness
Timezone differences
Test ordering
```

---

## 10. What is code coverage?

A measurement of how much code is executed by tests.

It doesn't measure how correct or useful the tests are.

---

## 11. Is 100% test coverage necessary?

Not necessarily. Important behavior and risk should guide testing effort. High coverage can still contain poor tests.

---

## 12. What should you test in an API?

Test:

```text
Success
Validation
Authentication
Authorization
Error responses
Edge cases
Database behavior
Important business rules
```

---

## 13. How would you test an API that depends on an external payment provider?

Use mocks/test doubles for most automated tests and separately test the real integration in an appropriate environment.

Test:

```text
Success
Failure
Timeout
Rate limit
Malformed response
Retries
Idempotency
```

---

## 14. How do you test database code?

Use unit tests with mocks for isolated business logic and integration tests with a real test database or isolated database environment for actual database behavior.

---

## 15. How would you test a race condition?

Create concurrent operations and verify the resulting state. Use database constraints, transactions, locks, or other synchronization mechanisms as appropriate.

---

# 81. Interview Scenario

> You changed your authentication middleware. What would you test?

Don't just test:

```text
Login works
```

Think:

```text
Valid token
Invalid token
Expired token
Missing token
Malformed token
Protected route
Public route
Different user
Different role
```

Then:

```text
Existing API endpoints
```

because authentication middleware may affect all of them.

---

# 82. Interview Scenario

> Your test suite has 95% code coverage, but production still has lots of bugs. Why?

Coverage only tells you which code executed.

It doesn't tell you whether:

```text
correct behavior was tested
edge cases were tested
failure scenarios were tested
real integrations work
concurrency works
authorization works
```

You can have high coverage with poor tests.

---

# 83. Interview Scenario

> All your tests pass locally, but CI randomly fails.

Think:

```text
Environment differences
Timezone
Database state
Race condition
Parallel execution
Environment variables
Test ordering
Network dependency
Randomness
Timing
```

If the failure is inconsistent:

> suspect flakiness.

Don't simply rerun forever.

Find the underlying cause.

---

# 84. Interview Scenario

> Your database integration tests are slow. What would you do?

First understand why.

Possibilities:

```text
Too much setup
Database recreated unnecessarily
Too many sequential tests
Expensive migrations
Unnecessary integration coverage
```

Possible improvements:

```text
Better test isolation
Transactions
Database reuse
Parallelization where safe
Test containers
Fewer unnecessary integration tests
```

Don't blindly mock the database just because tests are slow.

---

# 85. Mini Project — Add Tests to Your Todo API

Take the Todo API from the earlier chapters.

You should now have something like:

```text
projects/
└── todo-api/
```

Add:

```text
tests/
├── unit/
├── integration/
└── api/
```

A possible structure:

```text
todo-api/
├── src/
│   ├── controllers/
│   ├── services/
│   ├── repositories/
│   ├── middleware/
│   └── routes/
│
├── tests/
│   ├── unit/
│   │   └── todo.service.test.ts
│   │
│   ├── integration/
│   │   └── todo.repository.test.ts
│   │
│   └── api/
│       ├── auth.test.ts
│       └── todos.test.ts
│
├── package.json
└── ...
```

---

# 86. Test Todo Business Logic

Test:

```text
Create todo
Update todo
Complete todo
Delete todo
```

And edge cases:

```text
Empty title
Very long title
Invalid ID
Already completed
```

depending on your business rules.

---

# 87. Test Database Integration

Verify:

```text
Create todo
 ↓
PostgreSQL
 ↓
Read todo
```

Then:

```text
Update todo
 ↓
Database
 ↓
Read updated todo
```

Then:

```text
Delete todo
 ↓
Database
 ↓
Todo no longer exists
```

You're testing the actual database interaction.

---

# 88. Test API Endpoints

Test:

```http
POST /api/todos
GET /api/todos
GET /api/todos/:id
PATCH /api/todos/:id
DELETE /api/todos/:id
```

For each endpoint test:

```text
Success
Invalid input
Unauthorized
Not found
```

where applicable.

---

# 89. Test Authentication

Test:

```text
Register
Login
Wrong password
Missing credentials
Protected endpoint without token
Protected endpoint with valid token
Invalid token
```

---

# 90. Test Authorization

Create:

```text
User A
User B
```

User A creates:

```text
Todo 123
```

User B attempts:

```text
PATCH /todos/123
```

Expected:

```text
Forbidden
```

or whatever your API contract defines.

This is a very important security test.

---

# 91. Break Your Application on Purpose

This is one of the best ways to learn.

After writing tests:

```text
Break authentication
 ↓
Run tests
```

Does a test fail?

Then:

```text
Break authorization
```

Run tests.

Then:

```text
Break database query
```

Run tests.

Then:

```text
Return wrong HTTP status
```

Run tests.

You want your tests to actually catch mistakes.

---

# 92. Your Test Checklist

Before considering the project reasonably tested:

```text
□ Happy path tested
□ Invalid input tested
□ Authentication tested
□ Authorization tested
□ Not-found behavior tested
□ Database integration tested
□ Important business rules tested
□ External dependencies mocked
□ Important failures tested
□ Test data isolated
□ Tests deterministic
□ No production secrets
□ No production database
□ Tests run in CI
```

---

# 93. The Mental Model

When you're asked:

> "How would you test this backend?"

Don't immediately say:

> "I'll write Jest tests."

Instead think:

```text
What can go wrong?
        ↓
What is the risk?
        ↓
What behavior matters?
        ↓
What should be isolated?
        ↓
What needs a real integration?
        ↓
What can be mocked?
        ↓
What happens when dependencies fail?
        ↓
What happens concurrently?
        ↓
How do I know the test itself is reliable?
```

Then choose:

```text
Unit
Integration
API
E2E
```

based on the problem.

---

# 94. The Bigger Lesson

We've now reached an important point in this series.

A beginner often thinks:

```text
Backend
=
Routes + Database
```

Then they learn more:

```text
Backend
=
Routes
+ Database
+ Authentication
+ Caching
+ Queues
+ WebSockets
+ Storage
+ Logging
+ Monitoring
+ Testing
```

But there's another way to look at it.

A real backend is a system that needs to be:

```text
Correct
Secure
Observable
Testable
Reliable
Maintainable
```

Testing is one part of that.

And the best test suite isn't the one with the most tests.

It's the one that gives you enough confidence to change the code without being terrified that something unrelated will break.

That's the real reason we write tests.

---

# What You Should Know After Chapter 18

You should now be able to explain:

```text
✓ Why backend testing matters
✓ Unit tests
✓ Integration tests
✓ API tests
✓ E2E tests
✓ Testing pyramid
✓ Arrange / Act / Assert
✓ Mocks
✓ Stubs
✓ Fakes
✓ Spies
✓ Test isolation
✓ Fixtures
✓ Test factories
✓ Edge cases
✓ Boundary testing
✓ Code coverage
✓ Flaky tests
✓ Testing databases
✓ Testing external APIs
✓ Testing authentication
✓ Testing authorization
✓ Testing queues
✓ Testing WebSockets
✓ Testing file uploads
✓ Testing caching
✓ Testing retries
✓ Testing timeouts
✓ Testing idempotency
✓ Testing transactions
✓ Testing concurrency
✓ CI testing
```

And most importantly, you should be able to answer:

> **"How would you test a real backend feature?"**

without thinking only about:

```text
expect(result).toBe(...)
```

Instead:

```text
What should happen?
What shouldn't happen?
What happens with bad input?
What happens without authentication?
What happens with the wrong user?
What happens when the database fails?
What happens when an external service times out?
What happens when two requests happen simultaneously?
How do I test it without making the test suite fragile?
```

That's the testing mindset we're trying to build.