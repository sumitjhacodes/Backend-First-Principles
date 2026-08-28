# Chapter 1 — How the Internet Actually Works

If you're starting backend development, this is probably one of the first questions you should be able to answer:

> **"What actually happens when I type `google.com` in my browser and press Enter?"**

You might have heard some words already:

* DNS
* IP address
* TCP
* UDP
* HTTP
* HTTPS
* Port
* Server
* Packet

And maybe you've memorized a few definitions.

That's not what we're going to do here.

I want you to understand **what is actually happening**.

Because once you understand this, a lot of backend concepts that come later start making much more sense.

---

## 1. First, What Is the Internet?

Let's start with the simplest possible explanation.

The internet is basically a **huge network of computers connected to each other**.

Your laptop is one computer.

Your phone is another computer.

The server running your backend is another computer.

Google has thousands of computers.

Amazon has thousands of computers.

All of these computers can communicate with each other through networks.

That's the internet.

It's not some magical cloud somewhere.

When you upload a photo to a website, that photo is eventually travelling from your device to another computer somewhere in the world.

When you request:

```text
GET /users/123
```

your request travels through a network and eventually reaches a server.

The server processes it and sends something back.

So at a very high level:

```text
Your Computer
     |
     | request
     ↓
   Internet
     |
     ↓
Backend Server
     |
     | response
     ↓
   Internet
     |
     ↓
Your Computer
```

That's the basic idea behind almost everything we'll build in this series.

---

# 2. But How Does My Computer Find Another Computer?

This is where things become interesting.

Imagine I tell you:

> "Go to Rahul's house."

You can't really do that unless you know where Rahul lives.

Computers have the same problem.

If your browser wants to communicate with a server, it needs to know **where that server is**.

That's where an **IP address** comes in.

---

# 3. What Is an IP Address?

An IP address identifies a device on a network.

For example:

```text
142.250.195.14
```

That's an IPv4 address.

You don't need to memorize any particular IP address. Just understand the idea:

> **An IP address is an address used to identify a device on a network.**

Think about your home address.

If someone wants to send you a package, they need an address.

Similarly, if your computer wants to send data to another computer, it needs an address.

That address can be an IP address.

---

## IPv4

The most common format you'll see when starting is IPv4.

It looks like:

```text
192.168.1.10
```

It's four numbers separated by dots.

Each number can range from:

```text
0 → 255
```

You will often see addresses like:

```text
127.0.0.1
192.168.1.5
10.0.0.10
```

Don't worry about what all of them mean yet. We'll come back to them later.

---

# 4. Then Why Don't We Type IP Addresses?

Good question.

Technically, we could.

You could type an IP address into your browser.

But imagine having to remember:

```text
142.250.195.14
```

instead of:

```text
google.com
```

Not exactly friendly.

And IP addresses can change.

So instead of asking humans to remember IP addresses, we created **DNS**.

---

# 5. DNS — The Internet's Phonebook

DNS stands for:

> **Domain Name System**

The easiest way to understand DNS:

> **DNS converts domain names into IP addresses.**

For example:

```text
google.com
     ↓
142.250.x.x
```

You type:

```text
google.com
```

Your computer needs an IP address.

DNS helps it find that IP address.

A simple analogy:

Imagine your phone contains:

```text
Rahul → +91 98XXXXXXXX
```

You don't remember Rahul's phone number.

You search:

```text
Rahul
```

and your phone finds the number.

DNS works somewhat like that:

```text
google.com → IP address
github.com → IP address
example.com → IP address
```

That's why people often call DNS the **phonebook of the internet**.

It's not a perfect analogy, but it's useful when you're starting.

---

# 6. What Is a Domain Name?

A domain name is the human-friendly name we use to reach a website.

Examples:

```text
google.com
github.com
amazon.com
openai.com
```

Instead of remembering an IP address, we remember a name.

The domain eventually resolves to an IP address.

So:

```text
Domain
   ↓
DNS lookup
   ↓
IP address
```

---

# 7. Okay, We Have an IP Address. Now What?

Now your computer knows **which machine** it wants to communicate with.

But there's another problem.

Imagine one computer is running many applications.

