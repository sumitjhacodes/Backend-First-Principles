# Chapter 15 — WebSockets & Real-Time Systems

So far in this series, most of our backend communication has looked like this:

```text
Client
   ↓
HTTP Request
   ↓
Server
   ↓
HTTP Response
```

The client asks for something.

The server responds.

Done.

For many applications, this is perfectly fine.

But now imagine we're building a chat application.

You send:

```text
"Hey, are you free?"
```

You expect the other person to see it immediately.

You don't want their browser doing this:

```text
"Any new message?"

3 seconds later:

"Any new message?"

3 seconds later:

"Any new message?"
```

That's called **polling**.

It works, but it's wasteful.

What if the server could simply keep a connection open and say:

> "Whenever something happens, I'll tell you."

That's the basic idea behind **WebSockets**.

---

# 1. What Is Real-Time Communication?

Real-time communication means:

> **The server and client can exchange information with very little delay after an event occurs.**

Examples:

```text
Chat messages
Live notifications
Typing indicators
Online/offline status
Live dashboards
Multiplayer games
Stock/crypto price updates
Delivery tracking
Collaborative editing
```

For example:

```text
Rahul sends message
        ↓
      Server
        ↓
     Aman sees it
```

without Aman manually refreshing the page.

---

# 2. HTTP vs WebSocket

Traditional HTTP usually looks like:

```text
Client
  ↓
Request
  ↓
Server
  ↓
Response
```

The client starts the conversation.

With a WebSocket connection:

```text
Client
  ↕
Server
```

The connection stays open.

Either side can send messages.

That's the important difference.

---

# 3. Think of HTTP Like Sending Letters

Imagine you want to talk to someone using letters.

You:

```text
Write letter
   ↓
Send it
   ↓
They receive it
   ↓
They reply
```

Every interaction is separate.

That's roughly how normal request/response communication feels.

Now imagine a phone call:

```text
You
 ↕
Other person
```

The connection stays open.

Either person can speak whenever they need to.

That's a useful mental model for WebSockets.

It's not technically a phone call, but the analogy helps understand the communication pattern.

---

# 4. What Is a WebSocket?

A WebSocket is a protocol that provides **persistent, two-way communication** between a client and server over a connection.

The important words are:

```text
Persistent
Two-way
Connection
```

Persistent:

```text
Connection stays open
```

Two-way:

```text
Client → Server
Server → Client
```

So instead of:

```text
Request → Response
Request → Response
Request → Response
```

we can have:

```text
Client ←→ Server
       ↑
   open connection
```

---

# 5. How Does a WebSocket Connection Start?

This part confuses beginners.

WebSockets don't simply appear from nowhere.

They start with an HTTP request called a:

> **WebSocket handshake**

Conceptually:

```text
Client
   ↓
HTTP request asking to upgrade
   ↓
Server
   ↓
HTTP response accepting upgrade
   ↓
WebSocket connection
```

The connection starts as HTTP and then gets upgraded to WebSocket.

---

# 6. The Upgrade

The client essentially tells the server:

> "I want to upgrade this HTTP connection to a WebSocket connection."

The request contains headers related to the upgrade.

Conceptually:

```http
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
```

If the server accepts:

```text
HTTP
 ↓
Upgrade
 ↓
WebSocket
```

After that, communication follows the WebSocket protocol.

---

# 7. `ws://` and `wss://`

You might see:

```text
ws://example.com
```

and:

```text
wss://example.com
```

`ws://` is the non-TLS WebSocket scheme.

`wss://` is WebSocket over TLS.

For production applications, you normally want:

```text
wss://
```

because the connection should be encrypted.

Just like:

```text
HTTP
```

vs:

```text
HTTPS
```

you can think of:

```text
WS
```

vs:

```text
WSS
```

---

# 8. The Simplest WebSocket Example

On the browser side:

```js
const socket = new WebSocket(
  "ws://localhost:3000"
);
```

Now the browser tries to establish a WebSocket connection.

You can listen for events:

```js
socket.onopen = () => {
  console.log("Connected");
};
```

When the connection is closed:

```js
socket.onclose = () => {
  console.log("Disconnected");
};
```

And when a message arrives:

```js
socket.onmessage = (event) => {
  console.log(event.data);
};
```

You can send a message:

```js
socket.send("Hello server");
```

That's the basic client-side API.

---

# 9. The Communication Model

Once connected:

```text
Client
  │
  ├──── message ────→ Server
  │
  │
  ←──── message ─────┤
  │
  ├──── message ────→ Server
  │
  ←──── message ─────┤
```

There doesn't need to be a new HTTP request for every message.

That's one of the main benefits.

---

# 10. Why Not Just Use HTTP?

Good question.

HTTP is excellent for many things.

For example:

```text
GET /users/42
POST /orders
PATCH /profile
DELETE /comments/123
```

You don't need WebSockets for everything.

