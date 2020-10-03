# Installing and running a QEWD-Up/QEWD.js server on a public website using a reverse proxy setup

Putting your QEWD-Up/QEWD.js server behind a reverse proxy is recommended because your (public) Apache or Nginx server allows you to control and secure your server much better when you expose it on the internet.

## The setup

In this example setup, we assume:
- your (public) hostname is `my-qewd-server.com`
- your QEWD-Up server is proxied on the `/qewd` path/location (assuming you are running multiple instances like e.g. an orchestrator instance)
- your QEWD-Up/QEWD.js server url then becomes: `http://my-qewd-server.com/qewd`
- your internal QEWD-Up/QEWD.js server ip (reachable from your public webserver) is running on host 192.168.100.27 and port 8080

In this setting, your public webserver acts as a reverse proxy for your QEWD-Up/QEWD.js server.

## Step 1: setup your QEWD-Up/QEWD.js server for use behind an Apache/Ngnix server acting as a reverse proxy

For websocket requests on a proxied path/location to work correctly on your QEWD-Up/QEWD.js server back-end, you need to use the `webSockets.io_paths` option inside the `config.json` file in your QEWD-Up/QEWD.js server setup:

```javascript
{
  "qewd": {
    "serverName": "My QEWD-Up server",
    "poolSize": 2,
    "port": 8080,
    "database": {
      "type": "dbx",
      "params": {
        ...
      }
    },
    "webSockets": {
      "io_paths": ["/qewd"]
    }
  }
}
```
You can specify in this parameter your proxy path(s)/location(s) (there can be more than one in case you'd use different proxy servers with a different path/location pointing to the same QEWD-Up/QEWD.js server).

In case you reverse proxy your QEWD-Up/QEWD.js server on the root domain level (e.g. at `http://my-qewd-server.com` without a `/qewd` path/location), the `webSockets.io_paths` parameter is not needed.

## Step 2: reverse proxy setup for Apache and Nginx

### Apache (version 2.4 and higher)

For Apache, first make sure you enabled the following httpd modules:
```
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
LoadModule rewrite_module modules/mod_rewrite.so
```

In your (virtual host) config, you need these settings:
```
<VirtualHost *:80>
  ServerAdmin support@stabe.be
  DocumentRoot "${SRVROOT}/htdocs/"
  DirectoryIndex index.php
  ServerName my-qewd-server.com
  LogLevel warn
  ErrorLog logs/my-qewd-server_error.log
  CustomLog logs/my-qewd-server_access.log combined
	
  # enable the runtime rewriting engine
  RewriteEngine on
  # when the socket.io client checks transport mode polling, pass it on over http protocol
  RewriteCond %{QUERY_STRING} transport=polling
  # in that case, rewrite the request url
  RewriteRule /qewd/socket.io/(.*)$ http://192.168.100.27:8080/socket.io/$1 [P]
  
  ProxyRequests off
  # when the client downloads the socket.io client, proxy it over http ...
  ProxyPass /qewd/socket.io/socket.io.js http://192.168.100.27:8080/socket.io/socket.io.js
  ProxyPassReverse /qewd/socket.io/socket.io.js http://192.168.100.27:8080/socket.io/socket.io.js
  # when the client communicates with the websocket, proxy it over ws ...
  ProxyPass /qewd/socket.io/ ws://192.168.100.27:8080/socket.io/
  ProxyPassReverse /qewd/socket.io/ ws://192.168.100.27:8080/socket.io/
  # proxy all other (page/asset) requests over http ...
  ProxyPass /qewd http://192.168.100.27:8080
  ProxyPassReverse /qewd http://192.168.100.27:8080
</VirtualHost>
```
If your QEWD-Up/QEWD.js server is using https, you need to change all `http:` protocols to `https:` and `ws:` to `wss:`
```
  RewriteEngine on
  # when the socket tries transport mode polling, pass it on with http protocol
  RewriteCond %{QUERY_STRING} transport=polling
  # in that case, rewrite the request url
  RewriteRule /qewd/socket.io/(.*)$ https://192.168.100.27:8080/socket.io/$1 [P]
  
  ProxyRequests off
  # when the client downloads the socket.io client, proxy it over http ...
  ProxyPass /qewd/socket.io/socket.io.js https://192.168.100.27:8080/socket.io/socket.io.js
  ProxyPassReverse /qewd/socket.io/socket.io.js https://192.168.100.27:8080/socket.io/socket.io.js
  # when the client communicates with the websocket, proxy it over ws ...
  ProxyPass /qewd/socket.io/ wss://192.168.100.27:8080/socket.io/
  ProxyPassReverse /qewd/socket.io/ wss://192.168.100.27:8080/socket.io/
  # proxy all other (page/asset) requests over http ...
  ProxyPass /qewd https://192.168.100.27:8080
  ProxyPassReverse /qewd https://192.168.100.27:8080
```
# Nginx

In your Nginx server config, you need these settings:
```
server {
  listen       80;
  server_name  my-qewd-server.com;

  location / {
      root   html;
      index  index.html index.htm;
  }

  # proxy all http (page/asset) requests over http to the internal QEWD-Up/QEWD.js server
  location /qewd/ {
    proxy_pass "http://192.168.100.27:8080/";
  }

  # proxy the socket.io client script and WebSocket messages over http & ws
  location /qewd/socket.io/ {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;

    proxy_pass "http://192.168.100.27:8080/socket.io/";

    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
}
```
### Download & install Nginx for Windows

- download the Nginx for Windows build
- unpack to `C:\nginx`
- edit de config file as above
- start, stop Nginx and get status:
  ```
  C:\nginx> start nginx
  C:\nginx> nginx -s stop
  C:\nginx> tasklist /fi "imagename eq nginx.exe"
  ```
## Example settings for a basic html single page app using qewd-client.js

Applying this proxy setup to the basic example that comes with [qewd-client](https://github.com/robtweed/qewd-client):
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Hello World Demo</title>
    <script type="module">
      import { QEWD } from './qewd-client.js';
      // adjust the path for the qewd-client module depending on where you've saved it

      document.addEventListener('DOMContentLoaded', function() {

        QEWD.on('ewd-registered', function() {

          // Your QEWD environment is ready

          // optionally turn on logging to the console
          QEWD.log = true;

          // now you can begin sending QEWD messages, eg:

          QEWD.send({
            type: 'hello world'
          }, function(responseObj) {
            console.log('response: ' + JSON.stringify(responseObj));
            alert(responseObj.message.error || responseObj.message.text);
          });

          // etc
        });

        // start the QEWD service and register your application
        QEWD.start({
          application: 'hello-world',
          // use a url to allow correct startup of WebSocket using a namespace /qewd
          // you *need* to pass a url here too to make the Websocket transport=websocket mode work 
          // (it will fall back to polling mode if the url is not passed in)
          url: 'http://my-qewd-server.com/qewd',
          // pass the io_path to the WebSocket
          io_path: '/qewd',
          // optional: control the transport mode(s) the websocket will use
          io_transports: [ 'polling', 'websocket' ]
        });

      });      
    </script>
  </head>
  <body>
  </body>
</html>
```
## Example settings for a Vue.js 3.x app using qewd-client.js

Applying this proxy setup to a [Vue.js 3.x hello-world](https://github.com/wdbacker/vue3-qewd-hello-world) app example, change the `start()` method call inside the `App`'s component `created()` method:
```javascript
this.$qewd.start({
  application: 'hello-world',
  io,
  //url: 'http://localhost:8080'
  // pass in the url of the external QEWD-Up/QEWD.js server
  url: 'http://my-qewd-server.com/qewd',
  // pass the io_path to the WebSocket
  io_path: '/qewd',
  // optional: control the transport mode(s) the websocket will use
  io_transports: [ 'polling', 'websocket' ]
})
```
An more extensive Vue 2.x example using vuex is available at the [Vue RealWorld Conduit app repository](https://github.com/wdbacker/vue-realworld-example-app).