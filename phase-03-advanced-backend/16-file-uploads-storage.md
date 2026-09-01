# Chapter 16 — File Uploads & Storage

So far, most of the data we've sent to our backend has looked like this:

```json
{
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

Small JSON.

Easy.

But real applications don't only deal with JSON.

Users upload:

```text
Profile pictures
Resumes
PDFs
Videos
Documents
Invoices
Audio files
Product images
Attachments
```

Now imagine someone uploads:

```text
video.mp4

Size: 2 GB
```

Suddenly our normal:

```text
Client
   ↓
Backend
   ↓
PostgreSQL
```

architecture needs some thinking.

Should we put the entire 2 GB video inside PostgreSQL?

Should we save it inside:

```text
/uploads
```

on our Node.js server?

Should the entire 2 GB file even pass through our backend?

And what happens when we have:

```text
1 server
      ↓
10 servers
      ↓
100 servers
```

Where does the file live then?

That's what this chapter is about.

---

# 1. First: What Is a File to the Backend?

Let's remove some magic.

A file is ultimately:

```text
bytes
```

Your beautiful:

```text
profile.jpg
```

is data.

Your:

```text
resume.pdf
```

is data.

Your:

```text
video.mp4
```

is data.

The filename, type, size, owner, and other information are metadata around those bytes.

For example:

```text
File:

name:
resume.pdf

type:
application/pdf

size:
524288 bytes

content:
actual file bytes
```

When a browser uploads a file, those bytes need to travel somewhere.

---

# 2. Why Can't We Just Send JSON?

Imagine trying:

```json
{
  "name": "Rahul",
  "resume": ???
}
```

JSON is text-based.

Raw binary file data doesn't naturally fit into normal JSON.

You *can* encode binary data into something like Base64:

```json
{
  "resume": "JVBERi0xLjQKJ..."
}
```

But that often isn't ideal for normal file uploads.

Base64 makes the data larger and adds encoding/decoding overhead.

For ordinary uploads, browsers commonly use:

> **multipart/form-data**

---

# 3. What Is `multipart/form-data`?

You've probably used an HTML form like:

```html
<form enctype="multipart/form-data">
  <input type="text" name="name" />
  <input type="file" name="resume" />
</form>
```

The browser needs to send:

```text
normal text fields
+
file bytes
```

in one request.

`multipart/form-data` allows the request body to contain multiple separate parts.

Conceptually:

```text
Request Body

--------------------
name
Rahul
--------------------
resume
[file bytes]
--------------------
```

That's why it's called:

```text
multipart
```

The body contains multiple parts.

---

# 4. What Does the Backend Receive?

Suppose the client sends:

```text
POST /profile
```

with:

```text
name = Rahul
avatar = profile.jpg
```

The backend receives something conceptually like:

```text
HTTP Request
     ↓
multipart/form-data
     ↓
┌──────────────────┐
│ name             │
│ Rahul            │
├──────────────────┤
│ avatar           │
│ profile.jpg      │
│ binary data...   │
└──────────────────┘
```

Your backend needs something that can parse this format.

In Express applications, one common tool is:

> **Multer**

---

# 5. What Is Multer?

Multer is middleware for handling `multipart/form-data` in Node.js/Express applications.

Remember middleware from Chapter 3?

```text
Request
   ↓
Middleware
   ↓
Route Handler
```

Multer sits in that request pipeline:

```text
Client
   ↓
multipart request
   ↓
Multer
   ↓
Extract file
   ↓
Controller
```

So your controller doesn't have to manually parse multipart boundaries and raw file bytes.

---

# 6. Basic Multer Example

Install:

```bash
npm install multer
```

Then:

```js
import multer from "multer";

const upload = multer({
  dest: "uploads/"
});
```

Route:

```js
app.post(
  "/avatar",
  upload.single("avatar"),
  (req, res) => {
    console.log(req.file);

    res.json({
      message: "Uploaded"
    });
  }
);
```

Now if the client uploads:

```text
avatar = profile.jpg
```

Multer processes it.

---

# 7. `req.body` vs `req.file`

If the form contains:

```text
name = Rahul
avatar = profile.jpg
```

you'll typically get:

```text
req.body
   ↓
normal text fields
```

and:

```text
req.file
   ↓
uploaded file information
```

For example:

```js
console.log(req.body.name);
console.log(req.file);
```

Again, don't memorize the exact API.

Understand the flow.

---

# 8. Multiple Files

Sometimes users upload multiple files.

For example:

```text
product image 1
product image 2
product image 3
```

Then you might use:

```js
upload.array("images", 5);
```

Meaning:

```text
field = images
maximum files = 5
```

Then:

```text
req.files
```

contains the uploaded files.

But notice something important.

We're already putting limits on the upload.

That's intentional.

---

# 9. Never Allow Unlimited Uploads

Imagine your endpoint:

```http
POST /upload
```

accepts anything.

An attacker sends:

```text
500 GB file
```

Your server might run out of:

```text
Disk
Memory
Bandwidth
```

So uploads need limits.

For example:

```js
const upload = multer({
  limits: {
    fileSize: 5 * 1024 * 1024
  }
});
```

That means approximately:

```text
5 MB maximum
```

The exact limit depends on your application.

---

# 10. File Validation

Suppose your endpoint expects:

```text
Profile image
```

Should someone be able to upload:

```text
movie.mp4
```

Probably not.

You should validate things such as:

```text
File size
File type
Allowed extension
Number of files
```

But there's an important security detail.

---

# 11. Never Trust the Filename

Someone uploads:

```text
cute-cat.jpg
```

Does that prove it's an image?

No.

The client controls the filename.

They could rename:

```text
malicious-file.exe
```

to:

```text
cute-cat.jpg
```

So:

```text
extension
```

alone isn't enough to establish what a file actually contains.

---

# 12. Don't Blindly Trust `Content-Type` Either

The browser might send:

```http
Content-Type: image/jpeg
```

But that value ultimately comes from the client side of the request.

A malicious client can lie.

So secure file handling may require validating the actual file content/signature where appropriate.

This is sometimes called:

```text
magic byte
```

or:

```text
file signature
```

validation.

For example, many file formats begin with recognizable byte patterns.

The general lesson is:

> **Don't trust user-supplied metadata to prove what a file actually is.**

---

# 13. The First Storage Approach: Local Disk

A beginner application might do:

```text
uploads/
├── avatar1.jpg
├── resume1.pdf
├── avatar2.jpg
└── invoice.pdf
```

This works locally.

Your architecture becomes:

```text
Client
   ↓
