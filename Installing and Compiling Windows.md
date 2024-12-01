# Installing Qt and Building QtAgOpenGPS
As tested on Windows 11

## Download and Install Qt
### Download The Qt Installer

Start here: https://www.qt.io/download-open-source <br>
Note that you might have to sign in(create an account if necessary).

Scroll down to the bottom and click the "Download the Qt Online Installer" link. (Why can't this be a giant Download button, Qt???)<br>
 Select your OS.

 Run the installer that downloads.

Start running through the installer(You'll have to sign into your Qt Account). <br>

When you get to this page:
![](./.images/QtInst_Specify_Dir.png)
Select Custom Installation,<br>
Select the ojects shown below. You don't need anything in Preview, Qt Design Studio, or Extensions.<br>
I suppose one could use 6.8.0, but 6.7.3 has been proven to work. <br>

If you don't plan to build for Android, you don't need to select it.

Very likely, not all of these are needed. It's just easier to install all of them, then to miss one, and have to re-run the installer.
![](./.images/Inst_Select_Components.png)

Finish the installer and Download. It will take awhile to download and install.

Click "Open QtCreator" once the installer is done. 