For example, a server might be running:

```text
Website
API
Database
SSH
Email server
```

How does the network know which application should receive the incoming data?

That's where **ports** come in.

---

# 8. What Is a Port?

A port is basically a number used to identify a particular service/application on a machine.

Think of an IP address as:

> **The building address**

And a port as:

> **The apartment/office number inside that building**

For example:

```text
IP address: 192.168.1.10
Port:       3000
```

Together:

```text
192.168.1.10:3000
```

means:

> "Connect to port 3000 on this machine."

When you're developing with Node.js, you might write:

```js
app.listen(3000);
```

That means your server is listening for connections on port `3000`.

So when you visit:

```text
http://localhost:3000
```

you're saying:

> Connect to this machine on port 3000.

---

# 9. What Is `localhost`?

You'll see this constantly as a backend developer.

For example:

```text
http://localhost:3000
```

`localhost` basically means:

> **This computer.**

It's a name that points back to your own machine.

Another common representation is:

```text
127.0.0.1
```

So:

```text
localhost
```

and:

```text
127.0.0.1
```

usually refer to your own computer in local development.

For example, when you're developing an Express server:

```text
http://localhost:3000
```

you're not talking to some server on the internet.

You're talking to your own computer.

---

# 10. We Have an Address. Now How Do Computers Communicate?

Now we know:

```text
Domain
   ↓
IP address
   ↓
Port
```

But we still need rules for communication.

Two computers can't just randomly send bytes to each other and hope everything works.

They need an agreed set of rules.

These rules are called **protocols**.

You'll hear this word constantly in backend engineering.

---

# 11. What Is a Protocol?

A protocol is simply:

> **A set of rules that defines how computers communicate.**

Think about a conversation.

If I speak English and you only understand Japanese, communication is going to be difficult.

We need some common language.

Protocols play a similar role for computers.

Examples:

```text
HTTP
HTTPS
TCP
UDP
DNS
WebSocket
SMTP
```

Each solves a different communication problem.

For now, the two we really need to understand are:

```text
TCP
UDP
```

---

# 12. TCP — Reliable Communication

TCP stands for:

> **Transmission Control Protocol**

The important thing to remember:

> **TCP provides reliable, ordered delivery of data.**

Suppose you're sending a large file.

The data may be broken into smaller pieces and sent across the network.

TCP helps make sure:

* data arrives
* data arrives in the correct order
* lost data can be retransmitted
* the connection is managed properly

Think about sending an important document through a delivery service.

You want to know:

> Did it arrive?

> Did everything arrive?

> Is anything missing?

> Is it in the right order?

TCP provides mechanisms for this kind of reliability.

---

# 13. TCP Connection

Before TCP starts sending application data, the two sides establish a connection.

You'll often hear about the:

> **TCP three-way handshake**

It roughly looks like:

```text
Client                    Server

  SYN  -------------------->
       <----------------- SYN-ACK
  ACK  -------------------->
```

You don't need to memorize the packet-level details right now.

The important idea is:

```text
Client wants connection
        ↓
Server agrees
        ↓
Connection established
```

After that, they can communicate using the TCP connection.

---

# 14. UDP — Fast but No Delivery Guarantee

UDP stands for:

> **User Datagram Protocol**

UDP is much simpler than TCP.

It doesn't provide the same reliability guarantees.

You send data.

It doesn't guarantee:

* delivery
* ordering
* retransmission

So why would anyone use it?

Because sometimes **speed matters more than perfect delivery**.

Imagine a live multiplayer game.

You're moving your character:

```text
x = 100
y = 200
```

Then:

```text
x = 102
y = 201
```

Then:

```text
x = 105
y = 203
```

If one update gets lost, you probably don't want the system to stop everything and wait for that old update.

The next update is already coming.

That's one reason UDP can be useful for real-time applications.

---

# 15. TCP vs UDP

Don't memorize a giant table.

Remember the basic tradeoff:

```text
TCP
↓
Reliable
Ordered
Connection-oriented
More overhead

UDP
↓
Fast
No delivery guarantee
No ordering guarantee
Lower overhead
```

A simple way to remember it:

> **TCP cares about getting the data there correctly.**