Node.js Server
   ↓
Local Disk
```

For a small local project, that's completely fine.

But production introduces problems.

---

# 14. The Multiple Server Problem

Imagine your application grows.

Now:

```text
             Load Balancer
             /          \
            ↓            ↓
        Server A      Server B
```

A user uploads:

```text
profile.jpg
```

The request reaches:

```text
Server A
```

So the file gets stored:

```text
Server A
/uploads/profile.jpg
```

Later another request reaches:

```text
Server B
```

Server B tries:

```text
/uploads/profile.jpg
```

But the file isn't there.

It exists only on Server A.

Now we have a problem.

---

# 15. Server Disks Aren't Shared Storage

Your architecture looks like:

```text
Server A
   ↓
Disk A

Server B
   ↓
Disk B

Server C
   ↓
Disk C
```

Each server has its own filesystem.

That's one reason storing user uploads on application-server local disks becomes problematic when scaling horizontally.

---

# 16. Deployment Makes This Worse

Suppose your hosting platform replaces your server during deployment.

Old server:

```text
Server A

/uploads
   ↓
100,000 user images
```

Deployment happens.

New server:

```text
Server B

/uploads
   ↓
empty
```

Depending on the hosting environment, local filesystem storage may be ephemeral.

Your application files shouldn't depend on a particular application instance surviving forever.

---

# 17. Enter Object Storage

Instead of:

```text
Client
   ↓
Backend
   ↓
Server Disk
```

we use:

```text
Client
   ↓
Backend
   ↓
Object Storage
```

One of the most common examples is:

> **Amazon S3**

S3 stands for:

```text
Simple Storage Service
```

It's an object storage service.

---

# 18. What Is Object Storage?

Think of object storage as a large system designed for storing objects/files.

You might store:

```text
avatars/user-123/profile.jpg

resumes/user-456/resume.pdf

products/product-789/image.webp
```

Each object typically has:

```text
Key
Data
Metadata
```

For example:

```text
Key:
avatars/user-123/profile.jpg

Data:
[file bytes]

Content-Type:
image/jpeg
```

---

# 19. Bucket

In S3, objects live inside:

> **Buckets**

Think:

```text
my-app-uploads
│
├── avatars/
├── resumes/
├── products/
└── documents/
```

The bucket is the top-level storage container.

Inside it, objects have keys.

---

# 20. S3 Doesn't Really Have Normal Folders

This surprises some beginners.

You might see:

```text
avatars/user-123/profile.jpg
```

and think:

```text
folder
 ↓
folder
 ↓
file
```

But object storage primarily works with object keys.

The `/` characters make keys *look* like folders.

Conceptually:

```text
key =
"avatars/user-123/profile.jpg"
```

This distinction matters when you start working with object-storage APIs.

---

# 21. What Goes in PostgreSQL?

Suppose a user uploads:

```text
resume.pdf
```

Should PostgreSQL store the entire PDF?

Usually, for typical web applications, you instead store metadata/reference information.

For example:

```text
PostgreSQL

user_id
file_key
file_name
file_size
mime_type
created_at
```

Example:

```text
user_id:
123

file_key:
resumes/user-123/abc123.pdf

file_name:
resume.pdf
```

Then:

```text
S3
 ↓
actual file bytes

PostgreSQL
 ↓
metadata / relationship
```

That's a very common architecture.

---

# 22. Why Not Store Files Directly in PostgreSQL?

PostgreSQL *can* store binary data.

So saying:

> "Databases cannot store files."

would be wrong.

The better question is:

> **Should this application use the relational database for large file storage?**

For many web applications, object storage is a better fit because it's designed specifically for storing and serving large amounts of blob/object data.

Meanwhile PostgreSQL handles:

```text
Users
Relationships
Metadata
Permissions
Application state
```

Use the right system for the right job.

---

# 23. Basic Upload Architecture

Our first production-style architecture could be:

```text
Client
   ↓
Upload
   ↓
API Server
   ↓
S3
```

Then:

```text
API Server
   ↓
PostgreSQL
```

stores the file metadata.

So:

```text
Client
   ↓
Backend
   ├──→ S3
   │     file bytes
   │
   └──→ PostgreSQL
         metadata
```

This works.

But there's another problem.

---

# 24. The Backend Becomes a Middleman

Imagine someone uploads:

```text
2 GB video
```

Architecture:

```text
Client
   ↓