But imagine:

```text
Live chat
```

The server needs to push new messages to connected users.

With regular HTTP, you need something like polling.

With WebSockets:

```text
Server
  ↓
"New message!"
```

The server can send it immediately over the open connection.

---

# 11. Polling

Polling means:

> The client repeatedly asks the server whether something changed.

For example:

```text
Client → GET /messages
Server → No new messages

5 seconds later

Client → GET /messages
Server → No new messages

5 seconds later

Client → GET /messages
Server → New message!
```

This works.

But most requests might return:

```text
"No new messages."
```

That's unnecessary traffic.

---

# 12. Long Polling

There's another technique called:

> **Long polling**

Instead of responding immediately:

```text
Client
 ↓
Request
 ↓
Server waits
```

The server keeps the request open until something happens or a timeout occurs.

Then:

```text
New message
 ↓
Response
```

The client immediately opens another request.

It's more efficient than aggressive polling but still has request/response overhead.

WebSockets provide a more natural persistent bidirectional connection.

---

# 13. Server-Sent Events

You may also hear:

> **SSE — Server-Sent Events**

SSE allows the server to push events to the client over a long-lived HTTP connection.

The important difference:

```text
SSE:
Server → Client
```

while WebSockets provide:

```text
Client ↔ Server
```

So:

### Use SSE when:

The server mostly needs to push updates.

Examples:

```text
Live notifications
Progress updates
Streaming server events
Live dashboards
```

### Use WebSockets when:

Both sides need to communicate continuously.

Examples:

```text
Chat
Gaming
Collaborative applications
Interactive real-time systems
```

It's not an absolute rule, but it's a useful starting point.

---

# 14. Polling vs Long Polling vs SSE vs WebSockets

| Technology   | Communication   | Connection          | Good For                      |
| ------------ | --------------- | ------------------- | ----------------------------- |
| Polling      | Client → Server | Repeated requests   | Simple periodic updates       |
| Long polling | Client → Server | Long-lived requests | Basic real-time behavior      |
| SSE          | Server → Client | Long-lived HTTP     | Server-driven updates         |
| WebSocket    | Client ↔ Server | Persistent          | Interactive real-time systems |

Don't memorize this table.

Understand the communication direction.

---

# 15. WebSocket Messages

A WebSocket connection can carry messages.

For example:

```text
"Hello"
```

But real applications usually send structured data.

JSON is common:

```json
{
  "type": "message",
  "text": "Hello"
}
```

Or:

```json
{
  "type": "typing",
  "userId": "123"
}
```

The `type` field is useful because the receiver can understand what the message means.

---

# 16. Message Types

Imagine our chat application.

We might have:

```text
message
typing
user_online
user_offline
message_read
```

So a WebSocket message might look like:

```json
{
  "type": "typing",
  "conversationId": "abc123"
}
```

The server can then handle it differently from:

```json
{
  "type": "message",
  "text": "Hello"
}
```

This is a simple form of a **message protocol**.

---

# 17. Don't Trust WebSocket Messages

A WebSocket connection doesn't mean the client is trustworthy.

A malicious client can send:

```json
{
  "type": "message",
  "text": "<anything>"
}
```

So you still need:

```text
Authentication
Authorization
Validation
Rate limiting
Input sanitization
```

Just like normal HTTP APIs.

A WebSocket is another entry point into your backend.

---

# 18. Authentication

Suppose a user connects:

```text
WebSocket
   ↓
Server
```

How does the server know who they are?

You need an authentication strategy.

Depending on your architecture, authentication information can be provided during the connection setup or through a dedicated authentication message/mechanism.

For example, conceptually:

```text
Connect
  ↓
Authenticate
  ↓
User identified
  ↓
Connection accepted
```

After that, your server can associate:

```text
connection
     ↓
userId
```

For example:

```text
socket A → user 123
socket B → user 456
```

---

# 19. Authentication vs Authorization

Remember Chapter 10?

Authentication asks:

> **Who are you?**

Authorization asks:

> **What are you allowed to do?**

For a chat:

```text
Authentication:
This is user 123.
```

Authorization:

```text
Is user 123 allowed to send messages
to conversation ABC?
```

Don't confuse the two.

---

# 20. Connections

Every connected client creates a connection.

Imagine:

```text
Server

Connection 1 → User A
Connection 2 → User B
Connection 3 → User C
Connection 4 → User D
```

Your server needs to keep track of these connections.

For example:

```text
userId
   ↓
socket connection
```

Conceptually:

```js
connections.set(userId, socket);
```

But real applications become more complicated because a user can have multiple devices/tabs.

---

# 21. One User Can Have Multiple Connections

Suppose Rahul opens your application on:

```text
Laptop
Phone
Tablet
```

You might have:

```text
user:123
 ├── socket A
 ├── socket B
 └── socket C
```

So don't always assume:

```text
one user = one WebSocket
```

A better model can be:

```text
user
 ↓
multiple connections
```

This matters when broadcasting notifications.

---

# 22. Rooms

Now imagine a group chat:

```text
Backend Engineering Group
```

Users:

```text
Rahul
Aman
Priya
Neha
```

You don't want every message sent to every connected user.

You need a group.

That's where the concept of a:

> **Room**

is useful.

Conceptually:

```text
Room: backend-group

 ├── Rahul
 ├── Aman
 ├── Priya
 └── Neha
```

A message sent to that room goes to its members.

---

# 23. Socket.IO

You will often see:

> **Socket.IO**

when working with Node.js real-time applications.

Important:

> Socket.IO is **not the WebSocket protocol itself**.

It's a real-time communication library that can use WebSocket when possible and provides additional features and abstractions.

It gives you things like:

```text
Rooms
Events
Broadcasting
Reconnection
Namespaces
Acknowledgements
```

This can make application development easier.

---

# 24. WebSocket vs Socket.IO

Think of it like:

```text
WebSocket
↓
Protocol / communication technology
```

while:

```text
Socket.IO
↓
Library built for real-time application development
```

Socket.IO provides abstractions on top of the underlying transport mechanisms.

So don't say in an interview:

> "Socket.IO and WebSocket are the same thing."

They're not.

---

# 25. Creating a Socket.IO Server

For a Node.js application:

```bash
npm install socket.io
```

Then conceptually:

```ts
io.on("connection", (socket) => {
  console.log("User connected");

  socket.on("message", (message) => {
    console.log(message);
  });
});
```

The important idea isn't memorizing the API.

It's:

```text
New connection
     ↓
Register event handlers
     ↓
Receive messages
     ↓
Send messages
```

---

# 26. Events

Socket.IO commonly uses named events.

For example:

```text
message
typing
join_room
leave_room
notification
```

Client:

```js
socket.emit("message", {
  text: "Hello"
});
```

Server:

```js
socket.on("message", (data) => {
  console.log(data);
});
```

The event name tells the receiver what happened.

---

# 27. Broadcasting

Suppose Rahul sends:

```text
"Hello everyone"
```

The server receives it.

Now it needs to send the message to other users.

That's:

> **Broadcasting**

Conceptually:

```text
Rahul
  ↓
Server
  ↓
Aman
Priya
Neha
```

Depending on the requirement, you can broadcast to:

```text
Everyone
Everyone except sender
A particular room
A particular user
```

---

# 28. Chat Example

Imagine:

```text
Rahul → "Hey!"
```

Server receives:

```json
{
  "type": "message",
  "text": "Hey!"
}
```

Server decides:

```text
Conversation = room-123
```

Then broadcasts:

```text
room-123
 ├── Aman ← "Hey!"
 ├── Priya ← "Hey!"
 └── Neha ← "Hey!"
```

Rahul might receive a server acknowledgement separately.

---

# 29. Typing Indicators

Now something fun.

When Rahul starts typing:

```text
Rahul types...
```

the client can send:

```json
{
  "type": "typing",
  "conversationId": "123"
}
```

Server broadcasts:

```text
Aman
 ↓
"Rahul is typing..."
```

When Rahul stops:

```json
{
  "type": "stopped_typing",
  "conversationId": "123"
}
```

Now:

```text
"Rahul is typing..."
```

disappears.

This is a perfect WebSocket use case.

---

# 30. Online / Offline Status

When a user connects:

```text
User connects
   ↓
Server
   ↓
Mark online
```

When the connection closes:

```text
Connection closes
   ↓
Server
   ↓
Mark offline
```

But be careful.

Remember:

```text
one user
 ↓
multiple connections
```

If Rahul closes his phone connection but is still connected from his laptop, he shouldn't become:

```text
offline
```

So presence systems require a little more thought.

---

# 31. Heartbeats

What if the network disappears but the server doesn't immediately realize it?

You need a way to determine whether a connection is still alive.

This is where:

> **Ping/Pong**

or heartbeat mechanisms are used.

Conceptually:

```text
Server → ping
Client → pong
```

If the client stops responding:

```text
No pong
   ↓
Connection considered dead
```

Then the server can clean up the connection.

Socket.IO and WebSocket implementations have their own heartbeat mechanisms.

---

# 32. Reconnection

Networks are unreliable.

A user can:

```text
Enter elevator
Lose Wi-Fi
Switch from Wi-Fi to mobile data
Close laptop
Put phone to sleep
```

The connection can disappear.

A real-time application should handle this.

Conceptually:

```text
Connected
   ↓
Network lost
   ↓
Disconnected
   ↓
Reconnect
   ↓
Connected again
```

Socket.IO provides reconnection support.

But reconnecting creates another problem:

> **What happened while the client was disconnected?**

---

# 33. The Missed Message Problem

Imagine:

```text
Rahul connected
```

Then:

```text
Network lost
```

While Rahul is offline:

```text
Aman sends:
"Are you coming?"
```

Rahul reconnects.

If your system simply starts a new WebSocket connection, Rahul might never receive that message.

So real chat systems usually need a durable message history.

For example:

```text
WebSocket
   ↓
real-time delivery

PostgreSQL
   ↓
message history
```

When reconnecting:

```text
Reconnect
   ↓
Find messages missed since last message
   ↓
Send missed messages
```

This is a very important architecture concept.

---

# 34. WebSocket Is Not Your Database

Don't make this mistake:

```text
WebSocket
=
message storage
```

WebSocket handles communication.

Your database should usually store important persistent information.

For chat:

```text
PostgreSQL
 ↓
Messages
Users
Conversations
Read status
```

WebSocket:

```text
Real-time delivery
```

So:

```text
Database
+
WebSocket
```

work together.

---

# 35. A Better Chat Architecture

Now we can build something more realistic:

```text
                    Clients
                 /     |     \
                /      |      \
               ↓       ↓       ↓
          WebSocket WebSocket WebSocket
               \      |      /
                \     |     /
                 ↓    ↓    ↓
                Real-time
                  Server
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
      PostgreSQL            Redis
      messages             pub/sub
```

Why Redis?

We'll get to that shortly.

---

# 36. The Single-Server Problem

Suppose everything runs on one server:

```text
Server A
 ├── User Rahul
 ├── User Aman
 └── User Priya
```

Rahul sends a message.

The server knows all three connections.

Easy.

But now our application grows.

We add:

```text
Server A
Server B
Server C
```

A load balancer distributes users:

```text
                  Load Balancer
                 /      |      \
                ↓       ↓       ↓
            Server A Server B Server C
```

Now:

```text
Rahul → Server A
Aman  → Server C
```

Rahul sends a message.

Server A knows Rahul's connection.

But does Server A know about Aman's connection?

Maybe not.

That's the scaling problem.

---

# 37. The Multi-Server Problem

Imagine:

```text
Rahul
  ↓
Server A
```

and:

```text
Aman
  ↓
Server B
```

Rahul sends:

```text
"Hello Aman"
```

Server A needs to tell Server B:

> "Send this message to Aman."

How do the servers communicate?

One common solution is:

> **Redis Pub/Sub**

---

# 38. Redis Pub/Sub

Redis can provide a publish/subscribe mechanism.

One server can:

```text
PUBLISH
```

an event.

Other servers can:

```text
SUBSCRIBE
```

to it.

Conceptually:

```text
Server A
   ↓
Redis Pub/Sub
   ↓
Server B
Server C
```

So:

```text
Rahul
 ↓
Server A
 ↓
Redis
 ↓
Server B
 ↓
Aman
```

Now different WebSocket servers can share events.

---

# 39. Important: Pub/Sub Is Not Message History

This distinction matters.

Redis Pub/Sub is useful for distributing live events.

But a Pub/Sub message isn't automatically a durable chat history.

If:

```text
Server B
```

is disconnected when an event is published, it may miss that event.

So don't assume:

```text
Pub/Sub
=
durable queue
```

They're different tools with different guarantees.

If you need durable background processing, that's where the queue concepts from Chapter 14 become useful.

---

# 40. Sticky Sessions

Another concept you may hear:

> **Sticky sessions**

The load balancer tries to keep a client connected to the same server.

For example:

```text
Rahul
 ↓
Load Balancer
 ↓
Server A
```

Future connections/traffic from Rahul may continue going to Server A.

This can simplify some architectures.

But sticky sessions don't magically solve everything.

You still need to think about:

```text
Server failure
Scaling
State sharing
Reconnection
Broadcasting
```

For large real-time systems, shared infrastructure is often still needed.

---

# 41. WebSocket Scaling Architecture

A more scalable architecture might look like:

```text
                       Clients
                          │
                          ↓
                   Load Balancer
                  /      |      \
                 ↓       ↓       ↓
              WS-1     WS-2     WS-3
                │        │        │
                └────────┼────────┘
                         ↓
                   Redis Pub/Sub
                         │
                         ↓
                    PostgreSQL
```

Now:

```text
WebSocket servers
→ maintain connections

Redis
→ distributes real-time events

PostgreSQL
→ stores durable application data
```

Each component has a different responsibility.

---

# 42. Don't Put All State in Server Memory

Suppose:

```text
Server A memory:
user 123 → connected
```

Then Server A crashes.

That state disappears.

For data that needs to survive server restarts or be shared across servers, don't rely only on local memory.

For example:

```text
Important persistent data
→ Database

Shared ephemeral state
→ Redis

Connection itself
→ WebSocket server
```

This separation becomes increasingly important as you scale.

---

# 43. Message Delivery Is a System Design Problem

Suppose Rahul sends:

```text
"Hello"
```

You need to decide:

```text
Was the message saved?
Was it delivered?
Was it read?
```

These are different things.

For example:

```text
Sent
 ↓
Stored
 ↓
Delivered
 ↓
Read
```

A good chat system may represent these separately.

Don't assume:

> "WebSocket `send()` succeeded, therefore the message is permanently stored."

It isn't.

---

# 44. Acknowledgements

Sometimes the sender needs confirmation.

For example:

```text
Rahul
 ↓
send message
 ↓
Server
 ↓
acknowledgement
```

The server can say:

```json
{
  "messageId": "msg_123",
  "status": "accepted"
}
```

Now the client knows the server accepted the message.

This can help build reliable user experiences.

---

# 45. Message IDs

Every important message should usually have an identifier.

For example:

```json
{
  "messageId": "msg_123",
  "conversationId": "conv_42",
  "senderId": "user_10",
  "text": "Hello"
}
```

Why?

Because clients and servers need to reason about:

```text
Duplicates
Retries
Ordering
Acknowledgements
Synchronization
```

A message ID becomes extremely useful.

---

# 46. Message Ordering

Imagine Rahul sends:

```text
Message A
Message B
Message C
```

The receiver should ideally see:

```text
A
B
C
```

But distributed systems can make ordering tricky.

For example:

```text
A ───────→
B ─→
C ───────────→
```

Network timing can differ.

If message order matters, your system needs an explicit strategy.

For a chat system, you might use:

```text
server-generated sequence numbers
timestamps
database ordering
conversation versioning
```

depending on the requirements.

Don't blindly trust client timestamps.

---

# 47. Race Conditions in Real-Time Systems

Suppose two people edit the same resource:

```text
User A
   ↓
Update title = "Hello"

User B
   ↓
Update title = "World"
```

Which wins?

Real-time systems can expose race conditions quickly.

You may need:

```text
Transactions
Optimistic locking
Version numbers
Conflict resolution
Ordering rules
```

WebSockets don't solve concurrency problems.

They just make communication faster.

---

# 48. Rate Limiting WebSocket Messages

HTTP APIs are often rate-limited.

WebSockets need protection too.

Imagine a client sending:

```text
10,000 messages/sec
```

Your server could be overwhelmed.

You might limit:

```text
messages per connection
messages per user
messages per room
```

For example:

```text
100 messages / second / user
```

The exact limit depends on your application.

The principle:

> **A persistent connection doesn't mean unlimited communication.**

---

# 49. Connection Limits

Each WebSocket connection consumes resources.

Potential resources include:

```text
Memory
File descriptors
CPU
Network bandwidth
Connection tracking
```

Suppose:

```text
1 server
100,000 connections
```

That's very different from:

```text
1 server
100 connections
```

Real-time systems often need connection-capacity planning.

---

# 50. What Happens When a Server Restarts?

Suppose:

```text
Server A
 ↓
50,000 WebSocket connections
```

Server A restarts.

Those connections disappear.

Clients need to reconnect.

So your client should handle:

```text
disconnect
 ↓
reconnect
 ↓
authenticate again if necessary
 ↓
synchronize missed state
```

This is why reconnect handling is not optional in serious real-time applications.

---

# 51. Presence Is Harder Than It Looks

A simple implementation might say:

```text
connection opened
→ online

connection closed
→ offline
```

But consider:

```text
User has laptop + phone
```

Laptop disconnects:

```text
1 connection remaining
```

User is still online.

So presence might be:

```text
userId
 ↓
active connections
 ↓
count > 0
 ↓
online
```

But even that can get complicated with:

```text
Network failures
Heartbeats
Multiple servers
Mobile backgrounding
Reconnects
```

Real-time features often look simple in the UI but require careful backend state management.

---

# 52. WebSockets and Authentication Expiration

What happens if the user's authentication expires while the WebSocket remains open?

You need a strategy.

For example:

```text
Token/session expires
        ↓
Connection needs re-authentication
        ↓
Or disconnect
```

Don't assume:

```text
Connected once
=
authorized forever
```

Authentication state can change while the connection is alive.

---

# 53. Security Checklist

For WebSocket systems, think about:

```text
✓ Use WSS in production
✓ Authenticate connections
✓ Authorize actions
✓ Validate every message
✓ Rate limit users
✓ Limit message size
✓ Handle malformed messages
✓ Handle disconnects
✓ Don't trust client identity
✓ Protect private rooms
```

For example, never allow:

```json
{
  "userId": "someone-else"
}
```

from the client to automatically determine who the sender is.

The server should derive identity from authenticated connection state.

---

# 54. Message Size Limits

Imagine a client sends:

```text
500 MB
```

as one WebSocket message.

That's obviously a problem.

You should define reasonable limits.

For example:

```text
Maximum message size
Maximum number of messages
Maximum room size
```

The exact limits depend on your application.

