# Debugging your QEWD-Up server in VSCode

## Setting up your environment

First, create a `.vscode/launch.json` file in your QEWD-Up project:

```javascript
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "protocol": "inspector",
      "port": 9229,
      "request": "attach",
      "name": "Attach to QEWD Up server"
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Launch QEWD Up server in inspector mode",
      "program": "${workspaceFolder}\\node_modules\\qewd\\up\\run_native",
      "cwd": "${workspaceFolder}",
      "autoAttachChildProcesses": true,
      "console": "integratedTerminal"
    }
  ]
}
```
This launch.json file allows you to debug your server in two ways:
- start your QEWD-Up server as usual in an external console window using a debug option on the commandline
- start your QEWD-Up server within VSCode using VSCode's `integratedTerminal`

*In a Windows environment, there is one option that you must avoid: never use the default `"console": "internalConsole"` option. Because the VSCode debug console doesn't pass the Ctrl-C  (SIGINT) signal to the running master process, your QEWD-Up will not stop gracefully and leave all database connections open. Using `"console": "internalConsole"` is the way to go because you get a full PowerShell command window supporting SIGINT on pressing Ctrl-C to stop the server. Also never use the debugger's red square stop button, only use Ctrl-C in the PowerShell terminal inside VSCode!*

### Starting your QEWD-Up server in an external console window using a debug option on the commandline

Steps:
- open an external command window
- start your QEWD-Up server on the command line using:
  ```
  node --inspect node_modules/qewd/up/run_native
  ```
  the QEWD-Up server starts as usual but will listen for incoming debugging connections from VSCode on debug port 9229
- head over to your VSCode window and go to the Debug tab
- start debugging using the `Attach to QEWD-Up server` settings

You can debug your handler code now with breakpoints, step by step, ... using all features VSCode's Node debugger provides.

This debugging method is recommended for:
- debugging handlers and/or lifecycle hook functions
- debug a staging (or even production) server

To debug QEWD-Up server startup code however, this debug option is not suited (e.g. when debugging your addMiddleWare code).

### Starting your QEWD-Up server within VSCode using VSCode's `integratedTerminal`

Steps:
- inside VSCode, go to the Debug tab
- start debugging using the `Launch QEWD Up server in inspector mode` settings
  
You can debug now your QEWD-Up server completely from the very beginning, including your handler code, request/response hooks, all startup modules like addMiddleware, ...

This debugging method is not suited however on a staging (or even production) environment.