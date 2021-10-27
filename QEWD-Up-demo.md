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