> **UDP cares more about getting the data there quickly.**

The real world is more complicated than this, but this mental model is enough for now.

---

# 16. Now Let's Talk About HTTP

This is where backend development really starts becoming familiar.

HTTP stands for:

> **HyperText Transfer Protocol**

HTTP is an application-level protocol used for communication between clients and servers.

You've probably already used it without realizing it.

For example:

```text
GET https://example.com/users
```

The client sends an HTTP request.

The server sends an HTTP response.

```text
Client
   |
   | HTTP Request
   ↓
Server
   |
   | HTTP Response
   ↓
Client
```

We'll spend an entire chapter on HTTP later.

For now, just remember:

> **HTTP defines how web clients and servers communicate.**

---

# 17. What Is HTTPS?

You've probably noticed URLs like:

```text
https://github.com
```

instead of:

```text
http://github.com
```

HTTPS is essentially HTTP communication protected with **TLS encryption**.

You don't need to understand cryptography yet.

The important idea is:

```text
HTTP
↓
communication

HTTPS
↓
HTTP + encrypted connection
```

Without encryption, someone positioned in the right place on the network could potentially observe sensitive information.

HTTPS helps protect the communication.

That's why production websites and APIs should generally use HTTPS.

---

# 18. So What Actually Happens When You Type a URL?

Now let's put everything together.

Suppose you type:

```text
https://example.com
```

into your browser.

A simplified version of what happens is:

```text
1. Browser parses the URL
        ↓
2. DNS lookup finds the server's IP
        ↓
3. Client connects to the server
        ↓
4. TCP connection is established
        ↓
5. TLS connection is established
        ↓
6. Browser sends HTTP request
        ↓
7. Server receives request
        ↓
8. Server processes request
        ↓
9. Server sends HTTP response
        ↓
10. Browser receives response
        ↓
11. Browser renders the result
```

That's the big picture.

And yes, the real process has more details and optimizations.

We're deliberately keeping the first mental model simple.

---

# 19. Let's Look at a Real Request

Suppose your browser requests:

```text
https://example.com/users
```

Eventually, the server might receive something conceptually similar to:

```http
GET /users HTTP/1.1
Host: example.com
Accept: application/json
```

The server processes that request.

Maybe it queries a database:

```text
HTTP Request
     ↓
Backend
     ↓
Database
     ↓
Backend
     ↓
HTTP Response
```

The response might look like:

```http
HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "id": 1,
    "name": "Rahul"
  }
]
```

Now we're getting very close to the backend work you'll be doing.

---

# 20. Where Does Your Backend Fit?

Let's say you build a Node.js backend.

You might have:

```text
Frontend
   |
   | HTTP request
   ↓
Node.js server
   |
   ↓
Express/Fastify
   |
   ↓
Your business logic
   |
   ↓
PostgreSQL
```

The backend isn't "the internet."

It's one application running somewhere on a computer/server and communicating with other systems through networks and protocols.

This distinction is important.

---

# 21. What Is a Server, Then?

The word "server" can be confusing because people use it in different ways.

A server can simply mean:

> **A computer that provides some service to other computers.**

But developers also say:

> "My Node server is running."

In that case, they might actually mean:

> "My Node.js application is listening for incoming network requests."

We'll properly unpack this in Chapter 3.

For now, keep these two ideas separate:

```text
Server machine
↓
A computer

Server application
↓
A program listening for requests
```

---

# 22. What Are Packets?

When you send data across a network, the entire thing isn't necessarily sent as one giant block.

Data can be broken into smaller pieces called **packets**.

Think of sending a large book.

Instead of putting the entire book into one enormous box, you could divide it into several packages:

```text
Package 1
Package 2
Package 3
Package 4
```

The network moves these pieces around.

Networking protocols contain rules for handling these pieces.

You don't need to become a networking engineer to build APIs, but knowing that data travels through packets helps explain things like:

* latency
* packet loss
* retransmission
* network congestion
* connection problems

We'll encounter these concepts later.

---

# 23. What Is Latency?

Latency is basically:

> **How long it takes for data to travel between two points.**

For example:

```text
Client → Server
```

might take:

```text
40ms
```