2 GB
   ↓
Backend
   ↓
2 GB
   ↓
S3
```

Your backend receives all 2 GB.

Then sends all 2 GB again.

That means your backend is spending:

```text
Bandwidth
Memory/buffers
Connections
CPU
Time
```

moving data it doesn't really need to inspect or own directly.

Wouldn't it be better if the client could upload straight to S3?

---

# 25. Direct Upload

Ideally:

```text
Client
   ↓
S3
```

But there's a security problem.

You don't want to give the browser your AWS secret credentials.

Never do:

```text
Frontend
   ↓
AWS_SECRET_ACCESS_KEY
```

Anyone could inspect the frontend bundle/network environment and steal the credentials.

So how can the client upload directly without getting permanent AWS credentials?

That's where:

> **Presigned URLs**

come in.

---

# 26. What Is a Presigned URL?

A presigned URL is a temporary URL generated using server-side credentials that grants limited permission for a specific storage operation.

Think of it like a temporary ticket.

Your backend says:

> "For the next few minutes, this client may upload this specific object."

Instead of giving:

```text
Permanent AWS credentials
```

you give:

```text
Temporary limited permission
```

That's the important idea.

---

# 27. Presigned Upload Flow

Now our architecture becomes:

```text
Client
   ↓
"Can I upload profile.jpg?"
   ↓
Backend
   ↓
Authenticate user
   ↓
Validate upload request
   ↓
Generate presigned URL
   ↓
Client
   ↓
Upload directly to S3
```

Visually:

```text
        1. Request upload permission
Client ─────────────────────────────→ Backend

        2. Presigned URL
Client ←───────────────────────────── Backend

        3. Upload file
Client ─────────────────────────────→ S3
```

Notice:

```text
File bytes
```

don't have to pass through our application server.

---

# 28. Why Presigned URLs Are Useful

Without direct upload:

```text
Client
   ↓ 2 GB
Backend
   ↓ 2 GB
S3
```

With presigned upload:

```text
Client
   ↓ 2 GB
S3
```

Backend only handles:

```text
small request
+
small signed URL response
```

This can significantly reduce the load on your API servers.

---

# 29. Example Flow

User wants to upload:

```text
profile.jpg
```

Client calls:

```http
POST /api/v1/uploads/presign
```

with:

```json
{
  "fileName": "profile.jpg",
  "contentType": "image/jpeg"
}
```

Backend authenticates the user.

Then it generates a key:

```text
avatars/user-123/550e8400.jpg
```

and creates a presigned upload URL.

Response:

```json
{
  "uploadUrl": "...temporary signed URL...",
  "fileKey": "avatars/user-123/550e8400.jpg"
}
```

The browser uploads directly to storage.

---

# 30. Don't Let the Client Choose Arbitrary Keys

This is subtle.

Don't simply allow:

```json
{
  "key": "anything/i/want"
}
```

and blindly generate upload permission.

The server should control the storage key.

For example:

```text
authenticated user:
123

generated key:
avatars/123/random-id.jpg
```

This helps prevent users from overwriting other users' objects.

---

# 31. Don't Use Original Filenames as Unique IDs

Imagine two users upload:

```text
resume.pdf
```

If your storage key is simply:

```text
resume.pdf
```

the second upload could collide with the first.

Instead, generate unique keys.

For example:

```text
resumes/
  user-123/
    550e8400.pdf
```

You can still store the original filename separately:

```text
originalName:
resume.pdf
```

This separates:

```text
storage identity
```

from:

```text
display filename
```

---

# 32. Never Trust the Upload Just Because S3 Accepted It

This is important.

A presigned URL only gives permission to perform a particular upload under certain conditions.

It doesn't magically make the uploaded file safe.

A malicious user might still upload unwanted content depending on your signing rules.

Your application may need additional validation or post-upload processing.

For example:

```text
Upload
   ↓
Object Storage
   ↓
Background Job
   ↓
Inspect file
   ↓
Virus/malware scanning
   ↓
Validate actual format
   ↓
Mark safe
```

This is especially important for applications that accept arbitrary documents.

---

# 33. Upload Status

Imagine the backend creates:

```text
file record
```

before the upload happens.

You might store:

```text
status = pending
```

Then:

```text
Client uploads
```

After successful processing:

```text
status = ready
```

If something fails:

```text
status = failed
```

So:

```text
pending
   ↓
uploaded
   ↓
processing
   ↓
ready
```

This is much more robust than assuming:

> "I generated a presigned URL, therefore the upload definitely happened."

It may never happen.

---

# 34. How Does the Backend Know the Upload Finished?

There are several possible approaches.

One simple approach:

```text
Client
   ↓
Upload to S3
   ↓
Success
   ↓
Client calls backend
```

For example:

```http
POST /api/v1/uploads/complete
```

But don't blindly trust:

```text
"Yep, I uploaded it."
```

The backend can verify the object exists and/or use storage events depending on the architecture.

Another approach:

```text
S3
 ↓
Storage event
 ↓
Queue / event system
 ↓