---

# 55. WebSockets and File Uploads

Don't use WebSockets as your default solution for uploading huge files.

For large files, something like:

```text
Client
 ↓
Object Storage
```

with:

```text
Presigned URL
```

is often more appropriate.

WebSocket:

```text
"Upload completed!"
```

could then be used for real-time notification.

Again:

> Use each technology for the problem it's good at.

---

# 56. Real-Time Notifications

WebSockets aren't only for chat.

Imagine your dashboard is open.

Something happens:

```text
New job application
```

Server:

```text
Database updated
   ↓
Event
   ↓
WebSocket
   ↓
Browser
```

Browser immediately shows:

```text
🔔 New application received
```

No refresh required.

---

# 57. Live Dashboard Example

Imagine:

```text
Orders today: 1,240
```

A new order arrives.

Instead of:

```text
Browser
 ↓
GET /orders/count
```

every few seconds:

```text
Server
 ↓
WebSocket event
 ↓
Browser
```

The browser updates:

```text
Orders today: 1,241
```

This is another good real-time use case.

---

# 58. WebSockets + Database

A common architecture:

```text
Client
   ↕
WebSocket Server
   ↓
Application Logic
   ↓
PostgreSQL
```

For example:

```text
User sends message
        ↓
Authenticate
        ↓
Validate
        ↓
Store message
        ↓
Broadcast event
```

Notice the order.

For important messages, you generally don't want:

```text
Broadcast
 ↓
Database write
```

without thinking about what happens if the database write fails.

You need to define what "sent" means in your product.

---

# 59. A Better Chat Message Flow

A reasonable simple design:

```text
Client
  ↓
WebSocket message
  ↓
Server
  ↓
Authenticate
  ↓
Authorize
  ↓
Validate
  ↓
Store in PostgreSQL
  ↓
Publish event
  ↓
Other WebSocket servers
  ↓
Recipients
```

Now:

```text
PostgreSQL
→ durable message history

Redis Pub/Sub
→ distribute live event

WebSocket
→ deliver to connected clients
```

Three different pieces.

Three different jobs.

---

# 60. What If the Recipient Is Offline?

Suppose:

```text
Rahul sends message
```

but:

```text
Aman is offline
```

The server should still store:

```text
PostgreSQL
 ↓
message
```

When Aman reconnects:

```text
Reconnect
 ↓
Fetch unread/missed messages
 ↓
Display
```

This is why WebSocket systems normally need persistence outside the socket itself.

---

# 61. What If Redis Is Down?

Suppose:

```text
Server A
 ↓
Redis Pub/Sub
 ↓
Server B
```

and Redis fails.

Now cross-server broadcasting might stop.

But perhaps:

```text
Server A
```

can still communicate with users directly connected to it.

This is an example of partial failure.

Your system needs to decide:

```text
What functionality should continue?
What should degrade?
What should fail?
```

This is exactly the kind of thinking you'll need later in system design interviews.

---

# 62. WebSocket Backpressure

Suppose the server is producing messages faster than a client can consume them.

For example:

```text
Server
→ 10,000 messages/sec

Client
→ can process 1,000/sec
```

Messages can start accumulating.

You need strategies around:

```text
Buffer limits
Dropping non-critical events
Flow control
Disconnecting unhealthy clients
Reducing update frequency
```

Not every real-time event needs guaranteed delivery.

For example:

```text
typing indicator
```

can probably be dropped.

But:

```text
payment completed
```

cannot simply disappear.

This is a very important distinction:

> **Not all events have the same reliability requirements.**

---

# 63. Ephemeral vs Durable Events

Consider:

```text
"Rahul is typing..."
```

This is ephemeral.

If you miss it:

```text
No big deal.
```

But:

```text
"Payment successful"
```

is durable and important.

You need to store it.

So:

```text
Ephemeral event
→ WebSocket may be enough

Durable event
→ Persist it somewhere reliable
```

This distinction helps you design much better systems.

---

# 64. Real-Time Doesn't Mean Zero Latency

You may hear:

> "WebSockets are real-time."

Don't interpret that as:

```text
0ms latency
```

There is still:

```text
Network latency
Server processing
Database operations
Queueing
Serialization
```

"Real-time" usually means the system is designed to deliver updates quickly enough for the application's requirements.

---

# 65. Common Mistakes

## Mistake 1 — Using WebSockets for everything

HTTP is still the right choice for many APIs.

---

## Mistake 2 — Thinking Socket.IO = WebSocket

They're related but not identical.

---

## Mistake 3 — Storing messages only in memory

Server restart = messages disappear.

---

## Mistake 4 — Not handling reconnection

Networks fail.

Users switch networks.

Servers restart.

---

## Mistake 5 — Assuming one user has one connection

Users can have multiple tabs and devices.

---

## Mistake 6 — No authentication

A WebSocket connection is still an application entry point.

