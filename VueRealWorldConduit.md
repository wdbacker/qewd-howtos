# Vue RealWorld example app (REST version)

This app is using the Vuejs 2.x framework, see also similar apps for the [React](ReactRealWorldConduit.md) and [Angular](AngularRealWorldConduit.md) framework.

## Development environment setup on Windows

- create your (free) [GitHub account](https://github.com)
- download & install [Git](https://git-scm.com/downloads) (in the installer, just use the default settings)
- download & install [VSCode](https://code.visualstudio.com)
- start VSCode
- download & install [Node.js](https://nodejs.org)
- test Node.js in a cmd window:
  ```
  C:\> node -v
  v12.18.4
  ```
- download & install [@vue/cli](https://cli.vuejs.org/)
  ```
  C:\> npm install -g @vue/cli
  ```
At this point, your development environment is completely ready for building Vue.js apps using the standard build tools.

## <a name="setup"></a>Setup the example app with a REST back-end

- open the [Thinkster RealWorld frontends page](https://github.com/gothinkster/realworld#frontends) and select the [Vue RealWorld example app](https://github.com/gothinkster/vue-realworld-example-app)
- fork this repository in VSCode with Ctrl+Shift+P, Git Clone and enter the [url](https://github.com/gothinkster/vue-realworld-example-app) to a directory of your choice, e.g. `C:\GitHub`
- open the forked project in VSCode
- open a cmd window and cd to `C:\GitHub\vue-realworld-example-app`
  ```
  C:\>cd \GitHub
  C:\GitHub>cd vue-realworld-example-app
  ```
- install all project dependencies from npm using:
  ```
  C:\GitHub\vue-realworld-example-app>npm install
  ```
- start the Node.js development server:
  ```
  C:\GitHub\vue-realworld-example-app>npm run serve
  ...

  App running at:
  - Local:   http://localhost:8080/
  - Network: http://192.168.x.x:8080/

  Note that the development build is not optimized.
  To create a production build, run yarn build.
  ```
- you may need to adjust an eslint setting inside `.eslintrc.js`:
  ```javascript
  module.exports = {
    root: true,
    env: {
      node: true
    },
    extends: ["plugin:vue/essential", "@vue/prettier"],
    rules: {
      "no-console": process.env.NODE_ENV === "production" ? "error" : "off",
      "no-debugger": process.env.NODE_ENV === "production" ? "error" : "off"
    },
    parserOptions: {
      parser: "babel-eslint"
    }
  };
  ```
  remove `"@vue/prettier"` from the `extends` option array to avoid a lot of prettier/prettier warnings
- in VSCode, go to the debug tab and run the Vue.js app (you'll need to choose/create a launch config of type `Chrome: Launch`)
- the app will open in Chrome and show the latest articles from the public demo back-end from RealWorld

## Change the app to use a QEWD-Up/QEWD.js REST back-end

- open the `src/common/config.js` file and change the `API_URL` (adjust API_URL to your local QEWD-Up/QEWD.js qewd-conduit server ip address):
  ```javascript
  //export const API_URL = "https://conduit.productionready.io/api";
  export const API_URL = "http://192.168.x.x:8080/api";
  export default API_URL;
  ```
- save & run your app again from the debug tab: it should run now on your own QEWD-Up/QEWD.js REST back-end!

## Run this same app on a QEWD-Up/QEWD.js WebSockets back-end

There is also an [interactive version of this app](VueRealWorldConduitWS.md) running on the same QEWD-Up/QEWD.js back-end using WebSockets.