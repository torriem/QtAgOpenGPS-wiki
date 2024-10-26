* Install Java SE, latest version.  Go to https://www.oracle.com/ca-en/java/technologies/downloads/ and download the Windows installer.  At the time of this writing, 23.0.1 is the latest
* Download and run the Qt installer from https://www.qt.io/download-qt-installer-oss.  You will need to create a Qt Project account with you email address.  
    * Install Qt, latest 6.x version.  At the time of this writing is 6.8.0
        * Expand the 6.8.0 item and choose the following individual parts:
            * MSVC 2022 if you have Visual Studio installed, otherwise choose MingW 64-bit
            * Android
            * Under "Additional Libraries," select the following:
                * Qt 5 Compatibility Module
                * Qt Image Formats
                * Qt Multimedia
                * Qt Positioning
                * Qt Quick 3D (not sure if this is required)
                * Qt Serial Port
                * Qt Shader Tools
     