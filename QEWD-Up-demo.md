# QEWD-Up demo back-end (setup)

In this tutorial, we will setup a basic [QEWD-Up server](https://github.com/robtweed/qewd/tree/master/up#native-qewd-monolith) - native, not using Docker with an InterSystems IRIS (or Caché) data platform. You'll need to have this demo server up & running first to use it as a back-end server for the [NuxtJS 2](Nuxt2-demo.md) and [NuxtJS 3](Nuxt3-demo.md) demo applications.

## Install InterSystems IRIS or Caché

- to start, make sure you have the [InterSystems IRIS (or Caché) data platform](https://download.intersystems.com/download/register.csp) up & running on your development machine
- if you're a first time user, you can create an InterSystems developer account and download the [IRIS Community Edition](https://download.intersystems.com/download/login.csp)
- in System portal, go to `System Administration` -> `Security` -> `Services` and enable `%Service_CallIn` (unauthenticated)

## Install Node.js

- download & install [Node.js](https://nodejs.org/en/) (at least version 14.x)

## Install Visual Studio Code (VSC)

- install the latest [Visual Studio Code](https://code.visualstudio.com/download) (for Windows, pick the ``User Installer`` variant)

## Install & setup the QEWD-Up demo server

- create a GitHub working directory of your choice, e.g. ``C:\github``
- open VSC, clone the [QEWD-Up demo repository](https://github.com/wdbacker/qewd-up-demo) in VSC to your github working directory and confirm to open it
- in VSC, open an integrated terminal and install all required Node.js NPM modules using:
  ```
  # npm install
  ```
- in `configuration/config.json`, adjust the port and database connection settings to your local install, e.g. for IRIS:
  ```json
  "database": {
    "type": "dbx",
    "params": {
      "database": "IRIS",
      "path": "C:\\InterSystems\\IRIS\\mgr",
      "username": "_SYSTEM",
      "password": "SYS",
      "namespace": "USER"
    }
  }
  ```
  For Caché, change `database.params.database` above to `Cache` and in `database.params.path` the IRIS subpath to Cache

## Start the QEWD-Up demo server

- in VSC, open an integrated terminal and run:
  ```
  # node node_modules/qewd/up/run_native
  ```
- if you want to debug the QEWD-Up demo server in VSC, launch it from the debug tab (for details, refer to the [debugging howto](Debugging.md)).
  
  You can also use the "Attach" debug option in VSC, in this case launch QEWD-Up from the integrated terminal with `--inspect` option and let VSC attach to it:
  ```
  # node --inspect node_modules/qewd/up/run_native
  ```
- now test your QEWD-Up demo server using your browser by navigating to `http://localhost:8080/api/info` (adjust the port if you changed it in configuration)
- when you see a JSON response with server info, you're ready to go and start testing with the [NuxtJS 2](Nuxt2-demo.md) and [NuxtJS 3](Nuxt3-demo.md) front-end demo's!

## The QEWD-Up demo in more detail

We will explore now the basic inner workings of your QEWD-Up back-end in more detail: how do you define back-end handlers interacting with your database, what's the difference between WebSocket applications and REST api's.

### WebSocket applications

An interactive (WebSocket) application lives inside the `qewd-apps` directory of your QEWD-Up server. Each application has a unique *name* that maps to a corresponding directory inside `qewd-apps`.

The front-end in the browser (e.g. when you press a button or post a form) sends a (WebSocket) message to the QEWD-Up back-end. The handler you wrote for that message picks up the request, processes it in Node.js & IRIS and returns a response JSON object. The response is then returned to the front-end where the UI can be updated.

The directory structure for message handlers is:
```
qewd-apps
  nuxtjs-test   // application name
    test        // message type
      index.js  // message handler code (this file can also be named handler.js)
```

A message handler's structure is:
```javascript
module.exports = function(messageObj, session, send, finished) {
  // this handler picks up incoming text for testing purposes
  let incomingText = messageObj.params.text || ""
  // create a DocumentNode reference to the ^QEWDTest global in IRIS
  let qewdDemo = new this.documentStore.DocumentNode('QEWDDemo');
  // assign the text content to the subscript 'demo' in this global
  qewdDemo.$('demo').value = incomingText
  // put the complete message JSON object inside a second level subscript
  qewdDemo.$('demo').$('message').setDocument(messageObj);
  // read everything back in from IRIS into a JSON object
  let qewdDemoGlobal = qewdDemo.$('demo').getDocument(true);
  // return the response to the client using WebSockets (or Ajax mode)
  let d = new Date()
  finished({
    text: 'You sent: "' + incomingText + '" to QEWD-Up at ' + d.toUTCString(),
    qewdDemoGlobal
  })
}
```
In your message handler, you must export a function with these parameters and call the `finished` method when processing is complete.

In this example, you notice IRIS is used as a [documentStore](https://www.slideshare.net/robtweed/ewd-3-training-course-part-20-the-documentnode-object) in JS and globals become DocumentNode's in Node.js. You can create a global reference by defining a DocumentNode in JS (and you can do this at any subscript level!).

Accessing data at a certain subscript is done with the ``.$('subscript')`` selector syntax (chainable).

Setting/getting a value to/from a global node is done with the `.value` syntax.

Save a complete JSON object in IRIS using `setDocument()`.

Retrieve (a part of) your global content in JSON using `getDocument()`

As you see, you can interact directly in server-side JavaScript with your globals in your IRIS database at a very granular or higher level. If you want to delve deeper into all features of the QEWD-Up `documentStore`, follow the [documentation on QEWD-JSdb](https://github.com/robtweed/qewd-jsdb/blob/master/IRIS.md).

When you want to work with ``classes`` and ``SQL`` in IRIS, you can leverage all ObjectScript features in your message handlers using `this.db.call()` to invoke your ObjectScript code:
```javascript
// if you want to use IRIS Objects or SQL, just use an extrinsic function call:
module.exports = function(messageObj, session, send, finished) {
  let self = this
  // pass a parameter global to IRIS
  let params = {
    id: 1,
    nodejs: "hot",
    cache: "cool"
  }
  // call your extrinsic function in IRIS
  let response = this.db.call('testIRIS^jsQEWDDemo', params, self, session);
  // check the response object coming back from IRIS
  if (response.ok) {
    console.log("IRIS response was: ", response);
  }
  else {
    console.log("An error occurred: ", response.error);
  }
  // return the response to the front-end using WebSockets
  // we return some call data our ObjectScript code saved in the session 
  finished({
    response,
    sessionCallData: session.data.$('call').getDocument(true)
  });
}    
```
Btw, the `this.db.call` helper method is defined in `qewd-apps/nuxtjs-test/onLoad.js` (this file is loaded once when a QEWD-Up process starts an application).

The ``sessionCallData`` (params + json response of the call) is saved in the session by IRIS using the ObjectScript code below.

In IRIS, you can now write a `jsQEWDDemo.mac` routine you see in the JS handler:
```
jsQEWDDemo
 ;
 #Include %occStatus
 #Include jsHeader
 ;
testIRIS(sessid)
 New (sessid)
 
 Set error=""
 ;save the parameters of this call inside our session:
 Kill $$$session(sessid,"call","params")
 Merge $$$session(sessid,"call","params")=$$$jsData("params")
 ;first, let's try the Object way ...
 If $$$jsData("params","nodejs")="hot" {
   Set id = $Get($$$jsData("params","id")) If '$Length(id) Set error="person id missing in request" Quit
   Set person = ##class(User.Person).%OpenId(id) If '$ISOBJECT(person) Set error="person id does not exist" Quit
   Set $$$jsData("json","person","name") = person.name
   Set $$$jsData("json","person","city") = person.city
   Kill person
 }
 ;next, let's try the SQL way ...
 Set query = "SELECT * FROM Person"
 Set statement = ##class(%SQL.Statement).%New()
 Set status = statement.%Prepare(query)
 If 'status {
   Set error = ##class(%SYSTEM.Status).GetErrorText(status)
 }
 Else {
   Set n = 0, rset = statement.%Execute()
   While (rset.%Next()) {
     Set $$$jsData("json","persons",n,"name") = rset.%Get("name")
     Set $$$jsData("json","persons",n,"city") = rset.%Get("city")
     Set n = n + 1
   }
 }
 ;save the json result of this call inside our session:
 Kill $$$session(sessid,"call","json")
 Merge $$$session(sessid,"call","json")=$$$jsData("json")
 Quit error
 ;
```
As you notice, you can use all ObjectScript features in IRIS, receive & return complete JSON objects conveniently to & from IRIS using some macro's like ``$$$jsData(...)`` and `$$$session(sessid, ...)` (you'll find the macro code inside the `iris` directory of the `qewd-up-demo` repo).

### REST api's

REST api's are defined in `configuration/routes.json` and their handlers live in the `apis` directory of your QEWD-Up server.

The front-end in the browser (e.g. when you press a button or post a form) can send a (REST) request to the QEWD-Up back-end using the standard `fetch` method in JS or the `axios` module. The handler you wrote for that REST api endpoint picks up the request, processes it in Node.js & IRIS and returns a response JSON object. The response is then returned to the front-end where the UI can be updated.

As you'll see below, REST api handlers don't differ much from WebSocket handlers. The only major difference is that they are stateless.

You define all your REST endpoints in one JSON file: `configuration/routes.json`:
```json
[
  {
    "uri": "/api/info",
    "method": "GET",
    "handler": "getInfo"
  },
  {
    "uri": "/api/test",
    "method": "GET",
    "handler": "test"
  }
]
```
All your routes are defined in one place, making it easy to keep an overview. Each route has a uri, a http method and a handler.

The directory structure for REST api handlers is (there is no application level needed here as with WebSockets):
```
apis
  test        // endpoint handler name
    index.js  // endpoint handler code (this file can also be named handler.js)
```

A REST api handler's structure is:
```javascript
module.exports = function(args, finished) {
  // this handler picks up incoming text for testing purposes
  let incomingText = args.req.query.text || ""
  // create a DocumentNode reference to the ^QEWDTest global in IRIS
  let qewdDemo = new this.documentStore.DocumentNode('QEWDDemo');
  // assign the text content to the subscript 'demo' in this global
  qewdDemo.$('demo').value = incomingText
  // put the complete message JSON object inside a second level subscript
  qewdDemo.$('demo').$('request').setDocument(args.req);
  // read everything back in from IRIS into a JSON object
  let qewdDemoGlobal = qewdDemo.$('demo').getDocument(true);
  // return the response to the client using WebSockets (or Ajax mode)
  let d = new Date()
  finished({
    text: 'You sent: "' + incomingText + '" to QEWD-Up at ' + d.toUTCString(),
    qewdDemoGlobal
  })
}
```
In your api handler, you must export a function with these two parameters and call the `finished` method when processing is complete.

This QEWD-Up demo is very basic - it only shows a very small subset of it's features, like e.g.:
- very high speed, especially when using WebSockets (no REST request overhead)
- very thin layer to your JS code and IRIS, handling all the boilerplate
- creating very complex multi-server setups aggregating responses from all underlying servers
- linking multiple QEWD-Up instances using microservices
- dockerized instances
- ...

For detailed documentation on all features of QEWD-Up, read the [documentation](https://github.com/robtweed/qewd/tree/master/up) and the [advanced documentation](https://github.com/robtweed/qewd/tree/master/up/docs).

And last but not least ... writing applications using the QEWD-Up back-end on Node.js also leverages the power of all Node.js's [NPM modules](https://www.npmjs.com/) you can use now in your IRIS applications!