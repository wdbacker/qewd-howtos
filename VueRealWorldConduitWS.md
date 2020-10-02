# Vue.js RealWorld example app (WebSocket version)

This is the interactive app version of the [Vue RealWorld example app using a QEWD-Up/QEWD.js REST back-end](VueRealWorldConduit.md), the app is just a fork from the example at [RealWorld](https://github.com/gothinkster/realworld#frontends). A ready-to-run version of this WebSocket version is available from [this repository](https://github.com/wdbacker/vue-realworld-example-app). A summary of changes is described below.

## <a name="setup"></a>Setup this WebSocket app version on your local machine

You can follow the [instructions in the REST version](VueRealWorldConduit.md#setup), except:
- you'll need to open the [WebSockets repository of the app](https://github.com/wdbacker/vue-realworld-example-app)
- fork this repository in VSCode with Ctrl+Shift+P, Git Clone and enter the [url](https://github.com/wdbacker/vue-realworld-example-app) to a directory of your choice, e.g. `C:\GitHub`
- the remaining setup instructions are the same as for the REST version

## Adapting the app to use a QEWD-Up/QEWD.js WebSockets back-end

The same app can also act as an interactive app version (using WebSockets to the QEWD.js back-end)

- in the cmd window, install the `qewd-client` module:
  ```
  C:\GitHub\vue-realworld-example-app>npm install wdbacker/qewd-client
  ```
- in `src/common/config.js`, add a `QEWD_URL` WebSocket endpoint (adjust the url's to your local ip address or hostname):
  ```javascript
  //export const API_URL = "https://conduit.productionready.io/api";
  export const API_URL = "http://qewd-conduit.local:8080/api";
  export default API_URL;
  export const QEWD_URL = "http://qewd-conduit.local:8080";
  ```
- in `src/main.js`, import the `qewd-client` module and add `$qewd` as a global Vue property:
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
- in `src/App.vue`, we import the `QEWDService` wrapper object for WebSockets (see below, similar to the `ApiService` for REST), add some reactive `data` options and a `created()` lifecycle hook to init the QEWD.js/QEWD-Up WebSocket connection:
  ```html
  <template>
    <div id="app">
      <RwvHeader v-if="qewdReady" />
      <router-view v-if="qewdReady"></router-view>
      <RwvFooter v-if="qewdReady" />
    </div>
  </template>

  <script>
  import RwvHeader from "@/components/TheHeader";
  import RwvFooter from "@/components/TheFooter";
  // import QEWDService into our App component
  import { QEWDService } from "@/common/api.service";

  export default {
    name: "App",
    components: {
      RwvHeader,
      RwvFooter
    },
    data () {
      return {
        // define a reactive property set to true when QEWD's websocket is registered
        qewdReady: false,
        // define a reactive property set to true when QEWD's websocket becomes unavailable
        qewdNotReachable: false
      }
    },
    created () {
      // init the WebSocket connection to the QEWD-Up back-end
      QEWDService.init(this)
    }
  };
  </script>
  ```
- in `src/common/api.service.js`, we added a `QEWDService` wrapper object to let the vuex api communicate either via REST api or via WebSocket api calls (we use deliberately a mix of both api's in this example to show the capabilities of the QEWD-Up back-end, which allows WebSocket and REST requests simultaneously):
  ```javascript
  import Vue from "vue";
  import axios from "axios";
  import VueAxios from "vue-axios";
  import JwtService from "@/common/jwt.service";
  // add a second QEWD_URL parameter for the WebSocket endpoint
  import { API_URL, QEWD_URL } from "@/common/config";
  // import the socket.io client into our App component
  import io from 'socket.io-client'

  const ApiService = {
    // ... all code stays unchanged here from the original example
  };

  export default ApiService;

  // add a similar service wrapper object here for the WebSocket communication
  export const QEWDService = {
    _vcc: null,

    // initialise + start the WebSocket communication using qewd-client
    init(vcc) {
      // keep the Vue (App) component context
      this._vcc = vcc
      /*
        create an event handler invoked when QEWD's connection is registered/ready
      */
      vcc.$qewd.on('ewd-registered', function() {
        // Your QEWD environment is ready, set the qewdReady data property to true
        vcc.qewdReady = true
        // optionally turn on logging to the console
        vcc.$qewd.log = true
      });
      vcc.$qewd.on('ewd-reregistered', function() {
        // Your QEWD server is available again, set the qewdNotReachable data property to false
        vcc.qewdNotReachable = false
      });
      vcc.$qewd.on('socketDisconnected', function() {
        // Your QEWD server is not available, set the qewdNotReachable data property to true
        vcc.qewdNotReachable = true
      });
      /*
        start QEWD.js with these required params:
        - application: QEWD's application name
        - io: the imported websocket client module
        - url: the url of your QEWD.js server

        *** important: by default, a Vue.js app will run it's development server on localhost:8080 
        (this is the same port as QEWD.js's default port 8080)
        you'll *need* to change the port to e.g. 8090 in QEWD.js's config
        to make it work with a Vue.js app!
      */
      vcc.$qewd.start({
        application: 'qewd-conduit',
        io,
        url: QEWD_URL
      })
    },

    // wrap the qewd-client reply() method of qewd-client to return a compatible response format with axios
    reply(messageObj) {
      let vcc = this._vcc
      return new Promise((resolve, reject) => {
        vcc.$qewd.send(messageObj, function(responseObj) {
          responseObj.data = responseObj.message ? responseObj.message : {}
          resolve(responseObj);
        });
      })
    }
  }

  /*
    we changed some of the vuex services to use WebSockets calls
    and commented out the original REST calls
    other services still use REST calls to the same QEWD-Up back-end
    because we can use both REST and WebSocket endpoints simultaneously!
    at the back-end, we can even (re)use the same code for both
    REST & WebSocket handlers using a small wrapper function
  */

  export const TagsService = {
    get() {
      //return ApiService.get("tags");
      return QEWDService.reply({
        type: "getTags"
      })
    }
  };

  export const ArticlesService = {
    query(type, params) {
      /*
      return ApiService.query("articles" + (type === "feed" ? "/feed" : ""), {
        params: params
      });
      */
      return QEWDService.reply({
        type: (type === "feed") ? "getArticlesFeed" : "getArticlesList",
        query: params,
        JWT: JwtService.getToken()
      })
    },
    get(slug) {
      //return ApiService.get("articles", slug);
      return QEWDService.reply({
        type: "getArticleBySlug",
        slug,
        JWT: JwtService.getToken()
      })
    },
    create(params) {
      //return ApiService.post("articles", { article: params });
      return QEWDService.reply({
        type: "createArticle",
        body: {
          article: params
        },
        JWT: JwtService.getToken()
      });
    },
    update(slug, params) {
      return ApiService.update("articles", slug, { article: params });
    },
    destroy(slug) {
      return ApiService.delete(`articles/${slug}`);
    }
  };

  export const CommentsService = {
    get(slug) {
      if (typeof slug !== "string") {
        throw new Error(
          "[RWV] CommentsService.get() article slug required to fetch comments"
        );
      }
      return ApiService.get("articles", `${slug}/comments`);
      /*
      return QEWDService.reply({
        type: "getComments",
        slug,
        JWT: JwtService.getToken()
      })
      */
    },

    post(slug, payload) {
      return ApiService.post(`articles/${slug}/comments`, {
        comment: { body: payload }
      });
    },

    destroy(slug, commentId) {
      return ApiService.delete(`articles/${slug}/comments/${commentId}`);
    }
  };

  export const FavoriteService = {
    add(slug) {
      return ApiService.post(`articles/${slug}/favorite`);
    },
    remove(slug) {
      return ApiService.delete(`articles/${slug}/favorite`);
    }
  };

  ```
These files are the only ones you need to change to make the app work with QEWD-Up/QEWD.js WebSockets. All other source files in the Vue.js app stay untouched.

*** Note that the complete app functionality and error handling still needs to be completed, but in this draft app version you can already get the idea how to use a WebSocket back-end with this standard example app front-end.