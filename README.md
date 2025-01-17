This program and the accompanying materials are
made available under the terms of the Eclipse Public License v2.0 which accompanies
this distribution, and is available at https://www.eclipse.org/legal/epl-v20.html

SPDX-License-Identifier: EPL-2.0

Copyright Contributors to the Zowe Project.
# zlux-app-server
This is the default setup of the Zowe App Server, built upon the zLUX framework. Within, you will find a collection of build, deploy, and run scripts as well as configuration files that will help you to configure a simple zLUX server with a few Apps included.

## Server layout
At the core of the zLUX App infrastructure backend is an extensible server, written for nodeJS and utilizing expressJS for routing. It handles the backend components of Apps, and also can serve as a proxy for requests from Apps to additional servers as needed. One such proxy destination is the ZSS - a Zowe backend component called **Zowe System Services**. It's a so-called agent for the App server.

### ZSS & zLUX Server overlap
The zLUX App Server and ZSS utilize the same deployment & App/Plugin structure, and share some configuration parameters as well. It is possible to run ZSS and zLUX App Server from the same system, in which case you would be running under z/OS USS. This configuration requires that IBM's version of nodeJS is installed prior.

Another way to set up zLUX is to have the zLUX App Server running under LUW, while keeping ZSS under USS. This is the configuration scenario presented below. In this scenario, you'll need to clone these github repositories to two different systems, and they'll need to have compatible configurations. For first-timers, it is fine to have identical configuration files and /plugins folders in order to get going.

## First-time Installation & Use
Getting started with this server requires just a few steps:

0. [Install Prerequisites](#0-install-prerequisites)
1. [Acquire the source code](#1-acquire-the-source-code)
2. [Set the server configuration](#2-set-the-server-configuration)
3. [Build zLUX Apps](#3-build-zlux-apps)
4. [Deploy server configuration files](#4-deploy-server-configuration-files)
5. [Build ZSS](#5-build-zss)
6. [Run the server](#6-run-the-server)
7. [Connect in a browser!](#7-connect-in-a-browser)

So, with that in mind, follow each step and you'll be on your way to your first zLUX App Server instance!

### 0. Install Prerequisites
Wherever the App Server is installed, the following is required for running:

* **NodeJS** - v6.14.4 minimum for z/OS,  elsewhere 6, 8, and 10 work well.

* **npm** - v6.4 minimum

For building zLUX framework and apps:

* **jdk** - v8 minimum
 
* **ant** - v1.10 minimum

* **ant-contrib** - v1 minimum

For building zss:

* **IBM z/OS XLC compiler for Metal C Compilation**

For developent:

* **git** - 2.18 or higher is recommended off z/os

* **ssh agent** - Our repositories are structured to expect that you have ssh keys setup for github. This assists with rapid development and automation. 
Git bash or putty's pageant are some of various tools that can help you setup & work with ssh keys over git.

#### (Optional) Install git for z/OS
Because all of our code is on github, yet ZSS must run on z/OS and the zLUX App Server may optionally run on z/OS as well, having git on z/OS is the most convenient way to work with the source code. The alternative would be to utilize FTP or another method to transfer contents to z/OS.
If you'd like to go this route, you can find git for z/OS free of charge here: http://www.rocketsoftware.com/product-categories/mainframe/git-for-zos
On z/OS, git 2.14.4 is the minimum needed.


### 1. Acquire the source code
To get started, first clone (or download) the code necessary to build zss and the zss cross memory server.
If using git, the following commands should be used on z/OS:
```
git clone --recursive git@github.com:zowe/zss.git
```

Afterwards, clone (or download) the github capstone repository, https://github.com/zowe/zlux
As we'll be configuring ZSS on z/OS's USS, and the zLUX App Server on a LUW host, you'll need to put the contents on both systems.
If using git, the following commands should be used:
```
git clone --recursive git@github.com:zowe/zlux.git
cd zlux
git submodule foreach "git checkout master"
```

By default the trivial authentication backend is enabled which always returns successful unless authentication 
information provided is in an incorrect format. To use ZSS as an authentication backend, clone (or download) the 
`zss-auth` plugin code into the `zlux` directory:

```
git clone git@github.com:zowe/zss-auth.git
```

To bring Apps to zLUX App server, you need to clone (or download) corresponding repositories into `zlux`. For 
example, clone (or download) the following to get sample apps source code:

```
git clone git@github.com:zowe/sample-angular-app.git
git clone git@github.com:zowe/sample-react-app.git
git clone git@github.com:zowe/sample-iframe-app.git
```

At this point, you'll have the latest code from each repository on your system.
Continue from within zlux-app-server.

### 2. Set the server configuration
Read the [Configuration](https://github.com/zowe/zlux/wiki/Configuration-for-zLUX-App-Server-&-ZSS) wiki page for a detailed explanation of the primary items that you'll want to configure for your first server.

In short, ensure that within **zlux-app-server/config/zluxserver.json**, **node.https.port + other HTTPS parameters** are set to your liking on the LUW host, and that **agent.http.port** is set on the z/OS host.

#### Setup for ZSS
If you will be using ZSS as an authentication backend, set `dataserviceAuthentication.defaultAuthentication = "zss"` and
 `dataserviceAuthentication.implementationDefaults.zss.plugins = ["org.zowe.zlux.auth.zss"]`.
 
Next, set **agent.http.port** to the port where you want ZSS to listen on. This must be done at minimum on the z/OS host, but can also be done in the zluxserver.json where the App server is running, if it is not the same.
Finally, if the App server is running off of z/OS, then you will need to change **agent.http.ipAddresses** to a hostname or ip address that is externally visible.
**Note: It is highly recommended to turn on HTTPS for ZSS via [configuring AT-TLS](https://zowe.github.io/docs-site/latest/user-guide/mvd-configuration.html#configuring-zss-for-https) when using ZSS externally, as the session security is essential for all but trivial development environments**

For each App that is supposed to be loaded by zLUX App server, a plugin locator should be defined. Read the 
[Plugin Definition & Structure](https://github.com/zowe/zlux/wiki/Zlux-Plugin-Definition-&-Structure) wiki page for 
details. For example, to enable Angular Sample App, create 
`zlux-app-server/plugins/org.zowe.zlux.sample.angular.json` with the following contents:
```
{
  "identifier": "org.zowe.zlux.sample.angular",
  "pluginLocation": "../../sample-angular-app"
}
```

Before continuing, if you intend to use the terminal, at this time (temporarily) it must be pre-configured to know the destination host.
Edit *tn3270-ng2/_defaultTN3270.json* to set *host* and *port* to a valid TN3270 server telnet host and port and then save the file.
Edit *vt-ng2/_defaultVT.json* to set *host* and *port* to a valid ssh host and port and then save the file.

### 3. Build zLUX Apps
**Note when building, NPM is used. The version of NPM needed for the build to succeed should be at least 6.4. You can update NPM by executing `npm install -g npm`**

zLUX Apps can contain server and/or web components. The web components must be built, as webpack is involved in optimized packaging, and server components are also likely to need building if they require external dependencies from NPM, use native code, or are written in typescript.

This server only needs transpilation and packaging of web components, and therefore we do not need any special build steps for the host running ZSS.

Instead, on the host running the zLUX App Server, run the script that will automatically build all included Apps.
Simply,
```
cd zlux-build

//Windows
build.bat

//Otherwise
./build.sh
```
This will take some time to complete.

**Note:** It has been reported that building can hang on Windows if you have put the code in a directory that has a symbolic link. Build time can depend on hardware speed, but should take minutes not hours.

### 4. Deploy server configuration files
If you are running the zLUX App Server seperate from ZSS, you must ensure the ZSS installation has its configuration deployed. You can accomplish this via:

```
// in zlux-build directory
ant deploy
```

On the other hand, if you are running ZSS and the zLUX App Server on the same host, *build.sh* and *build.bat* execute *deploy* and therefore this task was accomplished in step #4.

However, if you need to change the server configuration files or want to add more Apps to be included at startup, you'll need to update the deploy content to reflect this. Simply running deploy.bat or deploy.sh will accomplish this, but files such as zluxserver.json are only read at startup, so a reload of the zLUX App Server & ZSS would be required.

### 5. Build ZSS
ZSS is a dependency of zLUX, but exists in a seperate repository and must be run on z/OS. To get the code, first do the following on z/OS:
```
git clone --recursive git@github.com:zowe/zss.git
cd zss/build
```
Ant is used to build ZSS, and ZSS is built in two parts: the ZSS Server and the ZSS Cross-memory Server. ZSS Server communicates through HTTP(S) to zLUX, while the cross memory server is communicated with by ZSS through in-system calls.
To build both, run:
```
ant zss
ant zis
```
Afterwards, you need to copy `zssServer` to the `zlux-app-server/bin` directory, so that `nodeServer.sh` and `zssServer.sh` can invoke it.
You should also set the p attribute on it.
Do:
```
cp zssServer ../../zlux-app-server/bin
extattr +p ../../zlux-app-server/bin/zssServer
```
Finally, the ZSS Cross memory server must be installed and configured according to [This Install Guide](https://github.com/zowe/docs-site/blob/master/docs/user-guide/install-zos.md#manually-installing-the-zowe-cross-memory-server)


### 6. Run the server
At this point, all server files have been configured and Apps built, so ZSS and the App server are ready to run.
First, from the z/OS system, start ZSS.
```
cd ../zlux-app-server/bin
./zssServer.sh
```
This should start the zssServer. If the server did not start, two common sources of error are:

1. The *zssPort* chosen is already occupied. To fix, edit *config/zluxserver.json* to choose a new one, and re-run *build/deploy.sh* to have that change take effect.
2. The zssServer binary does not have the APF bit set. Since this server is meant for secure services, it is required. To fix, execute `extattr +a zssServer`.  Note you may need to alter the execute permissions of zssServer.sh in the event that the previous command is not satisfactory (eg chmod +x zssServer.sh) 

Second, from the system with the zLUX App Server, start it with a few parameters to hook it to ZSS.
```
cd ../zlux-app-server/bin

// Windows:
nodeServer.bat <parameters>

// Others:
./nodeServer.sh <parameters>
```
Valid parameters for nodeServer are as follows:
- *-h*: Specifies the hostname where ZSS can be found. Use as *-h \<hostname\>*
- *-P*: Specifies the port where ZSS can be found. Use as *-P \<port\>*. This overrides *zssPort* from the configuration file.
- *-p*: Specifies the HTTP port to be used by the zLUX App Server. Use as *-p <port>*. This overrides *node.http.port* from the configuration file.
- *-s*: Specifies the HTTPS port to be used by the zLUX App Server. Use as *-s <port>*. This overrides *node.https.port* from the configuration file.
- *--noChild*: If specified, tells the server to ignore and skip spawning of child processes defined as *node.childProcesses* in the configuration file.

In the example where we're running ZSS on a host named mainframe.zowe.com, running on zssPort = 19997, the App server running on Windows could be started with the following:

`nodeServer.bat -h mainframe.zowe.com -P 19997 -s 19998`

After which we'd be able to connect to the App server at HTTPS port 19998.

**NOTE: the parameter parsing is provided by [argumentParser.js](https://github.com/zowe/zlux-server-framework/blob/master/lib/argumentParser.js), which allows for a few variations of input, depending on preference. For example, the following are all valid ways to specify the ZSS host**

- **-h myhost.com**
- **-h=myhost.com**
- **--hostServer myhost.com**
- **--hostServer=myhost.com**

When the zLUX App Server has started, one of the last messages you will see as bootstrapping completes is that the server is listening on the HTTP/s port. At this time, you should be able to use the server.

### 7. Connect in a browser
Now that ZSS & the zLUX App Server are both started, you can access this instance by pointing your web browser to the zLUX App Server.
In this example, the address you will want to go to first is the location of the window management App - Zowe Desktop.
The URL for this is:

http(s)://\<zLUX App Server\>:\<node.http(s).port\>/ZLUX/plugins/org.zowe.zlux.bootstrap/web/index.html

Once here, you should be greeted with a Login screen and Apps in the main menu if there were 
included some. If you set up ZSS as an authentication backend, you can login with your mainframe 
credentials. By default trivial authentication is used which allows to login with arbitrary credentials.

There're a few Apps that you can try out to see how they interact with the framework:
- [tn3270-ng2](https://github.com/zowe/tn3270-ng2): This App communicates with the zLUX App Server to enable a TN3270 connection in the browser
- [sample-angular-app](https://github.com/zowe/sample-angular-app): A simple app showing how a zLUX App frontend (here, Angular) component can communicate with an App backend (REST) component.
- [sample-react-app](https://github.com/zowe/sample-react-app): Similar to the Angular App, but using React instead to show how you have the flexibility to use a framework of your choice.
- [sample-iframe-app](https://github.com/zowe/sample-iframe-app): Similar in functionality to the Angular & React Apps, but presented via inclusion of an iframe, to show that even pre-existing pages can be included


#### Deploy example
```
// All paths relative to zlux-app-server/js or zlux-app-server/bin
// In real installations, these values will be configured during the install.
  "rootDir":"../deploy",
  "productDir":"../deploy/product",
  "siteDir":"../deploy/site",
  "instanceDir":"../deploy/instance",
  "groupsDir":"../deploy/instance/groups",
  "usersDir":"../deploy/instance/users"

```

### App configuration
This section does not cover any dynamic runtime inclusion of Apps, but rather Apps defined in advance.
In the configuration file, a directory can be specified which contains JSON files which tell the server what App is to be included and where to find it on disk. The backend of these Apps use the Server's Plugin structure, so much of the server-side references to Apps use the term Plugin.

To include Apps, be sure to define the location of the Plugins directory in the configuration file, via the top-level attribute *pluginsDir*

**NOTE: In this repository, the directory for these JSON files is /plugins. Yet, in order to seperate configuration files from runtime files, the zlux-app-server repository copies the contents of this folder into /deploy/instance/ZLUX/plugins. So, the example configuration file uses the latter directory.**

#### Plugins directory example
```
// All paths relative to zlux-app-server/js or zlux-app-server/bin
// In real installations, these values will be configured during the install.
//...
  "pluginsDir":"../deploy/instance/ZLUX/plugins",
```

### ZSS Configuration
When running ZSS, it will require a JSON configuration file similar or the same as the one used for the zLUX server. The attributes that are needed for ZSS, at minimum, are:*rootDir*, *productDir*, *siteDir*, *instanceDir*, *groupsDir*, *usersDir*, *pluginsDir* and **agent.http.port**. All of these attributes have the same meaning as described above for the zLUX server, but if the zLUX server and ZSS are not run from the same location, then these directories may be different if desired.

The one attribute that is specific to ZSS however is **agent.http.port**. This is the TCP port which ZSS will listen on to be contacted by the zLUX server. Define this in the configuration file as a value between 1024-65535. See [zss configuration](https://github.com/zowe/zlux/wiki/Configuration-for-ZLUX-App-Server-&-ZSS#zss-configuration) for more information and an example.

#### Connecting zLUX server to ZSS
When running the zLUX server, simply specify a few flags to declare which ZSS instance zLUX will proxy ZSS requests to:
- *-h*: Declares the host where ZSS can be found. Use as "-h \<hostname\>"
- *-P*: Declares the port at which ZSS is listening. Use as "-P \<port\>"



This program and the accompanying materials are
made available under the terms of the Eclipse Public License v2.0 which accompanies
this distribution, and is available at https://www.eclipse.org/legal/epl-v20.html

SPDX-License-Identifier: EPL-2.0

Copyright Contributors to the Zowe Project.