Worker
```

We'll see this event-driven style more later.

---

# 35. Large File Uploads

Imagine uploading:

```text
20 GB video
```

If the connection fails at:

```text
19 GB
```

do we really want to restart from:

```text
0 GB
```

Probably not.

Large object-storage systems support approaches such as:

> **Multipart upload**

This is different from HTTP `multipart/form-data`.

Yes, the naming is confusing.

---

# 36. Two Different "Multipart" Concepts

### `multipart/form-data`

An HTTP request encoding format.

Used to send:

```text
fields + files
```

to a server.

### S3 Multipart Upload

A large-file upload strategy.

The file is divided into pieces:

```text
Large File

Part 1
Part 2
Part 3
Part 4
...
```

The pieces can be uploaded separately and later combined.

Don't confuse the two.

This is a good interview detail.

---

# 37. Multipart Upload for Large Files

Imagine:

```text
10 GB file
```

Instead of:

```text
10 GB
   ↓
one giant request
```

we divide it:

```text
Part 1
Part 2
Part 3
Part 4
...
```

Then:

```text
Client
 ├──→ Part 1
 ├──→ Part 2
 ├──→ Part 3
 └──→ Part 4
```

If:

```text
Part 3
```

fails, you retry Part 3 instead of the entire 10 GB upload.

---

# 38. Parallel Uploads

Multipart upload can also allow parts to upload concurrently.

Instead of:

```text
Part 1
 ↓
Part 2
 ↓
Part 3
 ↓
Part 4
```

you may upload several parts simultaneously:

```text
      ┌── Part 1
Client├── Part 2
      ├── Part 3
      └── Part 4
```

This can improve upload performance when used appropriately.

But concurrency should still have limits.

---

# 39. Incomplete Multipart Uploads

What if the user starts:

```text
20 GB upload
```

uploads five parts and closes the browser?

Those uploaded parts can consume storage.

So production systems often need cleanup strategies for abandoned multipart uploads.

Again:

> Upload systems have lifecycle problems, not just `POST /upload`.

---

# 40. Downloading Files

Now the user wants their file back.

One option:

```text
Client
   ↓
Backend
   ↓
S3
   ↓
Backend
   ↓
Client
```

Again, your backend becomes a middleman.

For many applications, a better approach is:

```text
Client
   ↓
Backend
   ↓
Authorize
   ↓
Presigned download URL
   ↓
Client
   ↓
S3
```

The client downloads directly from object storage.

---

# 41. Public vs Private Files

Not every file has the same permissions.

For example:

```text
Public product image
```

might be accessible publicly.

But:

```text
Private resume
Medical document
Invoice
Internal company file
```

shouldn't necessarily be public.

For private objects:

```text
Client
   ↓
Backend
   ↓
Authenticate
   ↓
Authorize
   ↓
Temporary signed URL
   ↓
Download
```

This lets the application control access.

---

# 42. Don't Store Secrets in File URLs

Be careful about making sensitive files permanently public through predictable URLs.

Something like:

```text
/files/user-123/private-document.pdf
```

should not automatically mean:

> Anyone who knows this URL can access it forever.

Authorization should be designed around the sensitivity of the file.

---

# 43. What Is a CDN?

Now imagine your users are around the world.

Your storage is in one region.

User:

```text
India
```

requests an image.

Another user:

```text
Canada
```

requests the same image.

Another:

```text
Germany
```

requests it.

Instead of every request travelling back to the origin storage, we can use:

> **CDN — Content Delivery Network**

---

# 44. CDN Mental Model

Think:

```text
              Origin Storage
                    │
           ┌────────┼────────┐
           ↓        ↓        ↓
        India     Europe    US
        Edge      Edge      Edge
```

Copies of cacheable content can be served from locations closer to users.

So:

```text
User
 ↓
Nearby CDN edge
 ↓
File
```

instead of always:

```text
User
 ↓
Far-away origin
 ↓
File
```

This can reduce latency and origin load.

---

# 45. CDN + S3

A common architecture looks like:

```text
Client
   ↓
CDN
   ↓
S3
```

For uploads:

```text
Client
   ↓
S3
```

For downloads:

```text
Client
   ↓
CDN
   ↓
S3
```

Now S3 is your origin storage and the CDN helps distribute content efficiently.

---

# 46. CDN Caching

Suppose:

```text
logo.png
```

is requested millions of times.

The CDN can cache it.

Instead of:

```text
1,000,000 requests
       ↓
S3
```

you might get:

```text
Users
  ↓
CDN cache
  ↓
Only occasional origin request
  ↓
S3
```

This is caching again.

Chapter 13 wasn't just about Redis.

Caching exists throughout the stack.

---

# 47. Cache Invalidation Returns

Remember the fun problem from Chapter 13?

It's back.

Suppose:

```text
avatar/user-123.jpg
```

is cached by the CDN.

User uploads a new avatar using the same key.

Some CDN locations may still have the old version cached.

One simple technique is using versioned/unique filenames.

Instead of:

```text
avatar/user-123.jpg
```

use:

```text
avatar/user-123/a81f3.jpg
```

When the avatar changes:

```text
avatar/user-123/b92e7.jpg
```

Now the URL changes.

The old cached object can expire naturally.

This is often much easier than aggressively invalidating caches.

---

# 48. Image Processing

Suppose a user uploads:

```text
4000 × 4000 image
```

Do you really want to serve that huge image as a:

```text
50 × 50 avatar
```

Probably not.

You might generate:

```text
avatar-original.jpg
avatar-128.webp
avatar-512.webp
```

But image processing can be expensive.

Remember Chapter 14?

Perfect background job.

---

# 49. Upload + Queue Architecture

Now our chapters start connecting.

```text
Client
   ↓
S3 Upload
   ↓
