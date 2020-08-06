---
title: Deploying Android apps to Chromebooks
metadesc: Walkthrough the different ways developers can deploy their apps to Chromebooks, in order to debug and verify their app performance in the Chrome OS form factors.
date: 2020-06-06
weight: -6
---

Being able to run Android apps on a Chromebook is great, it gives users access to the vast Android ecosystem offerings and it gives Android developers the opportunity to reach Chrome OS users.

Developers should make a point to verify their apps on different form factors, since this will help improve users' experiences. This is why Chrome OS provides Android developers with the tools to deploy and test their apps on Chromebooks.

Whether developers are deploying their Android app directly from Chrome OS (using Android Studio in your Chromebook) or from another device, developers can use [ADB](https://developer.android.com/studio/command-line/adb) to deploy their apps and debug different interactions with the Chromebooks. For more details checkout the steps below.

## Enable ADB debug

Previously, using ADB on your Chromebook was only possible while in developer mode, which requires powerwashing the device and can reduce security. Luckily since Chrome 81, developers can keep their devices out of developer mode and still deploy apps they develop directly in Chrome OS, with the flip of a switch. Here is how:

First, make sure the Chromebook is not in [developer mode](https://chromium.googlesource.com/chromiumos/docs/+/master/developer_mode.md). Then go to settings and [turn on Linux](/{{locale.code}}/linux) (if you haven't done so before).

![Turn on Linux on Chrome OS](/images/android/deploy/turnon_linux.gif)

Once Linux is available open the Linux settings and you'll find a new option 'Develop Android apps', open that option.

Toggle enable ADB debugging and the computer will restart.

![Enable Chrome OS' ADB debug settings](/images/android/deploy/debug_settings.gif)

When the computer restarts you'll see a message that lets you know that there may be applications that were not downloaded from the app store on the device.

![This device may contain apps that haven't been verified by Google](/images/android/deploy/login_notice.jpg)

ADB is now available to deploy apps to your Chromebook, run debugging commands and interact directly with the device.

To ensure that your Android app works well on a variety of Chromebook devices and available form factors, Google recommends that you test your app on the following devices:

- An ARM-based Chromebook
- An x86-basedChromebook
- A device with a touchscreen and one without one
- A convertible device; that is, one that can change between a laptop and a tablet
- A device with a stylus

To view the full list of supported devices, see the [Chrome OS device support for apps](/{{locale.code}}/android/device-support) page.

## Deploy from Chrome OS

After enabling ADB debugging, you can load an Android app directly onto your Chrome OS device using one [Android Studio](/{{locale.code}}/develop/deploying-apps#deploy-with-android-studio) or if you have an APK you can [load it using the Terminal.](/{{locale.code}}/develop/deploying-apps#deploy-with-terminal)

### Deploy with Android Studio

With [Android Studio setup](/{{locale.code}}/en/linux/android/android-studio) and the
ADB setup above developers can push their apps to the Chromebook's Android container directly from Android Studio.  
The Chromebook will appear as an option in the device drop down:

![Android Studio devices dropdown](/images/android/deploy/as_devices.png)

Just push like any other Android device, you will see the authorization dialog and a window with your application running will start automatically after granting the auth.

![Deploy your app directly into Chrome OS](/images/android/deploy/run_app.gif)

That's it, you can now deploy the app to the Chromebook, test and debug _without_ the hassle of being in developer mode.

### Deploy with Terminal

If you haven't, install ADB:

```bash
sudo apt install adb
```

Connect to the device:

```bash
adb connect arc
```

A pop up is going to show asking for authorization for USB debugging, grant it.

![Authorization to connect to the device](/images/android/deploy/usb_dialog.png)

Install your app from the terminal:

```bash
adb install [path to your APK]
```

![Connect to the device through ADB in the terminal](/images/android/deploy/adb_connect.gif)

## Deploy from another device

If you can't use the method described above and need to push your app from another device, you have a couple of options: you can use [USB](#connect-to-adb-over-usb) or a [network address](#connect-to-adb-over-a-network) to connect
your device to ADB.

To push your APK from another device into the Chromebook, you must start your Chrome OS in [developer mode](https://www.chromium.org/chromium-os/poking-around-your-chrome-os-device) so that you can configure the Chromebook and push apps from the host machine. Follow this steps to get into [developer mode]({{locale.code}}/productivity/experimental-features#developer-mode)

!!! aside.message--warning
**Caution:** After switching your Chrome OS device to developer mode, it restarts
and clears all existing data on the device. The security level of the device is
also significantly reduced.
!!!

### Connect to ADB over USB

1. Make sure you [enabled ADB debugging.](/{{locale.code}}/develop/deploying-apps#enable-adb-debugging)
1. Determine if your device [supports USB debugging](https://www.chromium.org/chromium-os/chrome-os-systems-supporting-adb-debugging-over-usb)
1. Press [[Control]]+[[Alt]]+[[T]] to start the Chrome OS terminal.
1. Type `shell` to get to the bash command shell:

   ```bash
   crosh> shell
   chronos@localhost / $
   ```

1. Type the following commands to set up your device:

   ```bash
   $ sudo crossystem dev_enable_udc=1
   $ sudo reboot
   ```

1. After rebooting, open the terminal again and run the following command to enable ADB on the Chromebook's USB port:

   ```bash
   $ sudo ectool usbpd <port number> dr_swap
   ```

Use this command each time you disconnect and reconnect a USB cable. To ensure your Chromebook is in UFP mode, you can run `ectool usbpd <port number>`.

1. Plug in a USB cable to a [supported port](https://www.chromium.org/chromium-os/chrome-os-systems-supporting-adb-debugging-over-usb) on your device
2. Run `adb devices` from the Android SDK platform tools on your host machine to see your Chromebook listed as an ADB supported device
3. On your Chromebook, click **Allow** when prompted whether you want to allow the debugger. Your ADB session is established.

### Connect to ADB over a network { #adb-ip}

1. Make sure you [enabled ADB debugging.](/{{locale.code}}/develop/deploying-apps#enable-adb-debugging)

In order to debug over a network, you must configure the Chrome OS firewall to allow incoming ADB connections:

1.  Press [[Control]]+[[Alt]]+[[T]] to start the Chrome OS terminal.
1.  Type `shell` to get to the bash command shell:

    ```bash
    crosh> shell
    chronos@localhost / $
    ```

1.  Type the following commands to set up developer features and enable disk-write access for the firewall settings changes. If you need to enter a sudo password for the `chronos` user, you can (re)set one by running `chromeos-setdevpassword` at the [VT-2 prompt](https://www.chromium.org/chromium-os/poking-around-your-chrome-os-device#TOC-Get-the-command-prompt-through-VT-2) ([[Control]]+[[Alt]]+[[→]]); you'll need your root password.

    ```bash
    $ sudo crossystem dev_boot_signed_only=0
    $ sudo /usr/libexec/debugd/helpers/dev_features_rootfs_verification
    $ sudo reboot
    ```

1.  The `sudo reboot` command restarts your Chromebook. You can press the Tab key to enable autocompletion of file names. You must complete this procedure only once on your Chromebook.

After your device restarts, log in to your test account and type the following command to enable the secure shell and configure the firewall properly:

```bash
$ sudo /usr/libexec/debugd/helpers/dev_features_ssh
```

When the command completes, you can exit out of the shell.

Get the IP address of your Chromebook:

1.  Click the clock in the bottom-right area of the screen.
1.  Click the gear icon.
1.  Click the network type you are connected to (Wi-Fi or Mobile data) then the name of the network.
1.  Take note of the IP Address.

Connect to your Chromebook:

1.  Return to your development machine and use ADB to connect to your Chromebook using its IP address:

    ```bash
    adb connect <ip_address>:22
    ```

1.  On your Chromebook, click Allow when prompted whether you want to allow the debugger. Your ADB session is established.

#### Troubleshooting ADB debugging over a network

Sometimes the ADB device shows that it's offline when everything is connected properly. In this case, complete the following steps to troubleshoot the issue:

1.  Deactivate **ADB debugging** in _Developer options_.
2.  In a terminal window, run `adb kill-server`.
3.  Re-activate the **ADB debugging** option.
4.  In a terminal window, attempt to run `adb connect`.
5.  Click **Allow** when prompted whether you want to allow debugging. Your ADB session is established.