---

## Mistake 7 — No authorization

Being connected doesn't mean you're allowed to access every room.

---

## Mistake 8 — No rate limiting

A client can abuse a persistent connection.

---

## Mistake 9 — Ignoring message ordering

Distributed systems don't automatically preserve the business order you expect.

---

## Mistake 10 — Assuming `send()` means "delivered"

Sending to a socket is not the same thing as:

```text
stored
delivered
read
```

These are different states.

---

# 66. Interview Questions

## 1. What is a WebSocket?

A protocol that provides persistent, bidirectional communication between a client and server over a connection.

---

## 2. How is WebSocket different from HTTP?

HTTP follows a request-response model where the client typically initiates communication.

WebSockets maintain a persistent connection that allows both client and server to send messages.

---

## 3. What is a WebSocket handshake?

The initial HTTP-based exchange used to establish and upgrade the connection to the WebSocket protocol.

---

## 4. What is the difference between `ws://` and `wss://`?

`wss://` uses TLS encryption, while `ws://` does not.

Production applications should generally use `wss://`.

---

## 5. What is polling?

The client repeatedly sends requests to check whether new data is available.

---

## 6. WebSocket vs SSE?

WebSockets provide bidirectional communication.

SSE is primarily server-to-client streaming over HTTP.

---

## 7. What is Socket.IO?

A real-time communication library that provides higher-level features such as events, rooms, broadcasting, and reconnection support.

It is not simply another name for the WebSocket protocol.

---

## 8. What is a WebSocket room?

A logical group of connections that allows messages to be broadcast to a particular group of clients.

---

## 9. How do you scale WebSockets across multiple servers?

Use a load balancer for connections and shared infrastructure such as Redis Pub/Sub to distribute events between WebSocket servers.

Persistent application data should be stored in a durable data store.

---

## 10. Why do we need Redis Pub/Sub?

When users are connected to different WebSocket servers, Redis Pub/Sub can distribute real-time events between those servers.

---

## 11. What happens when a WebSocket server crashes?

Existing connections are lost.

Clients should reconnect, and the application should synchronize any state or messages missed during the disconnection.

---

## 12. How do you handle duplicate messages?

Use message IDs and appropriate idempotency/deduplication logic.

---

## 13. How do you handle WebSocket authentication?

Authenticate the connection during setup or through an appropriate authentication mechanism, associate it with the authenticated identity, and authorize every protected action.

---

## 14. How would you implement online/offline status?

Track active connections per user and use connection lifecycle events plus heartbeat mechanisms.

A user is generally considered online if they still have an active connection.

---

# 67. Interview Scenario

> You built a chat application using WebSockets. It works perfectly on one server. You add a second server, and users connected to different servers can't receive each other's messages. Why?

Think about it.

```text
Rahul
 ↓
Server A
```

Aman:

```text
Aman
 ↓
Server B
```

Rahul's message reaches:

```text
Server A
```

Server A doesn't necessarily know about Aman's connection on Server B.

You need a way for the servers to communicate.

For example:

```text
Server A
   ↓
Redis Pub/Sub
   ↓
Server B
   ↓
Aman
```

That's the basic scaling solution.

---

# 68. Interview Scenario

> A user loses internet for 30 seconds. During that time, five messages arrive. When they reconnect, how do you make sure they don't miss them?

Don't depend only on WebSocket.

Persist messages:

```text
PostgreSQL
```

When the client reconnects:

```text
Reconnect
   ↓
Tell server last received message ID
   ↓
Server finds newer messages
   ↓
Send missed messages
```

This is much more reliable than assuming the socket will somehow remember everything.

---

# 69. Interview Scenario

> Your WebSocket server has 100,000 connected users. One user starts sending thousands of messages every second. What do you do?

Think:

```text
Authentication
Authorization
Rate limiting
Message size limits
Connection limits
Monitoring
```

For example:

```text
User
 ↓
10,000 messages/sec
 ↓
Rate limit
 ↓
Reject / throttle
```

Don't allow one connection to consume unlimited resources.

---

# 70. Interview Scenario

> Why not store WebSocket connections in PostgreSQL?

Because the connection itself is a live runtime resource.

You generally don't want your database acting as a connection registry for every socket operation.

Instead:

```text
WebSocket server memory
→ active connections

Redis
→ shared ephemeral state/events when needed

PostgreSQL
→ durable application data
```

Each layer has a different responsibility.

---

# 71. Interview Scenario

> Your chat messages sometimes arrive out of order. Why?

Possible reasons include:

```text
Network timing
Multiple servers
Concurrent processing
Different message paths
Client-side rendering
```

You need an explicit ordering strategy.

For example:

```text
message sequence number
```

or:

```text
server-assigned ordering
```

depending on the system.

The important part is:

> Don't assume network arrival order is automatically business order.

---

# 72. Build It Yourself

Now build a small chat application.