Upload Event
   ↓
Queue
   ↓
Image Worker
   ↓
Resize / optimize
   ↓
S3
```

The user doesn't need to wait while the backend generates five image sizes.

That's exactly what background jobs are for.

---

# 50. Video Processing

Video is even more interesting.

User uploads:

```text
video.mp4
```

You might need:

```text
Transcoding
Thumbnails
Different resolutions
Compression
Metadata extraction
```

So:

```text
Upload
   ↓
Object Storage
   ↓
Queue
   ↓
Video Processing Workers
   ↓
360p
720p
1080p
thumbnail
```

Now you can see why large systems separate uploads from processing.

---

# 51. Virus and Malware Scanning

If your application accepts arbitrary user files, you need to think about malicious uploads.

For example:

```text
resume.pdf
```

might not be as harmless as it looks.

A more defensive pipeline could be:

```text
Upload
   ↓
Quarantine Storage
   ↓
Security Scan
   ↓
Valid?
 ┌─┴─┐
Yes  No
 ↓    ↓
Ready Reject/Delete
```

Don't immediately make every uploaded file publicly downloadable before validation when the threat model requires scanning.

---

# 52. Don't Execute Uploaded Files

This sounds obvious, but it's worth saying.

Suppose someone uploads:

```text
something.js
```

or:

```text
something.php
```

or another executable format.

Don't store user uploads somewhere your application server might execute them as application code.

User-generated content should be treated as untrusted data.

---

# 53. Path Traversal

Imagine your upload API trusts the filename:

```text
../../../something
```

If you're writing directly to a filesystem using user-controlled paths, this can become dangerous.

Never blindly construct filesystem paths from user input.

Use:

```text
Generated IDs
Safe filenames
Controlled directories
```

instead.

This type of attack is called:

> **Path traversal**

---

# 54. File Name Sanitization

Suppose the uploaded filename is:

```text
../../../../../weird-file
```

or contains:

```text
special characters
control characters
very long strings
```

Your system should not blindly use that value as the storage path.

Store the original filename only when useful for display.

Generate your own safe storage identifier.

---

# 55. Upload Authorization

Imagine:

```http
POST /users/123/avatar
```

User 456 sends the request.

Should they be allowed to change user 123's avatar?

No.

So:

```text
Upload request
   ↓
Authentication
   ↓
Authorization
   ↓
Generate upload permission
```

Presigned URLs don't replace authorization.

Your backend should decide whether the user is allowed to request the upload in the first place.

---

# 56. File Ownership

Your database might contain:

```text
files

id
owner_id
storage_key
original_name
mime_type
size
status
created_at
```

Now when someone requests:

```text
GET /files/abc123
```

you can check:

```text
Who owns this file?

Is the requesting user allowed to access it?
```

Then generate temporary access if appropriate.

---

# 57. File Deletion

Deleting the database row isn't enough if the object still exists in storage.

You might need:

```text
Delete request
    ↓
Authorize
    ↓
Delete database record
    ↓
Delete object from S3
```

But what if:

```text
Database deletion succeeds
```

and:

```text
S3 deletion fails?
```

Now we have another distributed-system consistency problem.

---

# 58. Background Deletion

For some applications, you might:

```text
Mark file as deleted
        ↓
Queue deletion job
        ↓
Worker deletes from storage
        ↓
Cleanup metadata
```

This gives you retries.

Again, Chapter 14 appears.

Background jobs aren't just for emails.

---

# 59. Orphaned Files

Imagine:

```text
Upload to S3
   ↓
successful
```

Then:

```text
Database insert
   ↓
fails
```

Now the file exists in S3 but nothing references it.

That's an:

> **Orphaned file/object**

You need to think about cleanup.

For example:

```text
Temporary upload prefix
Lifecycle rules
Periodic cleanup job
Upload status records
```

Distributed systems rarely give you one perfect atomic transaction across:

```text
PostgreSQL
+
S3
```

So design for partial failure.

---

# 60. The Opposite Problem

Imagine:

```text
Database record
   ↓
created
```

but:

```text
S3 upload
   ↓
never happens
```

Now PostgreSQL says:

```text
file exists
```

but the actual object doesn't.

That's why a status model like:

```text
pending
uploaded
processing
ready
failed
```

can be useful.

---

# 61. Object Storage Lifecycle

Not every file needs to live forever.

Imagine:

```text
Temporary exports
Old logs
Expired reports
Abandoned uploads
```

Object-storage systems can support lifecycle policies.

Conceptually:

```text
Temporary file
   ↓
30 days
   ↓
Delete automatically
```

Or older files may move to cheaper storage tiers depending on the use case.

This helps manage storage costs.

---

# 62. Storage Costs

File storage isn't free.

You may pay for things such as:

```text
Storage capacity
Requests
Data transfer
CDN traffic
Processing
```

Imagine:

```text
1 million users
×
100 MB
```

That's:

```text
100 TB
```

of data.

Suddenly storage architecture becomes a business problem too.

Good backend engineering considers cost.

---

# 63. Don't Return File Bytes Through JSON

Avoid doing something like:

```json
{
  "file": "massive-base64-string..."
}
```

for ordinary large-file delivery.

Usually return:

```json
{
  "fileUrl": "..."
}
```

or an identifier that the client can use to request authorized access.

Again, exact design depends on the application.

---

# 64. Streaming

Sometimes your backend does need to proxy a file.

Don't necessarily load the entire file into memory first.

Bad mental model:

```text
500 MB file
   ↓
