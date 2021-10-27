# qewd-howtos

This repository contains documents with general howtos on QEWD-Up/QEWD.js server setups.

## Getting started with QEWD-Up + NuxtJS demo front-end (TL;DR)

If you want to try out QEWD-Up quickly, follow these instructions (in this order):
- [QEWD-Up basic demo server setup](QEWD-Up-demo.md) with InterSystems IRIS (works with NuxtJS tutorial examples below)
- [NuxtJS 2.x demo](Nuxt2-demo.md) (the current active version)
  
  -- or --

- [NuxtJS 3 demo](Nuxt3-demo.md) (the new beta version)

## QEWD-Up server setups

- [QEWD-Up basic demo server setup](QEWD-Up-demo.md) with InterSystems IRIS (works with NuxtJS tutorial examples below)
- [QEWD-Up Conduit example server setup](https://github.com/robtweed/qewd-conduit) implementing the complete [RealWorld Conduit back-end specification](https://github.com/gothinkster/realworld#backends) - with both REST and WebSocket endpoints.

## Building applications with NuxtJS front-end

For multi-page web apps, NuxtJS is a Node.js server to build very powerful applications using Vue.js in-page. Using the QEWD-Up demo server setup above as back-end, try out these tutorial examples using QEWD's WebSockets client:

- [NuxtJS 2.x demo](Nuxt2-demo.md)
- [NuxtJS 3 demo](Nuxt3-demo.md) (using the new beta version)

This framework integration provides a seamless connection to your IRIS/Caché database.

## Building applications with Vue.js front-end

- [Vue 2.x hello world example](https://github.com/wdbacker/vue2-qewd-hello-world)
- [Vue 3.x hello world example](https://github.com/wdbacker/vue3-qewd-hello-world)
- [Vue 2.x RealWorld Conduit example](https://github.com/wdbacker/vue-realworld-example-app) with [REST back-end](VueRealWorldConduit.md)
- [Vue 2.x RealWorld Conduit example](https://github.com/wdbacker/vue-realworld-example-app) with [WebSocket back-end](VueRealWorldConduitWS.md)

## Building applications with React front-end

- [React Redux RealWorld Conduit example](https://github.com/wdbacker/react-redux-realworld-example-app) with [REST back-end](ReactRealWorldConduit.md)

## Building applications with Angular front-end

- [Angular RealWorld Conduit example](https://github.com/wdbacker/angular-realworld-example-app) with [REST back-end](AngularRealWorldConduit.md)

## Custom configurations for your QEWD-Up/QEWD.js server

- [setting up your public QEWD-Up/QEWD.js server behind a reverse proxy](ReverseProxy.md)
- starting all workers at QEWD-Up startup with the poolPrefork config option

## Developing with QEWD-Up

- [Standalone interactive WebSocket-based CRUD example](https://github.com/robtweed/qewd-microservices-examples/blob/master/WINDOWS-IRIS-2.md)
- [Standalone microservices example](https://github.com/robtweed/qewd-microservices-examples/blob/master/WINDOWS-IRIS-1.md)
- [Debugging your QEWD-Up server in VSCode](Debugging.md)
- Error handling in QEWD's master process and workers (TODO)
- Utility methods for function calls and returning responses (TODO)
