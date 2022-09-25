# Node

Node.js® is a JavaScript runtime built on Chrome's **V8** JavaScript engine. Node.js's package system `npm` is the largest ecosystem of open source libraries in the world.
Before Node.js there was no way to use JavaScript outside of the browser to build web servers or cli tools or native apps. Node.js is not a programming language, It gives the environment to create web-servers and many more while still using Javascript.

main dependencies -

### libuv (c++)

manages event loop and thread pool

### V8 (c++,js)

converts our js to machine code

# Content types

### application/json

content type is a json object. We can use `express.json` to parse incoming requests.

### application/x-www-form-urlencoded

default when html form is submitted. We can use `express.urlencoded` to parse incoming requests. when extended is true then body is parsed using `**qs**` library, means nested properties can also be parsed.
`person[name] = 'cw'` is parsed to `{person: {name: cw }}`.

### multipart/form-data

To set this type when form is submitted we have to set `enctype` attribute on our form
it is same as the `x-www-form-urlencoded` but using this we can send files along side text fields.
we can use libraries like `**multer**`.

```html
<form enctype="multipart/form-data" action="/campgrounds/new" method="post" />
```

### text/plain

content type is text data. `express.text` middleware can be used to parse incoming requests.

# REPL

Read Eval Print Loop
which further means evaluating code on the go.

# REST (Representational State Transfer)

It's basically a set of guidelines for how a client + server should communicate and perform CRUD operations on given resource.

# Single threaded vs Multi threaded

### Multi threaded or blocking or synchronous

For every incoming request a new thread is created and assigned. Each respective thread is only handles that particular request, So if we have very large amount of requests then we can run out of resources in such case client has to wait until free threads are available or we need to add more hardware.

eg. ASP.net, Rails

### Single threaded or non-blocking or asynchronous

We have a single thread to handle all requests. If a request is arrived that thread is used to handle request. If we need to query the database our thread doesn't need to wait for the result instead our thread is used to serve another client. When the database returns the results it puts a message in Event Queue. Node monitors this queue in the background, when it finds event in this queue then it will take it out and process it.

ideal for network access and disk access application usages
not ideal for cpu intensive apps like video encoding or image manipulation service

Since node is single threaded when we perform complex calculations since performing calculations is blocking or synchronous in nature client has to wait until our single thread performs the calculation.