Load entire thing into RAM
   ↓
Send
```

Better:

```text
File source
   ↓
small chunks
   ↓
Response stream
```

This is called:

> **Streaming**

Data can move progressively instead of requiring the entire file to sit in memory.

---

# 65. Why Streaming Matters

Imagine:

```text
100 users
```

each downloading:

```text
500 MB
```

If your backend loads every complete file into memory:

```text
100 × 500 MB
=
50 GB
```

That's obviously problematic.

Streaming allows data to flow through without holding the whole file in memory.

---

# 66. Backpressure Appears Again

Remember Chapter 14?

Backpressure isn't only about queues.

Suppose:

```text
S3
 ↓
Backend
 ↓
Slow client
```

S3 can provide data faster than the client can consume it.

Streaming systems need a way to avoid endlessly buffering data in memory.

Node.js streams have mechanisms for handling this kind of backpressure.

We'll go deeper into Node.js streams in another context if needed.

The important concept:

> **Don't produce data infinitely faster than the consumer can handle it.**

---

# 67. Resumable Downloads

Large downloads can also fail.

If someone downloads:

```text
10 GB
```

and the connection fails after:

```text
9 GB
```

starting from zero isn't ideal.

HTTP supports:

```text
Range requests
```

which can allow clients to request only a portion of a file.

For example:

```http
Range: bytes=1000000-
```

This is especially useful for:

```text
Large downloads
Video streaming
Resume support
```

Object storage and CDNs can often handle this efficiently.

---

# 68. File Upload Architecture — Small App

For a simple project:

```text
Client
   ↓
Node.js
   ↓
Multer
   ↓
Local Disk
```

That's okay.

You're learning.

Don't build Amazon S3 infrastructure for a tiny weekend project just because it's "production-like."

---

# 69. File Upload Architecture — Growing App

As your application grows:

```text
Client
   ↓
Backend
   ↓
S3
```

Backend handles:

```text
Authentication
Authorization
Validation
Metadata
```

S3 handles:

```text
File storage
```

---

# 70. File Upload Architecture — Larger App

For larger files:

```text
                     ┌───────────────┐
                     │    Client     │
                     └──────┬────────┘
                            │
                 Request upload permission
                            ↓
                     ┌───────────────┐
                     │ API Server    │
                     └──────┬────────┘
                            │
                    Presigned URL
                            ↓
                         Client
                            │
                            │ file bytes
                            ↓
                           S3
                            │
                            ↓
                      Upload Event
                            │
                            ↓
                          Queue
                            │
                            ↓
                         Worker
                            │
                  ┌─────────┼──────────┐
                  ↓         ↓          ↓
               Validate   Resize      Scan
                  │         │          │
                  └─────────┼──────────┘
                            ↓
                           S3
                            │
                            ↓
                           CDN
                            │
                            ↓
                          Users
```

Now we have a serious upload pipeline.

---

# 71. Notice How Everything Connects

Look at the chapters we've learned.

### HTTP

Used for:

```text
Upload requests
Download requests
Presigned URL requests
```

### Authentication

Used to determine:

```text
Who is uploading?
```

### Authorization

Used to determine:

```text
Are they allowed to upload/download this?
```

### PostgreSQL

Stores:

```text
File metadata
Ownership
Status
```

### Redis

May help with:

```text
Caching
Temporary state
Rate limiting
```

### Queues

Handle:

```text
Image processing
Video processing
Virus scanning
Cleanup
```

### WebSockets

Can notify:

```text
"Your video finished processing."
```

This is why we're learning backend concepts in this order.

They aren't isolated topics.

They combine into systems.

---

# 72. Common Mistakes

## Mistake 1 — Storing every upload on the API server

Works locally.

Becomes difficult with multiple servers and ephemeral deployments.

---

## Mistake 2 — Sending huge files through the backend unnecessarily

For large files, direct object-storage uploads can reduce API-server load.

---

## Mistake 3 — Giving AWS credentials to the frontend

Never expose permanent server credentials to the client.

Use limited temporary mechanisms such as presigned URLs.

---

## Mistake 4 — Trusting file extensions

```text
photo.jpg
```

doesn't prove the file is actually JPEG.

---

## Mistake 5 — Trusting MIME type blindly

Client-controlled metadata can be spoofed.

---

## Mistake 6 — No size limits

A malicious or accidental giant upload can consume server resources.

---

## Mistake 7 — Using original filenames as storage keys

Names can collide and contain unsafe input.

Generate your own storage keys.

---

## Mistake 8 — Making private uploads public

Always think about file authorization.

---

## Mistake 9 — Loading huge files entirely into memory

Use streaming/direct uploads when appropriate.

---

## Mistake 10 — Assuming upload success because a URL was generated

Presigning doesn't mean the upload completed.

---

## Mistake 11 — Ignoring orphaned files

Partial failures can leave unused objects in storage.

---

## Mistake 12 — Forgetting cleanup

Temporary and abandoned uploads can accumulate forever.

---

# 73. Interview Questions

## 1. How are files normally uploaded through HTTP?

A common approach is `multipart/form-data`, which allows a request body to contain multiple parts such as normal fields and binary file data.

---

## 2. What is Multer?

Multer is Express middleware commonly used to process `multipart/form-data` uploads.

---

## 3. Why shouldn't we store user uploads on the application server's local disk?

Because application instances may be replaced or scaled horizontally, and local disks aren't automatically shared between servers.

---

## 4. What is object storage?

A storage model designed around storing objects identified by keys along with their data and metadata.

Amazon S3 is a common example.

---

## 5. What should PostgreSQL store about a file?

Often metadata such as:

```text
owner
storage key
filename
size
type
status
timestamps
```

while the actual file bytes live in object storage.

---

## 6. What is a presigned URL?

A temporary signed URL that grants limited permission to perform a specific operation on an object without exposing permanent storage credentials to the client.

---

## 7. Why use presigned uploads?

They allow clients to upload directly to object storage, reducing the amount of file traffic passing through application servers.

---

## 8. Why shouldn't we trust file extensions?

Because filenames are client-controlled and can be changed.

The extension doesn't prove the actual file content.

---

## 9. What is a CDN?

A distributed network of edge locations that can serve/cache content closer to users, reducing latency and origin load.

---

## 10. What is streaming?

Processing or transferring data incrementally rather than loading the entire object into memory first.

---

## 11. What is multipart upload?

For object storage, multipart upload means splitting a large object into parts that can be uploaded separately and later assembled.

Don't confuse this with HTTP `multipart/form-data`.

---

## 12. How would you handle large file uploads?

A common approach is:

```text
Client
 ↓