A server could be extremely fast at processing a request, but if the network takes a long time, the user still experiences a slow response.

This becomes very important when you start thinking about production systems.

For example:

```text
API processing: 20ms
Database:       30ms
Network:        150ms
----------------------
Total:          ~200ms
```

So when an API is slow, don't immediately assume:

> "The Node.js code must be slow."

The problem could be:

* network latency
* database query
* external API
* DNS
* connection setup
* server overload
* serialization
* many other things

This is the beginning of thinking like a backend engineer.

---

# 24. A Few Terms You'll Keep Hearing

Let's quickly clean up some jargon.

### Client

The thing making the request.

Examples:

```text
Browser
Mobile app
Frontend
curl
Postman
Another backend service
```

---

### Server

The system receiving the request and providing some service.

---

### IP Address

An address identifying a device/network interface.

Example:

```text
192.168.1.10
```

---

### Domain

A human-friendly name that can be resolved to an IP address.

Example:

```text
github.com
```

---

### DNS

The system used to resolve domain names to IP addresses.

```text
github.com
     ↓
IP address
```

---

### Port

A number used to identify a network service on a machine.

Example:

```text
localhost:3000
```

---

### Protocol

Rules for communication.

Examples:

```text
HTTP
TCP
UDP
DNS
```

---

### Packet

A small unit of data sent across a network.

---

### Latency

The time taken for data to travel between systems.

---

# 25. Common Beginner Mistakes

### Mistake 1: "The internet is a server."

No.

The internet is a network of interconnected systems.

A server is one machine/system providing some service.

---

### Mistake 2: "DNS is the internet."

No.

DNS is one system used on the internet to resolve names into addresses.

---

### Mistake 3: "HTTP and TCP are the same thing."

They're not.

A simplified way to think about the relationship is:

```text
HTTP
 ↓
Application-level protocol

TCP
 ↓
Transport-level protocol
```

HTTP can use TCP as its underlying transport.

---

### Mistake 4: "Port 3000 is the server."

No.

`3000` is just a port number.

For example:

```text
localhost:3000
```

means:

```text
this computer
+
port 3000
```

---

### Mistake 5: "localhost means the internet."

Actually, it's the opposite in most development situations.

```text
localhost
```

usually means:

> "My own machine."

---

### Mistake 6: "If my API is slow, my code must be slow."

Not necessarily.

Always ask:

```text
Where is the time going?
```

Maybe it's:

```text
DNS
Network
Database
External API
Queue
CPU
Lock
Connection pool
```

We'll learn how to investigate these problems later.

---

# 26. Let's Do a Small Experiment

You don't need to build a complete application for this chapter.

Open your terminal.

Try:

```bash
ping google.com
```

You'll probably see something similar to:

```text
Reply from ...
time=...
```

The exact output will depend on your operating system and network.

The interesting part isn't the command itself.

It's the idea.

Your computer is communicating with another machine and measuring the response.

---

## Try DNS

Run:

```bash
nslookup google.com
```

or, on systems where it's available:

```bash
dig google.com
```

You'll see information about how the domain resolves.

Again, don't worry about every line.

Just observe:

```text
google.com
    ↓
IP address
```

That's DNS in action.

---

## Try curl

You can also make an HTTP request from the terminal:

```bash
curl https://example.com
```

Now you're doing something very similar to what a browser does:

```text
Your computer
     ↓
HTTP request
     ↓
example.com
     ↓
HTTP response
     ↓
Your terminal
```

We'll use `curl` much more in Chapter 6.

---

# 27. A Small Mental Model

If you remember only one diagram from this chapter, remember this:

```text
                 INTERNET
                    |
        ┌───────────┴───────────┐
        ↓                       ↓
     CLIENT                    SERVER
        |                       |
        |      Network          |
        └───────────────────────┘
                  |
             IP Address
                  |
                Port
                  |
             TCP / UDP
                  |
                HTTP
                  |
          Request / Response
```

This isn't a literal network architecture diagram.

It's a mental model.

It helps you understand where the different concepts fit.

---

# 28. What You Should Be Able to Explain Now

Before moving to the next chapter, try answering these without looking back.

### Beginner level

