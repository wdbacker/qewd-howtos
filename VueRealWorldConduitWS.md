# Vue RealWorld example app - WebSocket version

This is the interactive app version of the [Vue RealWorld example app using a QEWD.js REST back-end](VueRealWorldConduit.md).

## Adapting the app to use a QEWD-Up WebSockets back-end

The same app can also act as an interactive app version (using WebSockets to the QEWD.js back-end)

- in the cmd window, install the `qewd-client` module:
  ```
  C:\GitHub\vue-realworld-example-app>npm install wdbacker/qewd-client
  ```
- in `src/main.js`, add qewd-client:
  ```javascript
  ...
  // import QEWD from the qewd-client
  import { QEWD } from 'qewd-client'

  // add QEWD as a global property
  // now you can use QEWD in all your app components as `this.$qewd.reply(...)`
  Vue.prototype.$qewd = QEWD
  
  Vue.config.productionTip = false;
  ...
  ```
- in `src/App.vue`, add a `created()` method to start the QEWD.js websocket connection:
  ```javascript
  
  ```
- open `src/common/api.service.js`
  
... still in development :-)
