# LenovoSmartClock2
Root and configure a Lenovo Smart Clock 2 for use as a smart home display

This guide focuses on Ubuntu 23.04.

I recommend using Ubuntu over Windows purely because there are no shifty third-party-drivers-from-untrustworthy-website shenanigans required to get the clock recognised in Ubuntu.

I put together this guide as a product of following many guides and trying different things, seeing what works and what doesn't, and trying to rectify those problems. Most noteably the outdated webview component was causing issues when trying to use the HomeAssistant companion app, so I needed to find a way to install a more modern webview. [HomeAssistant also does not play well with Bromite](https://community.home-assistant.io/t/issue-with-bromite-webview-and-reverse-proxy/316393/3?u=thomasprior), so existing guides to install that were not of use to me.

# Pre-requisites

- A USB cable
- Preci-Dip or POGO pins (https://www.mouser.co.uk/ProductDetail/437-813S100810016101)
- Soldering iron and solder

#### Optional but highly recommended: 

- Print a stand to ensure the pins make contact with the correct spots on the clock. (https://www.printables.com/model/450827-lenovo-smart-clock-2-diy-programming-pogo-pin-dock/files)
- A bluetooth keyboard

#### Making the cable

1. Cut one end from a USB A cable (I used a USB-A to Micro-B cable). Ensure that it is a data cable before proceeding, a charging-only cable will not work!
2. Solder the pogo pins to match the following picture:

 ⚠️⚠️ Soldering this incorrectly or placing the data pins in contact with the power positions on the device can cause serious damage to your device. Be sure you are connecting the right pins to the right cables! ⚠️⚠️

![image](https://github.com/ThomasPrior/LenovoSmartClock2/assets/34111848/ac314a92-8c29-4dab-bbca-5e357bd118a4)

(Photo credit bj00rn @ XDA Developers)

AS SHOWN IN THE PICTURE ABOVE, with the wide side of the case horizontal and the connector at upper left, assuming correct usage of colours inside the USB cable:

| DP \/ Green | DM \/ White |
| ---      | ---       |
| GND \/ Black | GND \/ Black |
| VBUS \/ Red | N\/A |
| N\/A |	N\/A |

Pin arrangements courtesy of cmccambridge @ XDA Developers

#### Install Android Platform Tools

```sudo apt-get install android-sdk-platform-tools```

## Install and set up the mtclient environment (via https://github.com/bkerler/mtkclient)

#### Install Python, Git and other dependencies for mtclient

```sudo apt install python3 git libusb-1.0-0 python3-pip python3-virtualenv```

#### Grab the mtkclient project files, build and install

```
cd ~
git clone https://github.com/bkerler/mtkclient
cd mtkclient
python3 -m venv .
source bin/activate
pip3 install -r requirements.txt
python3 setup.py build
python3 setup.py install
```

#### Install rules

```
sudo usermod -a -G plugdev $USER
sudo usermod -a -G dialout $USER
sudo cp Setup/Linux/*.rules /etc/udev/rules.d
sudo udevadm control -R
```

# Get the stock boot.img from the device
!! This is needed to recover your device if anything goes wrong !!

1. Send the command below. The MTK client will begin to scan for the device

```python mtk r boot,vbmeta boot.img,vbmeta.img```

2. Whilst holding ```Volume +``` and ```Volume -``` and with the USB cable attached, insert the power cable for the clock
3. The ```mtkclient``` software should detect the device and dump the partitions we specified.
4. Reset the mtkclient connection to the device:

```python mtk reset``` 

5. Power off the clock then power it back on again.

#### Download the Magisk APK file for use on the clock

Magisk-v26.1.apk (https://github.com/topjohnwu/Magisk/releases/tag/v26.1)

# Set up ADB access

#### Turn on the Screen Reader

1. Set the device up from the Google Home app until the clock face is shown on the device
2. Open the Google Home app and open the Devices tab at the bottom
3. Select your smart clock in the list of available devices
4. Press the settings icon in the top right hand corner
5. Open the Accessibility menu and enable Screen reader.

# Use the calendar to download the launcher

This general process is shown very well in this video by Cameron Gray (https://youtu.be/uSHpvbbvz7Q?t=444)
My process which differs slightly is written below: </summary>

##### Prepare a special calendar entry

1. With the account you are using to set up the device, open Google Calendar
2. Create a new event in the future with the title consisting only of ```https://blakadder.com/assets/files/ultra-small-launcher.apk```

1. Ask the clock to show upcoming events
2. With the event on screen, swipe sideways until the clock reads out the title of the event. (```https://blakadder.com/assets/files/ultra-small-launcher.apk```)
3. Draw a "L" shape on the screen to open the TalkBack menu
4. Swipe sideways until "Copy last utterance to clipboard" is selected, then tap twice to select it.
5. Draw a "L" shape on the screen again to open the TalkBack menu
6. Swipe right on the screen until "Open Talkback settings" is selected, then tap twice to select it.
7. You can disable Screen Reader via the Google Home app on your phone now, we won't be needing it again for a while.
8. Scroll to the bottom of the list. Near the end is "Privacy policy". Tap it.
9. In the Browser window that opens, allow the permission prompts. We need to be able to download files using this browser, so storage access is necessary.
10. Tap the address bar. This will select the URL. Tap again in the address bar and press the ```X``` button to clear the address bar
11. Long press in the address bar and paste the URL in.
12. Long press the URL again and press "Open"
13. The launcher will now download and ou will be taken to the Downloads folder.
14. Click the downloaded file to install the APK
15. Follow the prompts to allow the Browser to install the APK.
16. When prompted, change the default launcher to the launcher we just installed.

# Connect a bluetooth keyboard

1. Oprn the settings app from the home screen
2. Pair your bluetooth keyboard in the normal way
3. ```Windows + Enter``` or ```Ctrl + Esc``` will simulate the "home" button

# Enable developer options

1. Open the settings app from the home screen
2. Select "about phone"
3. Find the build number and tap on it 7 times until it says "you are now a developer"
4. Go back to the main Settings menu and select System.
5. Select "Developer Options"
6. Enable "USB debugging"

# Prepare the new boot.img file

1. Install the magisk app:

```adb install Magisk-v26.1.apk```

2. Push the extracted ```boot.img``` file to the device:

```adb push boot.img /sdcard/Download```

3. Open the Magisk app and press "Install".
4. Select the option "Select and Patch a File"
5. Select the Downloads folder, and then the boot.img we sent to the device earlier.
6. Allow the process to complete, this may take a few minutes.
7. Rename this new file so we don't get it confused with the original one:

```adb shell mv /sdcard/Download/[displayed magisk patched boot filename here] /sdcard/Download/boot.patched```

8. Pull the patched ```boot.patched``` file back to yout computer:

```adb pull /sdcard/Download/boot.patched```

#### Unlock the bootloader

1. Erase metadata and userdata:

```python mtk e metadata,userdata,md_udc```

2. Unlock bootloader:

```python mtk da seccfg unlock```

3. Reset the mtkclient connection:

```python mtk reset```

4. Reboot the clock by removing the power cable and plugging it back in.

##Flash the patched boot image

1. Flash the boot.patched file to the clock:

```python mtk w boot,vbmeta boot.patched,vbmeta.img.empty```

2. Reset the mtkclient connection:

```python mtk reset```

3. Reboot the clock by removing the power cable and plugging it back in.

# 🎉 THE DEVICE IS NOW ROOTED! 🎉

You may stop here if that's all you were intending to do. This method of root survives reboots and factory resets.


#### Install updated WebView component

I used the following files to install a more modern WebView:

LSPosed_1.8.6.apk
AnyWebView_1.2_experimental.apk (https://github.com/neoblackxt/AnyWebView/releases/tag/v1.2)
LSPosed-v1.8.6-6712-zygisk-release.zip (https://github.com/LSPosed/LSPosed/releases/tag/v1.8.6)
com.google.android.webview_114.0.5735.147-573514700_minAPI24_maxAPI28(armeabi-v7a)(nodpi).com.apk

#### Enable Zygisk

1. Open the Magisk app and open the settings tab
2. Enable Zygisk
3. Reboot if prompted

#### Install LSPosed, AnyWebView and a modern WebView component

1. Push the LSPosed files to the clock

```
adb push LSPosed-v1.8.6-6712-zygisk-release.zip /sdcard/Download
adb install LSPosed_1.8.6.apk
```

2. Open Magisk and select the modules tab
3. Select "Install from storage"
4. Navigate to the Download folder and select ```LSPosed-v1.8.6-6712-zygisk-release.zip```
5. Install the APK files for the new WebView component and AnyWebView:

```
adb install  --user 0 com.google.android.webview_114.0.5735.147-573514700_minAPI24_maxAPI28(armeabi-v7a)(nodpi)_apkmirror.com.apk
adb install --user 0 AnyWebView_1.2_experimental.apk
```

6. Reboot the device
7. Open the LSPosed app and open the modules section
8. Enable AnyWebView, selecting the webview you want to be enabled before you exit the module setup
9. Reboot the device.
10. Open the settings app, then navigate to Developer Options
11. Open "WebView implementation" and select the newly installed WebView component.

## Install Home Asssistant Companion and set it to auto launch

1. Download the Home Assistant Companion APK from F-Droid (https://f-droid.org/en/packages/io.homeassistant.companion.android.minimal/)
2. Install the Home Assistant APK on your device:
	
```adb install io.homeassistant.companion.android.minimal_[version].apk```

3. Download the MacroDroid APK and install it on your device:

```adb install MacroDroid - Device Automation_5.35.11.apk```

4. Grant both of these applications the "Display over other apps" permission:
!! This is important if you want to use Home Assistant notification commands to control the device. MacroDroid requires this permission to launch Home Assistant on boot.

```
adb shell pm grant io.homeassistant.companion.android.minimal android.permission.SYSTEM_ALERT_WINDOW
adb shell pm grant com.arlosoft.macrodroid android.permission.SYSTEM_ALERT_WINDOW
```

5. Open MacroDroid and set up a macro as follows:

 - Trigger > Device Events > Device Boot
 - Actions > Applications > Launch Application > Select Application > HomeAssistant
 - Optional: Force new > OK
 - Set macro name

6. Set up Home Assistant to your liking.

Upon reboot, your device will now open Home Assistant immediately after the device has fully booted. The launcher may show briefly. 

## Enable Wireless ADB

Follow the instructions at https://github.com/mrh929/magisk-wifiadb

## Optional: Remove built in apps

### Remove built in apps
 - Install and enable Magisk module https://github.com/sunilpaulmathew/De-Bloater
 - Rebbot
 - Open De-Bloater app and select all com.google.assistant.* and com.lenovo.* and apks to your liking.
 - Reboot

### Restore built in apps

**Option 1**
  - Open De-bloater app
  - Restore debloated apps
  - Reboot

**Option 2**
  - Disable Magisk De-Bloater module
  - Reboot

## Optional: Add navigation bar

  - Download the [VirtualSoftKeys app from F-Droid](https://f-droid.org/en/packages/tw.com.daxia.virtualsoftkeys/)
  - Grant the "display over other apps" permission

    ```adb shell pm grant tw.com.daxia.virtualsoftkeys android.permission.SYSTEM_ALERT_WINDOW```

  - Open the app, follow the on screen prompts to configure the nav bar to your liking.

## Optional: Install microG services

  - Push microG Installer Revived to sdcard

    ```adb push microG_Installer_Revived.zip /sdcard/Download```

  - Push LSPosed framework to sdcard

    ```adb push LSPosed-v1.8.6-6712-zygisk-release.zip /sdcard/Download```

  - Install the LSPosed app:

    ```adb install LSPosed_1.8.6.apk```

  - Open the Magisk app and enable Zygisk from Magisk's settings.
  - Reboot
  - Open the Magisk app and install both modules from storage.
  - Reboot
  - Open the LSPosed app
  - Follow the steps to enable signature spoofing: https://github.com/microg/GmsCore/wiki/Signature-Spoofing
  - Push the FakeGapps package to sdcard https://github.com/whew-inc/FakeGApps/releases

    ```adb push app-release.zip /sdcard/Download```

  - Open LSPosed and enable the FakeGapps modules
  - Reboot
  - Open microG settings and observe whether the self-check passes. All checks should now pass if done correctly.

  🎉 You can now add a Google account, register the device, enroll in cloud messaging and run apps that rely on the gApps package like Youtube Music.
