# Minimum Protocol Specification for Cyberland

## Server implementations

This are the minimum requirements for a server to work with most cyberland compatible clients. A server can implement as many extra features as it wants, and have as many boards as it wants.

* All content (with the exclusion of the plaintext pages / and /tut.txt) must be in JSON returned as Content-Type: application/json

### Reserved boards
These boards are reserved for potentially creating a federation network in the future:
* /s/ - Shared

### Required plain text pages

#### /
A home page MUST be in .txt format and reside  at the top (/), such that a get request to the domain of the server will respond with the home page, in .txt format.

The top 8 lines of the home page must be ASCII art representing the name of the instance. To generate this, you can use the program `figlet`, however the ASCII art can be created any way you like.
This is so that clients can display the correct ASCII art depending on the server they are connected to.

#### /tut.txt
The URI /tut.txt must return a plain text file specifying a minimal guide to interacting with the server. An example can be seen below:

```text
Tutorial
#######

* Posting
POST /o content=x&replyTo=y
 - This will create a post to the off topic board containing the content x and replying to y, if y is unspecified, then it will be considered that it does not reply to anything.
* Retrieving posts
GET cyberland.digital/o/?thread=x&num=y&offset=z&opsOnly=true
 - This will get y number of posts from the off topic board that reply to post number x with the newest first as a JSON object. If x is unspecified, just y number of recent posts will be returned. The number of posts you can recieve at once may be limited at some point depending on how this goes. Setting an offset z will offset the posts - for example, with num=2 and offset=1, out of [1,2,3,4,5] you'll receive posts 3 and 4. Set opsOnly to true in order to only receive top-level posts.
```

### HTTP codes
A server must respond with the following HTTP codes for the given conditions:
* Success - 200 OK
* User is on ban list - 403 Forbidden
* Rate limiting - 429 Too Many Requests
* No content provided - 204 No Content



### Get Requests
A plain get request to a given board with no given parameters must return valid JSON containing the 50 most recent posts to that board.

#### /boards
The server must respond to /boards with information in json format about the current boards on the server.
* slug - the fully qualified path of the board /*/, eg: /t/
* name - the long name of the board, eg: Technology
* charLimit - the character limit per post on the board
* posts - the total number of posts to the board at the time of the request

#### Posts
The JSON returned from a get request to a board must contain a list of posts with: 
* a string assigned to 'content' with the post's content
* an integer assigned to 'replyTo' with the post number which the post is replying to, 0 for OPs (referencing the board), 
* an integer assigned to 'bumpCount' with the number of replies the post has
* an string assigned to 'time' with the time at which the server received the post in UTC, in the format "YYYY-MM-DD hh:mm:ss"

#### Parameters
Get requests must support the following parameters:

* thread - Specifies a post number, must return the post specified and replies to that post. 
  - When thread = 0, the server must only return OP posts.

* num - Specifies number of posts to return

* offset - Number of results to skip before returning (EG: offset=3 must skip the first 3 posts it would respond with)

### Post requests
The server must accept post requests to board URIs with a minimum of the following form data:

* content - this should be the main content of the post.
* replyTo - This should specify the ID number of a previous post, or if not specified default to 0