**1. What is the internet?**

A network of interconnected computers and networks that communicate using agreed protocols.

**2. What is an IP address?**

An address used to identify a device/interface on a network.

**3. What is DNS?**

A system that helps translate domain names into IP addresses.

**4. What is a port?**

A number used to identify a particular network service/application on a machine.

**5. What is localhost?**

A hostname that refers back to the local machine.

---

### Intermediate level

**6. What is TCP?**

A transport protocol that provides reliable, ordered communication between endpoints.

**7. What is UDP?**

A connectionless transport protocol with lower overhead but without TCP's delivery and ordering guarantees.

**8. TCP vs UDP?**

TCP prioritizes reliable, ordered delivery. UDP prioritizes simplicity and lower overhead where the application can tolerate loss or handle reliability itself.

**9. What is HTTP?**

An application-layer protocol used for communication between clients and servers.

**10. What is HTTPS?**

HTTP sent over a TLS-protected connection.

---

### Interview-style question

**"What happens when you type `https://google.com` into your browser?"**

Don't give a memorized one-liner.

Try explaining it like this:

> First, the browser needs to figure out where `google.com` is, so it performs DNS resolution to get an IP address.
>
> Then it needs to communicate with the server. For HTTPS, a secure connection is established using TCP and TLS.
>
> Once the connection is ready, the browser sends an HTTP request.
>
> The server receives the request, processes it, and sends an HTTP response.
>
> The browser receives the response and uses it to render the page.
>
> There are a lot of optimizations and details underneath this, but that's the basic flow.

That's already a much better answer than:

> "DNS then TCP then HTTP then browser."

The goal isn't to memorize the pipeline.

**The goal is to understand it.**

---

# 29. What Can You Build With This Knowledge?

You might be thinking:

> "Okay, this is interesting, but I'm here to learn backend development. When do I actually use this?"

Pretty much immediately.

When you build a backend, you'll constantly work with:

```text
IP addresses
ports
HTTP
requests
responses
clients
servers
DNS
network latency
connections
```

For example, when you write:

```js
app.listen(3000);
```

you're using the concept of ports.

When you call:

```js
fetch("https://api.example.com/users");
```

you're using DNS, networking, HTTPS and HTTP.

When your frontend calls your backend:

```text
Frontend
   ↓
HTTP
   ↓
Backend
   ↓
Database
```

you're applying the concepts from this chapter.

So this isn't just networking theory.

It's the foundation underneath the backend code you'll write.

---

# 30. A Small Challenge

Before moving to Chapter 2, try this yourself.

Run:

```bash
nslookup github.com
```

Then:

```bash
curl https://github.com
```

Then ask yourself:

1. What did DNS do?
2. What does the IP address represent?
3. What is the client?
4. What is the server?
5. Where does HTTPS come into the picture?
6. What is the HTTP request?
7. What is the HTTP response?
8. What does the port represent?

You don't need to know every detail yet.

If you can explain the basic flow in your own words, you're ready for the next chapter.

---

# 31. Where We Go From Here

Right now we understand the world underneath our backend:

```text
Internet
   ↓
IP
   ↓
DNS
   ↓
Ports
   ↓
TCP / UDP
   ↓
HTTP / HTTPS
```

But we've only touched HTTP.

And HTTP is **the language we'll be speaking constantly as backend developers**.

So next we'll slow down and properly understand it.

We'll look at things like:

```text
GET
POST
PUT
PATCH
DELETE

Headers
Body
Status codes
Query parameters
Path parameters
Content-Type
Cookies
Statelessness
Idempotency
```

And instead of memorizing definitions, we'll actually send requests and see what changes.

---

## Before moving on

If you are completely new to backend development, don't rush through this chapter.

You don't need to become a networking expert.

But you should be able to look at:

```text
https://api.example.com:3000/users?id=42
```

and roughly understand:

```text
https
  ↓
Protocol

api.example.com
  ↓
Domain

:3000
  ↓
Port

/users
  ↓
Path

?id=42
  ↓
Query parameter
```

That little bit of understanding will save you a surprising amount of confusion later.

**Next: Chapter 2 — HTTP Deep Dive.**
