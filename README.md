# WindowsDeployment
Here is a guide on how to create a custom windows image for deployment using WinPE and netbootxyz



## Step1: Windows installation
First of all you'll need to install Windows (10 or 11) on your computer. For this you can use Windows MediaCreationTool or rufus if you already have an iso to create a bootable usb key.
https://support.microsoft.com/en-us/windows/create-installation-media-for-windows-99a58364-8c02-206f-aa6f-40c3b507420d#id0ejd=windows_10

After you have created the bootable usb key, you'll need to boot with it. When booting, press F2, F10 or the key that will allow you to enter the On Time Boot menu. Then boot ith the usb key and preceed with the installation.

If you don't want to use a Microsoft account during installation, disconnect from any network if connected, press Shift+F10 and run :

```
oobe\bypassnro
```

This is the command for Windows 10, this might not work on Windows 11.

## Step2: Prepare the Windows image

Now that Windows is installed on your computer, install all windows updates and the drivers or apps needed if you will deploy Windows on the same computers. For exemple, I will deploy Windows 10 on Dell laptops, so I will install Dell Command Update.

Once this is done, we can launch a sysprep to delete from the image all informations that are related to the local user you are using right now, with the graphical interface or the following command :

```
sysprep /oobe /generalize /shutdown
```

Sometimes, there are errors while running sysprep. 
I encountered a similar error:
1. Package Microsoft.LanguageExperiencePackit-IT_19041.3.7.0_neutral__8wekyb3d8bbwe was installed for a user, but not provisioned for all users. This package will not function properly in the sysprep image.

2. SYSPRP Failed to remove staged package Microsoft.LanguageExperiencePackit-IT_19041.49.150.0_neutral__8wekyb3d8bbwe. Failed to remove apps for the current user.

and fixed it with :
```
Remove-AppxPackage -allusers Microsoft.XboxApp_31.32.16002.0_neutral_~_8wekyb3d8bbwe
```
You'll need to change the name of the package with the one causing the error. You can find it on sysprep logs.