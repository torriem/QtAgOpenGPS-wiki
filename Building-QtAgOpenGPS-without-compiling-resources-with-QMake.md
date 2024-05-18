WARNING: This page applies only to the settingsdavid branch of QtAgOpenGPS at this time.

Normally all the QML files and other resources used by QtAgOpenGPS are compiled right into the executable by the qrc resource compiler. This is convenient, but it makes QML development excruciatingly slow.  Fortunately during the development process it is possible to set a qmake option to tell QtAgOpenGPS to load resources from the disk at run time, instead of compiling them in.  To do this append the following to your qmake command:
`DEFINES+=LOCAL_QML`
With this DEFINE set, when QtAgOpenGPS is run, it will look in the following places for the resources in their respective folders (`qml`, `images`, `sounds`, etc), relative to the current working directory or the executable's directory:
* `../`
* `../qtaog/`
* `../QtAgOpenGPS/`

It stops looking when it finds qml/AOGInterface.qml, so if you have multiple copies of the resources, it may not be finding the directory you are expecting. QtAgOpenGPS will display a warning message to the console indicating what directory it is using.

In Qt Creator you can set up a custom build configuration to do this for you automatically.  This makes it easy to switch between developing without the resources compiled in, or creating a debug or release build with the resources all compiled it.

To this, load the QtAgOpenGPS.pro project file in Qt Creator.  
* Click on the "Projects" tab.  
* Find your current, activated kit and click on "Build."  
* In the Build Settings pane, be sure that "Debug" is selected as the current build configuration. 
* Then click on "Add."  Enter a configuration name such as "Debug no qrc."  
* Under the "Build Steps" area, click on the "Details" button for the **qmake** step.  
* Under "Additional Arguments" set the field to `DEFINES+=LOCAL_QML`.

You can go back to the "Edit" tab.  To switch between normal "Debug" and "Debug no qrc," click on the icon above the play button and select on or the other. Qt Creator will automatically reconfigure itself and set up the build steps.

