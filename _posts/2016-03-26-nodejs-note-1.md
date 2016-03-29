---
layout: post
title: "Server-side Development Using NodeJS (1)"
comments: True
---

This is my note for the NodeJS class offered by HKUST on Coursera.

## Two Salient Features of JavaScript

- First-class functions: A function can be treated the same as any other
variable.
- Closures:
    - A function defined inside another function has access to all the 
    variables declared in the outer function scope.
    - The inner function will continue to have access to the variables 
    from the outer scope even after the outer function has returned.

## The Callback Function

- Node has this convention: the first parameter is `error` for a callback
function

```javascript
function(err, someFunction)
```

An example,

`rectange.js` module:

```javascript
module.exports = function(x, y, callback) {
  try {
    if (x < 0 || y < 0) {
      throw new Error('Rectangle dimensions should be greater than zero:' +
        'l = ' + x + ', and b = ' + y);
    } else {
      callback(null, {
        perimeter: function () {
          return 2 * (x + y);
        },
        area: function () {
          return x * y;
        }
      });
    }
  } catch (error) {
    callback(error, null);
  }
};
```

`solve.js` calls it,

```javascript
var rect = require('./rectangle-2');

function solveRect(l, b) {
  console.log('Solving for rectangle with l = ' + l + ' and b = ' + b);
  
  // The 3rd param is a callback function
  rect(l, b, function(err, rectangle) {
    if (err) {
      console.log(err);
    } else {
      console.log('The area is ' + rectangle.area());
      console.log('The perimeter is ' + rectangle.perimeter());
    }
  });
}

solveRect(2, 4);
solveRect(3, 5);
solveRect(-3, 5);
```

The output:

```javascript
Solving for rectangle with l = 2 and b = 4
The area is 8
The perimeter is 12
Solving for rectangle with l = 3 and b = 5
The area is 15
The perimeter is 16
Solving for rectangle with l = -3 and b = 5
[Error: Rectangle dimensions should be greater than zero:l = -3, and b = 5]
```

## The `yargs` Module

To install `yargs` locally, do `npm install yargs --save`.

Add this at the top of `solve.js`

```javascript
var argv = require('yargs')
  .usage('Usage: node $0 --l=[num] --b=[num]')
  .demand(['l', 'b'])
  .argv;
```

And add `solveRect(argv.l, argv.b);` at the bottom.

To supply the parameters in command line, for example,

```
node solve --l=2 --b=3
```

## HTTP

HTTP request message:

- Request line
    - Method: GET, POST, PUT, DELETE
    - URL
    - Version of HTTP
- Headers
    - (Multiple) header field name: Value
- Blank line
- Body: content

For example,

Request line
```
GET /index.html HTTP/1.1
```

Headers
```
host: localhost:3000
connection: keep-alive
user-agent: Mozilla/5.0
accept-encoding: gzip, deflate, sdch
```

The HTTP response could be

```
HTTP/1.1 200 OK
Connection: keep-alive
Content-Type: text/html
Date: Sun, 21 Feb 2016 06:01:43 GMT
Transfer-Encoding: chunked

<html>...</html>
```

Response codes include

```
200: OK
201: Created
301: Moved Permanently
304: Not Modified
400: Bad Request
401: Unauthorized
403: Forbidden
404: Not Found
422: Unprocessable Entry
500: Internal Server Error
505: HTTP Version Not Supported
```

## Node and HTTP Module

- Using the module: `var http = require('http');`
- Creating a server: `var server = http.createServer(function(req, res) {...});`
- Starting the server: `server.listen(port, ...);`
- Incoming request message info available thru the first parameter `req`
    - `req.headers`, `req.body`
- Response message is constructed on the second parameter `res`
    - `res.setHeader("Content-Type", "text/html");`
    - `res.statusCode = 200;`
    - `res.writeHead(200, {'Content-Type': 'text/html'});`
    - `res.write('Hello World!');`
    - `res.end('<html><body><h1>Hello World</h1></body></html>');`, this
    will finalize `res` and send out the message.
