# Angular RealWorld example app (REST version)

This app is using the Angular framework, see also similar apps for the [Vue.js](VueRealWorldConduit.md) and [React](ReactRealWorldConduit.md) framework.

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
- download & install [@angular/cli](https://cli.angular.io/)
  ```
  C:\> npm install -g @angular/cli
  ```
At this point, your development environment is completely ready for building Angular apps using the standard build tools.

## <a name="setup"></a>Setup the example app with a REST back-end

- open the [Thinkster RealWorld frontends page](https://github.com/gothinkster/realworld#frontends) and select the [Angular RealWorld example app](https://github.com/gothinkster/angular-realworld-example-app)
- fork this repository in VSCode with Ctrl+Shift+P, Git Clone and enter the [url](https://github.com/gothinkster/angular-realworld-example-app) to a directory of your choice, e.g. `C:\GitHub`
- open the forked project in VSCode
- open a cmd window and cd to `C:\GitHub\angular-realworld-example-app`
  ```
  C:\>cd \GitHub
  C:\GitHub>cd angular-realworld-example-app
  ```
- install all project dependencies from npm using:
  ```
  C:\GitHub\angular-realworld-example-app>npm install
  ```
- start the Node.js development server:
  ```
  C:\GitHub\angular-realworld-example-app>ng serve
  ```
- navigate to `http://localhost:4200/`, you should see the app appear in your browser running on the public demo REST back-end
- from VSCode, go to the debug tab and run the Angular app (you'll need to choose/create a launch config of type `Chrome: Launch` and adjust the port from 8080 to 4200)
- the app will open in Chrome and show the latest articles from the public demo back-end from RealWorld

## Change the app to use a QEWD-Up/QEWD.js REST back-end

- open the `src/environments/environment.ts` file and change the `API_URL` (adjust API_URL to your local QEWD-Up/QEWD.js qewd-conduit server ip address):
  ```javascript
  export const environment = {
    production: false,
    //api_url: 'https://conduit.productionready.io/api'
    api_url: 'http://192.168.x.x:8080/api'
  };
  ```
- save & run your app again from the debug tab: it should run now on your own QEWD-Up/QEWD.js REST back-end!

## Run this same app on a QEWD-Up/QEWD.js WebSockets back-end

There is no WebSocket version *yet* - I should learn more about Angular for that to happen :-)