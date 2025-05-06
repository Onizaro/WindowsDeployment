# WindowsDeployment
Here is a guide on how to create a custom windows image for deployment using WinPE and netbootxyz



## Requirements

### For image capture 
 - 2 computers or 1 computer with 1 Virtual Machine
 - 1 USB key 32 Gb or 16 Gb but it can fail depending on what you've installed on the image (my Windows 10 image was 15.5 Gb)

### For netbootxyz
 - DHCP + DNS servers



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
I encountered a similar error :
 - Package Microsoft.LanguageExperiencePackit-IT_19041.3.7.0_neutral__8wekyb3d8bbwe was installed for a user, but not provisioned for all users. This package will not function properly in the sysprep image.

and fixed it with :
```
Remove-AppxPackage -allusers Microsoft.XboxApp_31.32.16002.0_neutral_~_8wekyb3d8bbwe
```
You'll need to change the name of the package with the one causing the error. You can find it on sysprep logs.

https://azvise.com/2020/09/08/windows-10-sysprep-fails-due-to-an-app-that-was-installed-for-a-user-but-not-provisioned-for-all-users/

Once the sysprep is finished, do not turn on your computer, we need to prepare the WinPE usb key before. 


## Step 3: Prepare the WinPE USB key

WinPE is like a mini Windows and allows us to run scripts and capture images. Unlike sysprep, WinPE is not included in Windows so you'll have to install it on another machine that the one you used in the previous steps. 

https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/download-winpe--windows-pe?view=windows-11

Once installed, you should be able to run "deployment and imaging tool environment" on your machine:
![alt text](image.png)

Run this app as administrator and run the following commands 

```
copype amd64 "C:\WinPE_amd64"
Dism /Mount-Image /ImageFile:"C:\WinPE_amd64\media\sources\boot.wim" /index:1 /MountDir:"C:\WinPE_amd64\mount"
```

### 3.1 Optionnal: Create a script 
With WinPE, we can create a script that will run immediatly when booting. to create this script, you should modify the "startnet.cmd" file that is under this path for me :
```
C:\WinPE_amd64\mount\Windows\WinSxS\amd64_microsoft-windows-winpe_tools_31bf3856ad364e35_10.0.26100.1_none_f7f34b419f6c6524
```

It should be under the WinSxS folder, ther is another startnet.cmd file in the System32 folder but it will copy the content of the WinSxs's file so we don't need to modify it.

### 3.2 Optionnal: Add drivers to WinPE
We can also add drivers to WinPE. In my script I needed to be connected to the network so to ensure the connexion is done, I will add an ethernet driver to my WinPE.

To do so, you need the .inf file from the driver. Then, run the following command :
```
Dism /Add-Driver /Image:"C:\WinPE_amd64\mount" /Driver:"C:\driverLocation\driver.inf"
```

To add more customization to your WinPE such as adding langages or apps, follow this link, as I know, the default keyboard in WinPE is qwerty so you should add the azerty package if you have an azerty keyboard:

https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/winpe-mount-and-customize?view=windows-11


Then commit the changes and unmount the WinPE image, and create an USB bootable key containing your custom WinPE:

```
Dism /Unmount-Image /MountDir:"C:\WinPE_amd64\mount" /commit
MakeWinPEMedia /UFD C:\WinPE_amd64 F:                    (replace F with your usb letter)
```

use /discard instead of /commit if you don't want to save the changes you made

## Step 4: Capture the image

Take the 1rst computer again, run again the sysprep if the computer booted since step 2, otherwise you don't need to run it. Boot with the WinPE key, the script should run. Cancel it with ```[CTRL+C]``` if you don't want it to complete.

Use thoses commands to see which letters are assigned to which disks :
```
diskpart
list vol
exit
```

then you can run this command, D is the disk where the image will be downloaded and C the disk you want to capture:

```
dism /Capture-Image /ImageFile:D:\install.wim /CaptureDir:C:\ /Name:"Windows10-Pro"
```

Congratulations you have your custom Windows image !

## Step 5: Configuring netbootxyz

Netbootxyz uses iPXE, which allows us to boot via network. So we will use netbootxyz to deploy our custom WinPE and Windows 10 images.