Request authorization
 ↓
Backend generates limited upload permission
 ↓
Client uploads directly to object storage
 ↓
Multipart upload for very large files
 ↓
Background processing if needed
```

---

# 74. Interview Scenario

> Your application lets users upload 2 GB videos. Currently every video passes through your Node.js server before being uploaded to S3. CPU looks okay, but your servers are struggling with bandwidth and long-running connections. What would you change?

Don't immediately say:

> "Add more servers."

Look at the architecture:

```text
Client
 ↓
2 GB
 ↓
Node.js
 ↓
2 GB
 ↓
S3
```

Your backend is acting as a data-transfer middleman.

A better design might be:

```text
Client
 ↓
Request upload permission
 ↓
Backend
 ↓
Presigned URL
 ↓
Client
 ↓
S3
```

Now the heavy file transfer bypasses your API server.

---

# 75. Interview Scenario

> Users sometimes see another user's uploaded document. What would you investigate?

Think:

```text
Storage keys
Authorization
Caching
Database ownership
Presigned URL generation
Public bucket/object permissions
```

Maybe both files were stored under:

```text
resume.pdf
```

Maybe authorization isn't checking:

```text
owner_id
```

Maybe the object is public.

Maybe a shared cache key is wrong.

Don't jump straight to one answer.

Trace the entire access path.

---

# 76. Interview Scenario

> A user uploads `profile.jpg`, but it's actually a different kind of file. How would you protect the system?

Don't trust:

```text
filename
```

or only:

```text
Content-Type
```

Use appropriate validation of actual content/file signatures where needed.

Also apply:

```text
Size limits
Allowed formats
Safe storage
Scanning when required
Generated filenames
```

And never execute user-generated files.

---

# 77. Interview Scenario

> Your database contains 100,000 file records, but S3 contains 120,000 objects. Why?

You probably have orphaned objects.

For example:

```text
Upload succeeds
       ↓
Database operation fails
```

or users abandoned uploads.

You might need:

```text
Upload lifecycle states
Temporary object prefixes
Scheduled cleanup
Storage lifecycle rules
Reconciliation jobs
```

This is a good example of distributed systems creating partial states.

---

# 78. Interview Scenario

> Users upload profile images, but your application loads very slowly because every image is 8 MB. What would you do?

Don't only say:

> "Use CDN."

First ask why you're serving an 8 MB image for a tiny avatar.

A better pipeline:

```text
Original Upload
      ↓
Queue
      ↓
Image Worker
      ↓
Resize
Compress
Convert format if appropriate
      ↓
Object Storage
      ↓
CDN
```

Then the client receives an appropriately sized asset.

CDN improves delivery.

Image optimization reduces what needs to be delivered in the first place.

---

# 79. Interview Scenario

> A user uploads a 10 GB video. The upload reaches 9 GB and the connection fails. How would you improve the experience?

Use a resumable/multipart upload strategy.

Instead of:

```text
10 GB
 ↓
one request
```

use:

```text
10 GB
 ↓
Part 1 ✓
Part 2 ✓
Part 3 ✓
Part 4 ✗
```

Retry:

```text
Part 4
```

instead of restarting the entire upload.

---

# 80. Mini Project — Production-Style Image Upload

Let's build this properly.

Create:

```http
POST /api/v1/uploads/presign
```

The endpoint should:

```text
1. Authenticate user
2. Validate requested file metadata
3. Check file size/type rules
4. Generate safe unique storage key
5. Generate presigned upload URL
6. Return URL + file key
```

---

# 81. Upload From the Client

Client receives:

```json
{
  "uploadUrl": "...",
  "fileKey": "avatars/user-123/abc123.jpg"
}
```

Then:

```text
Browser
   ↓
PUT file
   ↓
S3
```

The file doesn't pass through your API server.

---

# 82. Store Metadata

Create a database record:

```text
files
────────────────────────
id
owner_id
storage_key
original_name
mime_type
size
status
created_at
```

Start with:

```text
status = pending
```

After successful validation/processing:

```text
status = ready
```

---

# 83. Add Image Processing

After upload:

```text
S3
 ↓
Queue
 ↓
