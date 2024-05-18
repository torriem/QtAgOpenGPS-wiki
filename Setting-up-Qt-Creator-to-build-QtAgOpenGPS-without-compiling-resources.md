Normally all the QML files and other resources used by QtAgOpenGPS are compiled right into the executable by the qrc resource compiler. This is convenient, but it makes QML development excruciatingly slow.  Fortunately during the development process it is possible to set a qmake option to tell QtAgOpenGPS to load resources from the disk at run time, instead of compiling them in.  To do this append the following to your qmake command:
`DEFINES+=LOCAL_QML`
With this DEFINE set, when QtAgOpenGPS is run, it will look in the following places for the resources in their respective folders (`qml`, `images`, `sounds`, etc), relative to the current working directory:
* `../`
* `../qtaog/`
* `../QtAgOpenGPS/`