- Using `path` module (specific to OS): `var path = require('path');`
    - `path.resolve('./public' + fileUrl)` gives the absolute path of the file
    - `path.extname(filePath)` gives the extension of that file
- Using `fs` module: `var fs = require('fs');`
    - `fs.exists(filePath, function(existsVar) {...});`, gives `true` or `false`,
    the second parameter is a callback function.
    - `fs.createReadStream(filePath).pipe(res);`, reads from the file and pipe
    the value into the response.


## Using `Express` Module to Build a Web Server

- What is `Express`: Fast, unopinionated, minimalist web framework for Node.js
- `Express` middleware provides a lot of plug-in functionalities that can be
used within the Express application
- Example: `morgan` for logging

```javascript
var morgan = require('morgan');
app.use(morgan('dev'));
```

- Serving static web resources

```javascript
app.use(express.static(__dirname + '/public'));
```
Note: `__filename` and `__dirname` give the full path to the file and directory
for the current module.

## Intro to REST - Representational State Transfer

- What is a web service: A system designed to support interoperability of systems
connected over a network
  - Service Oriented Architecture (SOA)
  - A standardized way of integrating web-based applications using open standards
  operating over Internet
- Two common approaches used in practice
  - SOAP (Simple Object Access Protocol) based services
    - Uses WSDL (Web Services Description Language)
    - XML based
    - More features than REST. More suitable for distributed applications.
  - REST (Representational State Transfer)
    - Use web standards
    - Exchange of data using either XML or JSON
    - Simpler compared to SOAP, WSDL etc. But more frequently used.
- REST
  - A style of software architecture for distributed hypermedia systems such as
  the World Wide Web
  - Introduced in the doctoral dissertation of Roy Fielding, one of the principal
  authors of the HTTP specification.
  - A collection of network architecture principles which outline how resources
  are defined and addressed.
  - Four basic design principles
    - Use HTTP methods explicitly
    - Be stateless
    - Expose directory structure-like URIs
    - Transfer using XML, JSON or both
- REST and HTTP
  - The motivation for REST was to capture the characteristics of the Web
    - URI (Uniform Resource Indicator) addressable resources
    - HTTP protocol
    - Make a Request -> Receive Response -> Display Response
  - Exploits the use of the HTTP protocol beyond POST and GET
    - PUT, DELETE
    - Preserve idempotence
- Nouns, Verbs, and Representations
  - Nouns: resources, unconstrained, e.g. http://www.conFusion.food/dishes/123
  - Verbs: constrained, GET, PUT, POST, DELETE
  - Representations: constrained, XML, JSON
- Resources
  - Any forms of data
  - URI, path like (tree) structure
- Verbs <--> CRUD
  - HTTP GET <--> READ
  - HTTP POST <--> CREATE
  - HTTP PUT <--> UPDATE
  - HTTP DELETE <--> DELETE
- Stateless server: every request is a new request from the client
  - Client side should track its own state, e.g. cookies

## Express Router

- Express Application Routes
- Identify an endpoint using URI
- Apply the verb on the URI
- Express supports this thru `app.all`, `app.get`, `app.post`, `app.put`,
`app.delete`

```javascript
app.get('/dishes/:dishId', function (req, res, next) {
  res.end('Will send details of the dish' + req.params.dishId + ' to you!')
});
```

- Body parser: Middleware to parse the body of the message, populates the
`req.body` property.

```javascript
var bodyParser = require('body-parser');
// Parse the JSON in the body
app.use(bodyParser.json());
```

- Express Router creates a mini-Express application

```javascript
var dishRouter = express.Router();
dishRouter.use(bodyParser.json());

dishRouter.route('/')
  .all(...);
  .get(...);
  ...
```

## Setting up a REST API: Week 1 Assignment