Start simple.

### Step 1 — Create the WebSocket server

```text
Node.js
   ↓
WebSocket / Socket.IO
```

---

### Step 2 — Connect the browser

```text
Browser
   ↓
WebSocket
   ↓
Server
```

---

### Step 3 — Send messages

```text
Client
 ↓
message
 ↓
Server
```

---

### Step 4 — Broadcast

```text
Server
 ↓
Other connected clients
```

---

### Step 5 — Add rooms

```text
Room A
 ├── User 1
 ├── User 2
 └── User 3

Room B
 ├── User 4
 └── User 5
```

---

### Step 6 — Add authentication

Use the authentication concepts from Chapter 10.

---

### Step 7 — Store messages

Use PostgreSQL.

```text
message
 ├── id
 ├── conversationId
 ├── senderId
 ├── content
 └── createdAt
```

---

### Step 8 — Add reconnection

Handle:

```text
connect
disconnect
reconnect
```

---

### Step 9 — Add missed-message synchronization

Track:

```text
lastReceivedMessageId
```

and fetch anything missed after reconnecting.

---

### Step 10 — Scale it

Run:

```text
Server A
Server B
```

and connect users to both.

Then introduce:

```text
Redis Pub/Sub
```

to synchronize events.

---

# 73. Your Final Architecture

By the end, you should be able to build something like:

```text
                         Clients
                       /    |    \
                      /     |     \
                     ↓      ↓      ↓
                  WebSocket Connections
                     \      |      /
                      ↓     ↓     ↓
                  Load Balancer
                  /      |      \
                 ↓       ↓       ↓
              WS-1     WS-2     WS-3
                │        │        │
                └────────┼────────┘
                         ↓
                   Redis Pub/Sub
                         │
                         ↓
                  Application Logic
                         │
                         ↓
                    PostgreSQL
                         │
                         ↓
                  Message History
```

And the responsibilities are clear:

```text
WebSocket
→ real-time communication

WebSocket servers
→ maintain client connections

Redis Pub/Sub
→ distribute live events across servers

PostgreSQL
→ durable message/application data

Load balancer
→ distribute connections
```

---

# 74. The Mental Model

If you remember only one thing from this chapter, remember this:

```text
HTTP:

Client
  ↓
Request
  ↓
Server
  ↓
Response
```

WebSocket:

```text
Client
  ↕
Persistent connection
  ↕
Server
```

And for a real application:

```text
Client
   ↕
WebSocket
   ↓
Server
   ↓
Database
```

When you scale:

```text
                 Load Balancer
                /      |      \
               ↓       ↓       ↓
             WS-1    WS-2    WS-3
               \       |       /
                \      |      /
                 Redis Pub/Sub
                       ↓
                   PostgreSQL
```

And then ask yourself:

```text
What happens if the connection drops?

What happens if the server crashes?

What if the user has 3 devices?

What if the same message arrives twice?

What if messages arrive out of order?

What if the user is offline?

What if Redis goes down?

What if one user sends 10,000 messages/sec?

What if we have 1 million connections?
```

That's the mindset I want you to build.

Because **WebSockets are easy to demonstrate.**

The hard part is building a real-time system that still behaves correctly when the network, servers, clients, and dependencies don't behave perfectly.

---

# 75. What You Should Know After Chapter 15

You should now be able to explain:

```text
✓ What WebSockets are
✓ Why they are useful
✓ WebSocket handshake
✓ ws:// vs wss://
✓ Polling
✓ Long polling
✓ SSE
✓ WebSocket vs HTTP
✓ Socket.IO
✓ Events
✓ Rooms
✓ Broadcasting
✓ Authentication
✓ Reconnection
✓ Heartbeats
✓ Message ordering
✓ Message IDs
✓ Acknowledgements
✓ Redis Pub/Sub
✓ Scaling WebSocket servers
✓ Offline users
✓ Persistent message storage
✓ Rate limiting
✓ Backpressure
```

More importantly, you should be able to look at a requirement like:

> "Build a chat application that supports 100,000 concurrent users."

and start asking the right questions instead of immediately writing:

```js
socket.on("message", ...)
```

That's the real goal of this series.

---

# Next — Chapter 16: File Uploads & Storage

We've handled:

```text
HTTP
APIs
Databases
Authentication
Caching
Background jobs
WebSockets
```

But our applications still need to handle something very common:

> **Files.**

Users upload:

```text
Profile pictures
PDFs
Videos
Documents
Images
Resumes
```

And here's the problem:

You probably don't want to store a 500 MB video directly inside PostgreSQL.

We'll learn:

```text
File uploads
Multipart/form-data
Object storage
Amazon S3
Presigned URLs
CDNs
Large file uploads
File validation
Upload security
Storage architecture
```

And we'll answer:

> **Why do production applications usually upload large files directly to object storage instead of sending the entire file through the backend server?**