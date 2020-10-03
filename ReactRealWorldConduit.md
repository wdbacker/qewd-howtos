# React Redux RealWorld example app (REST version)

This app is using the React framework, see also similar apps for the [Vue.js](VueRealWorldConduit.md) and [Angular](AngularRealWorldConduit.md) framework.

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
- the default toolchain used in this project (*you don't need install this for this demo app*) for generating React app skeletons is [Create React App](https://github.com/facebook/create-react-app)

At this point, your development environment is completely ready for building React apps using the standard build tools.

## <a name="setup"></a>Setup the example app with a REST back-end

- open the [Thinkster RealWorld frontends page](https://github.com/gothinkster/realworld#frontends) and select the [React-Redux RealWorld example app](https://github.com/gothinkster/react-redux-realworld-example-app)
- fork this repository in VSCode with Ctrl+Shift+P, Git Clone and enter the [url](https://github.com/gothinkster/react-redux-realworld-example-app) to a directory of your choice, e.g. `C:\GitHub`
- open the forked project in VSCode
- open a cmd window and cd to `C:\GitHub\react-redux-realworld-example-app`
  ```
  C:\>cd \GitHub
  C:\GitHub>cd react-redux-realworld-example-app
  ```
- install all project dependencies from npm using:
  ```
  C:\GitHub\react-redux-realworld-example-app>npm install
  ```
- start the Node.js development server:
  ```
  C:\GitHub\react-redux-realworld-example-app>npm start
  ```
  the React development server will start and open a Chrome tab with the app running
- you can also navigate to `http://localhost:4100/`, you then see the app appear in your browser running on the public demo REST back-end
- from VSCode, go to the debug tab and run the React app (you'll need to choose/create a launch config of type `Chrome: Launch` and adjust the port from 8080 to 4100)
- the app will open in Chrome and show the latest articles from the public demo back-end from RealWorld

## Change the app to use a QEWD-Up/QEWD.js REST back-end

- open the `src/agent.js` file and change the `API_URL` (adjust API_URL to your local QEWD-Up/QEWD.js qewd-conduit server ip address):
  ```javascript
  import superagentPromise from 'superagent-promise';
  import _superagent from 'superagent';

  const superagent = superagentPromise(_superagent, global.Promise);

  //const API_ROOT = 'https://conduit.productionready.io/api';
  const API_ROOT = 'http://192.168.x.x:8080/api';

  ...
  ```
- save & run your app again from the debug tab: it should run now on your own QEWD-Up/QEWD.js REST back-end!

## Run this same app on a QEWD-Up/QEWD.js WebSockets back-end

There is no WebSocket version *yet* - I should upgrade my knowledge on React for that to happen :-)