---
title: 'Bug hunting on Android apps'
published: 2025-09-23
draft: false
tags: ['android', 'frida']
toc: true
coverImage:
  src: './android.png'
  alt: 'Lain with phone'
---

This guide provides a concise cheatsheet to set up the Android Emulator on Linux (and possibly macOS), gain root access, and configure Frida to bypass SSL pinning of an Android application.
# Install frida-tool
```bash
pip install --upgrade frida-tools frida 
```
# Setting up Emulator
1. Download and install Android studio, run it and navigate to AVD to create an emulator. I prefer using Pixel 5. Select the minimum SDK version (Google Play or Open Source) required for your APK:
```bash
aapt dump badging com.app.android.apk | grep sdkVersion
```
2. Choose either x86 or x86_64 - one might work for a particular application, while the other might not. Same for SDK versions. Do not expect any application to run successfully.

If the emulator won't run, try:
```bash
sudo modprobe kvm_intel kvm kvm_amd
```
## Warning
**Pay attention to which SDK version and arch you use in the next commands**
## 1) Rooting Emulator
1. Run in your terminal:
```bash
git clone https://gitlab.com/newbit/rootAVD.git/ && cd rootAVD
./rootAVD.sh | grep 30
./rootAVD.sh system-images/android-30/google_apis/x86/ramdisk.img
```
3. Run Emulator again
4. Open Magisk, allow notifications and click Ok to reboot
5. Run Magisk again and update it if asked
## 2) Install frida on Emulator
1. Check frida version on your system:
```bash
frida --version                                                               
16.7.14
```
2. Download corresponding version of magisk-frida from https://github.com/ViRb3/magisk-frida/releases/
3. Transfer zip file to the device:
```bash
adb push ~/Downloads/MagiskFrida-16.7.14-1.zip /sdcard/
```
4. Open Magisk on Emulator, click Module and select `Install from Storage`, navigate to `sdk_hphone64_x86_64` and select the archive
5. Wait and click Reboot
6. Disable protection (not sure if it's necessary):
```bash
adb shell su -c setenforce 0
```
5. Import Burp cert from proxy options and upload to the device:
```bash
adb push ~/cert-der.crt /data/local/tmp/cert-der.crt
```
6. Ensure `frida-server` is running:
```bash
adb shell
ps -A | grep frida
```
7. Check if it's working on host machine:
```bash
frida-ps -U
```
# Installing and running apk file
1. Set up Burp proxy listener on `0.0.0.0:8081`
2. Run Emulator with proxy:
 ```bash
 emulator -avd "Pixel_5" -writable-system -http-proxy 127.0.0.1:8081
 ```
3. Install your apk which you downloaded from APK Pure or APK Mirror:
```bash
adb install ./com.name.apk
# or if its unpacked .xapk
adb install-multiple ./com.name.apk config.arm64_v8a.apk config.xxxhdpi.apk 
```
2. Start your App
3. Run frida-script on host machine:
```bash
frida -U --codeshare sowdust/universal-android-ssl-pinning-bypass-2 -n <app name from frida-ps list> 
```
Another frida scripts if the previous fails:
```bash
frida -U --codeshare pcipolloni/universal-android-ssl-pinning-bypass-with-frida -n AppName
frida -U --codeshare akabe1/frida-multiple-unpinning -n AppName
```
