PHP Debug Adapter for Visual Studio Code
========================================

[![Latest Release](https://vsmarketplacebadge.apphb.com/version-short/felixfbecker.php-debug.svg)](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug) [![Installs](https://vsmarketplacebadge.apphb.com/installs/felixfbecker.php-debug.svg)](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug) [![Rating](https://vsmarketplacebadge.apphb.com/rating-short/felixfbecker.php-debug.svg)](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug) [![Build Status](https://travis-ci.org/felixfbecker/vscode-php-debug.svg?branch=master)](https://travis-ci.org/felixfbecker/vscode-php-debug) [![Build Status Windows](https://ci.appveyor.com/api/projects/status/hda6n2umfdt6eyms/branch/master?svg=true)](https://ci.appveyor.com/project/felixfbecker/vscode-php-debug/branch/master) [![Coverage](https://codecov.io/gh/felixfbecker/vscode-php-debug/branch/master/graph/badge.svg)](https://codecov.io/gh/felixfbecker/vscode-php-debug) [![Dependency Status](https://gemnasium.com/felixfbecker/vscode-php-debug.svg)](https://gemnasium.com/felixfbecker/vscode-php-debug) [![Gitter](https://badges.gitter.im/felixfbecker/vscode-php-debug.svg)](https://gitter.im/felixfbecker/vscode-php-debug?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

![Demo GIF](images/demo.gif)

Installation
------------

Install the extension: Press `F1`, type `ext install php-debug`.

This extension is a debug adapter between VS Code and [XDebug](https://xdebug.org/) by Derick Rethan. XDebug is a PHP extension (a `.so` file on Linux and a `.dll` on Windows) that needs to be installed on your server.

 1. [Install XDebug](https://xdebug.org/docs/install)  
  ***I highly recommend you make a simple `test.php` file, put a `phpinfo();` statement in there, then copy the output and paste it into the [XDebug installation wizard](https://xdebug.org/wizard.php). It will analyze it and give you tailored installation instructions for your environment.***  
  In short:
   - On Windows: [Download](https://xdebug.org/download.php) the appropiate precompiled DLL for your PHP version, architecture (64/32 Bit), thread safety (TS/NTS) and Visual Studio compiler version and place it in your PHP extension folder.
   - On Linux: Either download the source code as a tarball or [clone it with git](https://xdebug.org/docs/install#source), then [compile it](https://xdebug.org/docs/install#compile).
 2. [Configure PHP to use XDebug](https://xdebug.org/docs/install#configure-php) by adding `zend_extension=path/to/xdebug` to your php.ini.  
  The path of your php.ini is shown in your `phpinfo()` output under "Loaded Configuration File".
 3. Enable remote debugging in your php.ini:

  ```ini
  [XDebug]
  xdebug.remote_enable = 1
  xdebug.remote_autostart = 1
  ```
  There are other ways to tell XDebug to connect to a remote debugger than `remote_autostart`, like cookies, query parameters or browser extensions. I recommend `remote_autostart` because it "just works". There are also a variety of other options, like the port (by default 9000), please see the [XDebug documentation on remote debugging](https://xdebug.org/docs/remote#starting) for more information.
 4. If you are doing web development, don't forget to restart your webserver to reload the settings
 5. Verify your installation by checking your `phpinfo()` output for an XDebug section.

### VS Code Configuration
In your project, go to the debugger and hit the little gear icon and choose _PHP_. A new launch configuration will be created for you with two configurations:
 - **Listen for XDebug**  
   This setting will simply start listening on the specified port (by default 9000) for XDebug. If you configured XDebug like recommended above, everytime you make a request with a browser to your webserver or launch a CLI script XDebug will connect and you can stop on breakpoints, exceptions etc.
 - **Launch currently open script**  
   This setting is an example of CLI debugging. It will launch the currently opened script as a CLI, show all stdout/stderr output in the debug console and end the debug session once the script exits.

#### Supported launch.json settings:
 - `request`: Always `"launch"`
 - `port`: The port on which to listen for XDebug (default: `9000`)
 - `stopOnEntry`: Wether to break at the beginning of the script (default: `false`)
 - `localSourceRoot`: The path to the folder that is being served by your webserver and maps to `serverSourceRoot` (for example `"${workspaceRoot}/public"`)
 - `serverSourceRoot`: The path on the remote host where your webroot is located (for example `"/var/www"`)
 - `log`: Wether to log all communication between VS Code and the adapter to the debug console. See _Troubleshooting_ further down.

Options specific to CLI debugging:
 - `program`: Path to the script that should be launched
 - `args`: Arguments passed to the script
 - `cwd`: The current working directory to use when launching the script
 - `runtimeExecutable`: Path to the PHP binary used for launching the script. By default the one on the PATH.
 - `runtimeArgs`: Additional arguments to pass to the PHP binary
 - `externalConsole`: Launches the script in an external console window instead of the debug console (default: `false`)
 - `env`: Environment variables to pass to the script

Features
--------
 - Line breakpoints
 - Conditional breakpoints
 - Function breakpoints
 - Step over, step in, step out
 - Break on entry
 - Breaking on uncaught exceptions and errors / warnings / notices
 - Multiple, parallel requests
 - Stack traces, scope variables, superglobals, user defined constants
 - Arrays & objects (including classname, private and static properties)
 - Debug console
 - Autocompletion in debug console for variables, array indexes, object properties (even nested)
 - Watches
 - Run as CLI
 - Run without debugging

Remote Host Debugging
---------------------
To debug a running application on a remote host, you need to tell XDebug to connect to a different IP than `localhost`. This can either be done by setting [`xdebug.remote_host`](https://xdebug.org/docs/remote#remote_host) to your IP or by setting [`xdebug.remote_connect_back = 1`](https://xdebug.org/docs/remote#remote_connect_back) to make XDebug always connect back to the machine who did the web request. The latter is the only setting that supports multiple users debugging the same server and "just works" for web projects. Again, please see the [XDebug documentation](https://xdebug.org/docs/remote#communcation) on the subject for more information.

To make VS Code map the files on the server to the right files on your local machine, you have to set the `localSourceRoot` and `serverSourceRoot` settings in your launch.json. Example:
```json
"serverSourceRoot": "/var/www/myproject",
"localSourceRoot": "${workspaceRoot}/public"
```
Please also note that setting any of the CLI debugging options will not work with remote host debugging, because the script is always launched locally. If you want to debug a CLI script on a remote host, you need to launch it manually from the command line.

Troubleshooting
---------------
 - Ask a question on [Gitter](https://gitter.im/felixfbecker/vscode-php-debug)
 - If you think you found a bug, [open an issue](https://github.com/felixfbecker/vscode-php-debug/issues)
 - Make sure you have the latest version of this extension and XDebug installed
 - Try out a simple PHP file to recreate the issue, for example from the [testproject](https://github.com/felixfbecker/vscode-php-debug/tree/master/testproject)
 - In your php.ini, set [`xdebug.remote_log = /path/to/logfile`](https://xdebug.org/docs/remote#remote_log)
   (make sure your webserver has write permissions to the file)
 - Set `"log": true` in your launch.json

Contributing
------------
To hack on this adapter, clone the repository and open it in VS Code. You need NodeJS and typings installed (`npm install -g typings`). Install dependencies by running `npm install` and `typings install`.

You can debug the extension (run it in "server mode") by selecting the "Debug adapter" launch configuration and hitting `F5`. Then, open a terminal inside the project, and open the included testproject with VS Code while specifying the current directory as `extensionDevelopmentPath`:

```sh
code testproject --extensionDevelopmentPath=.
```

VS Code will open an "Extension Development Host" with the debug adapter running. Open `.vscode/launch.json` and uncomment the `debugServer` configuration line. Hit `F5` to start a debugging session. Now you can debug the testproject like specified above and set breakpoints inside your first VS Code instance to step through the adapter code.

The extension is written in TypeScript and compiled using a Gulpfile that first transpiles to ES6 and then uses Babel to specifically target VS Code's Node version. You can run the compile task through `npm run compile`, `gulp compile` or from VS Code with `Ctrl`+`Shift`+`B`. `npm run watch` / `gulp watch` enables incremental compilation.

Tests are written with Mocha and can be run with `npm test`. The tests are run in CI on Linux and Windows against PHP 5.4, 5.6, 7.0 and XDebug 2.3, 2.4.