Worker
```

Worker:

```text
Validate
 ↓
Resize
 ↓
Optimize
 ↓
Store processed image
```

For example:

```text
avatars/user-123/original.jpg

avatars/user-123/128.webp

avatars/user-123/512.webp
```

---

# 84. Add Real-Time Notification

Remember Chapter 15?

When processing finishes:

```text
Worker
 ↓
Update database
 ↓
Publish event
 ↓
WebSocket
 ↓
Client
```

Browser receives:

```json
{
  "type": "avatar_ready",
  "fileId": "abc123"
}
```

Now your chapters are working together.

---

# 85. Add Cleanup

Create a scheduled background job:

```text
Find:

status = pending
AND
created_at < 24 hours ago
```

Then clean up abandoned uploads.

Now you're thinking about the entire file lifecycle.

Not just:

```text
upload()
```

---

# 86. The Architecture You Should Understand

By the end of this chapter, you should understand this:

```text
                      Client
                        │
                        │ 1. Request upload
                        ↓
                  ┌────────────┐
                  │ API Server │
                  └─────┬──────┘
                        │
                        │ 2. Presigned URL
                        ↓
                      Client
                        │
                        │ 3. File bytes
                        ↓
                  ┌────────────┐
                  │     S3     │
                  └─────┬──────┘
                        │
                        │ Upload event
                        ↓
                     Queue
                        │
                        ↓
                     Worker
                  /      |      \
                 ↓       ↓       ↓
             Validate  Resize   Scan
                 \       |       /
                  \      |      /
                       S3
                        │
                        ↓
                       CDN
                        │
                        ↓
                      Users
```

Meanwhile:

```text
PostgreSQL

stores:
↓
ownership
metadata
status
permissions
relationships
```

Each technology has a responsibility.

That's the important part.

---

# 87. The Mental Model

When someone says:

> "We need file uploads."

Don't immediately think:

```text
npm install multer
```

Think:

```text
What kind of file?

How large can it be?

Who can upload it?

Who can download it?

Where should it be stored?

Does it need processing?

Can it contain malicious content?

Should it be public or private?

What happens if upload fails halfway?

What happens if processing fails?

What happens if the DB write succeeds but storage fails?

What happens if storage succeeds but DB write fails?

How long should the file exist?

How will users around the world download it efficiently?
```

Then choose the architecture.

For a tiny app:

```text
Client
 ↓
Node.js
 ↓
Local storage
```

might genuinely be enough.

For a production application:

```text
Client
 ↓
Backend authorization
 ↓
Presigned URL
 ↓
Object Storage
 ↓
Background processing
 ↓
CDN
```

might make much more sense.

The point isn't to always build the complicated version.

The point is to understand **why the complicated version exists.**

---

# 88. What You Should Know After Chapter 16

You should now be able to explain:

```text
✓ What a file looks like to a backend
✓ multipart/form-data
✓ Multer
✓ File size limits
✓ File validation
✓ MIME types
✓ Why filenames can't be trusted
✓ Local storage problems
✓ Object storage
✓ Amazon S3
✓ Buckets
✓ Object keys
✓ File metadata
✓ Presigned URLs
✓ Direct uploads
✓ Large file uploads
✓ Multipart uploads
✓ Streaming
✓ Range requests
✓ CDN
✓ Private vs public files
✓ Upload authorization
✓ File ownership
✓ Image/video processing
✓ Malware scanning
✓ Orphaned objects
✓ Upload lifecycle
✓ Cleanup strategies
```

But more importantly, you should be able to answer:

> **"Design a file-upload system for a production application."**

with something better than:

> "I'll use Multer."

You should be able to reason:

```text
Small file?
→ Backend upload may be fine.

Large file?
→ Consider direct object-storage upload.

Private file?
→ Authenticate + authorize access.

Large download?
→ Object storage/CDN + streaming/range support.

Needs processing?
→ Queue + worker.

User-generated content?
→ Validate and treat as untrusted.

Multiple servers?
→ Don't depend on one server's local disk.

Global users?
→ Consider CDN.

Upload failed halfway?
→ Multipart/resumable upload.

Database and storage disagree?
→ Status + cleanup/reconciliation.
```

That's backend engineering.

Not memorizing which npm package handles uploads.

Understanding **where the bytes should go, who should be allowed to move them, and what happens when something fails.**

---

# Next — Chapter 17: Logging, Monitoring & Error Handling

Our backend now has:

```text
API Server
PostgreSQL
Redis
Queues
Workers
WebSockets
Object Storage
CDN
External services
```

Which creates a new problem.

Imagine a user tells you:

> "Your app is broken."

You open your laptop.

Everything works perfectly.

Then another user says:

> "Checkout took 8 seconds yesterday around 11 PM."

And another:

> "I uploaded my image, but it never finished processing."

How do you figure out what actually happened?

You can't debug production with:

```js
console.log("here");
console.log("here2");
console.log("why is this not working");
```

anymore.

We need to understand what our system is doing when we're not looking at it.

That's where we'll learn:

```text
Logs
Structured logging
Log levels
Request IDs
Correlation IDs
Metrics
Monitoring
Alerts
Tracing
Error tracking
Health checks
P50 / P95 / P99 latency
SLIs / SLOs
Production debugging
```

And we'll answer one of my favorite backend interview scenarios:

> **"Your API normally responds in 80ms. P99 suddenly jumps to 2 seconds, but CPU and memory look normal. How would you investigate it?"**