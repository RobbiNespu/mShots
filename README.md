
mShots.JS
=========

Overview
--------
This application is split into four distinct components, the mShots PHP class (receiving requests from the WordPress mShots plugin),
the node.JS cluster service to manage the filtered incoming requests passed on from the mShots class, the native node.JS module,
Snapper, which performs the actual snapshot generation and custom compiled Qt5.1 libraries to run X-less in the command line
server environment on Debian Linux (Wheezy). Note: The virtual frame buffer Kernel module (VFB) is required for the service.

Installation for Development
----------------------------
If anything goes awry or is unclear in one of these steps, take a look at the "Details" section below for slightly more detailed info.

1) Install the latest node.js binaries from http://nodejs.org

2) Place the mShots folder in "/opt/", so the final path is "/opt/mShots.JS/".

3) Extract the "deps.tar.bz" file in the "/opt/mShots.JS/deps" folder into the same folder with the command:
	"sudo tar xjf /opt/mShots.JS/deps/deps.tar.bz -C /opt/mShots.JS/deps/".

4) Run the following commands to place the require files in "/usr/lib/".
	sudo cp -d /opt/mShots.JS/deps/<platform>/lib/libQt5Core.so* /usr/lib/
    sudo cp -d /opt/mShots.JS/deps/<platform>/lib/libQt5Network.so* /usr/lib/
    sudo cp -d /opt/mShots.JS/deps/<platform>/lib/libQt5WebKitWidgets.so* /usr/lib/

5) mShots.JS requires a logging library, install this with "sudo npm install log4js", while in the mShots.JS root directory "/opt/mShots.JS".

6) While in the mShots root directory ("/opt/mShots.JS"), run "sudo node-gyp configure". Followed by "sudo node-gyp build".
	If node-gyp is not installed, run "sudo npm install -g node-gyp" to install it.

7) You are now ready to run the mShots.JS service with "./mshots_ctl.sh start|stop". Note: The virtual frame buffer Kernel module (VFB) is required.

Details
-------

The mShots PHP Class (code in "public_html")

This class is designed to buffer the thumbnail requests, by limiting snapshots to only be taken every 24 hours. The most useful attribute
of this class is the "disable_requeue" constant, but changing this to true you effectively stop all calls to the mShots.JS service, which
can be used if the service misbehaves for any reason. With this setting changed existing thumbnails will be served, but no new thumbnails
are generated.

Snapper node.JS Module (code in "src")

To compile the node module you will need to follow the following steps:
1. Install node.JS from http://nodejs.org
2. Once this is installed confirm node-gyp is installed by typing "node-gyp -v" at the command prompt.
	If it is not installed, install it with the command "sudo npm install -g node-gyp".
3. For the compilation of the module you need the Qt5 header and Qt5.1 library files. These are provided as a separate tar file in the deps
	folder and will need to be extracted into the "deps" folder.
4. If you are installing onto Debian you will need to ensure that libssl is installed before proceeding to the next step. To do this run
	"sudo apt-get install openssl".
5. To compile the Snapper node module you, whilst in the root of the mShots.JS directory, run "sudo node-gyp configure" and then
	"sudo node-gyp build". The resulting file will be in the build/Release/ directory.
6. Before running the mShots node script, you will need to run the following commands:
	sudo cp -d /opt/mShots.JS/deps/<platform>/lib/libQt5Core.so* /usr/lib/
    sudo cp -d /opt/mShots.JS/deps/<platform>/lib/libQt5Network.so* /usr/lib/
    sudo cp -d /opt/mShots.JS/deps/<platform>/lib/libQt5WebKitWidgets.so* /usr/lib/

mShots node.JS Program (code in "lib")

The node program has two dependencies, the Snapper module above and the log4js module for program logging.
1. At the prompt, change to the root directory of mShots.JS, install the log4js module with the command "npm install log4js".
2. Control the execution of the program with the mshots_ctl.sh bash script in the root directory or by manually running the
	following command from the terminal, whilst in the mShots root directory:
	"node lib/mshots.js -p <port number> -n <nun threads>" and terminating execution with Ctrl-C.

Custom Qt Compilation for Debian (headers and binaries must be in placed the relevant "deps" folder)

The Snapper node.JS module for Debian Linux required a custom compilation of Qt5.1, using the simple patch located in "deps/linux_x64/",
and the compilation options in the same directory. The reason for modifification from the original was overcome a text/glyph rendering bug
in Qt, which causes many sites to abruptly segfault.

Development for Max OS X
------------------------

This is supported, you will need to install the latest Qt 5.1 libraries and then copy the relevant modules required for linking into the
"./deps/darwin_x64/" directory (or modify the paths in "binding.gyp" accordingly). The required models for linking are Core, Network and WebKitWidgets.