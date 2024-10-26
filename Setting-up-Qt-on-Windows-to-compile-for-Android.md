* Install Java SE, latest version.  Go to https://www.oracle.com/ca-en/java/technologies/downloads/ and download the Windows installer.  At the time of this writing, 23.0.1 is the latest
* Download and run the Qt installer from https://www.qt.io/download-qt-installer-oss.  You will need to create a Qt Project account with you email address.  
    * Install Qt, latest 6.x version.  At the time of this writing is 6.8.0.  In the installer, expand the 6.8.0 item and choose add the following individual parts:
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
* Launch Qt Creator
   * Choose Edit->Preferences, click on "Devices", then select the "Android" tab.
   * Click "Setup SDK."  Qt Creator will ask if you want to install missing packages. Click "Ok."  After the that finishes, you should see a valid path in the Android NDK list box.
   * Click "Manage SDK." 
      * expand the "Tools" tree and check the following items:
          * Google USB Drivers
          * NDK (Side by side) 26.1.xxx
          * NDK (Side by side) 25.1.xxx
          * Android Emulator
          * Android SDK Platform-Tools
          * Android SDK Build-Tools 34
          * Android SDK Build-Tools 31
      * expand the Android SDK 35 tree and check the following items:
          * SDK Platform
          * ARM 64 v8a System Image
      * click the "Apply" button to download these packages.
   * Dialog box should now say "Android settings are OK."
   * Click on "Kits" in the left-hand pane and you should now see Auto-detected build kits for Android that you can activate for the current project.
   * Test by opening the example calqlatr project.  You should be able to configure a Android Qt 6.8.0 Clang arm64-v8a target for this project.
* TODO: set up emulator
* Attach phone by USB and Qt Creator can upload the apk to it and run it or debug it from within Qt Creator.

     