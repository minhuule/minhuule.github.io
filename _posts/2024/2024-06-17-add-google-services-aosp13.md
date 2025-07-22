---
title: Add Google Services to AOSP 13
date: 2024-06-17 15:12:00 +0800
categories: [Android Automotive]
tags: [google services, aosp, raspberry pi 4]
image: /assets/img/posts/google_services_aosp13/cover.png
---
## Google Services and AOSP
<div style="text-align: justify">
Android Open Source Project (AOSP) is the foundation of most Android devices. It provides the core operating system, but it does not include Google’s proprietary apps and services like the Play Store, Google Maps, or Firebase by default. These components, collectively known as Google Services, are licensed separately by Google. This blog will show you how to add Google Services to AOSP 13. I am using is Raspberry Pi 4.
</div>

## Google Services APKs
<div style="text-align: justify">
I choose MindTheGapps to download google package. It provides a minimal and reliable Google Apps (GApps) package tailored for custom ROM users, ensuring seamless integration of necessary Google services without unnecessary bloat.
</div>

Raspberry Pi 4 has arm64 architecture, so that I downloaded Android 13 Arm64 Google package from [mindthegapps.com](https://mindthegapps.com)

To use google services, we need at least the following APKs from downloaded google packages:
* GmsCore.apk: Provides essential Google services and APIs used by many apps (Google Play Services).
* GoogleServicesFramework.apk: Manages communication and syncing between the Android device and Google servers.
* Phonesky.apk: The Play Store app that lets users browse, install, and update Android apps.
* XML files to grant privileged permissions to Google apps:
  * privapp-permissions-google-product.xml
  * privapp-permissions-google-system-ext.xml

Put these APKs and an **Android.mk** in AOSP source tree: `<AOSP-source>/vendor/prebuilt/app/GoogleServices`
* GmsCore.apk
* GoogleServicesFramework.apk
* Phonesky.apk
* privapp-permissions-google-product.xml
* privapp-permissions-google-system-ext.xml
* Android.mk

`Android.mk` tells the AOSP build system how to package and include Google APKs and permission XMLs into the system image during the build process. Here is its content:
```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)
LOCAL_MODULE := privapp-permissions-google-product.xml
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_CLASS := ETC
LOCAL_MODULE_PATH := $(TARGET_OUT_PRODUCT_ETC)/permissions
LOCAL_SRC_FILES := $(LOCAL_MODULE)
LOCAL_PRODUCT_MODULE := true
include $(BUILD_PREBUILT)

include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE := GmsCore
LOCAL_MODULE_CLASS := APPS
LOCAL_PRIVILEGED_MODULE := true
LOCAL_CERTIFICATE := PRESIGNED
LOCAL_MODULE_PATH := $(TARGET_OUT)/priv-app
LOCAL_SRC_FILES := GmsCore.apk
LOCAL_REQUIRED_MODULES := privapp-permissions-google-product.xml
include $(BUILD_PREBUILT)

include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE := Phonesky
LOCAL_MODULE_CLASS := APPS
LOCAL_PRIVILEGED_MODULE := true
LOCAL_CERTIFICATE := PRESIGNED
LOCAL_MODULE_PATH := $(TARGET_OUT)/priv-app
LOCAL_SRC_FILES := Phonesky.apk
LOCAL_REQUIRED_MODULES := privapp-permissions-google-product.xml
include $(BUILD_PREBUILT)

include $(CLEAR_VARS)
LOCAL_MODULE := privapp-permissions-google-system-ext.xml
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_CLASS := ETC
LOCAL_MODULE_PATH := $(TARGET_OUT_PRODUCT_ETC)/permissions
LOCAL_SRC_FILES := $(LOCAL_MODULE)
LOCAL_PRODUCT_MODULE := true
include $(BUILD_PREBUILT)

include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE := GoogleServicesFramework
LOCAL_MODULE_CLASS := APPS
LOCAL_PRIVILEGED_MODULE := true
LOCAL_CERTIFICATE := PRESIGNED
LOCAL_MODULE_PATH := $(TARGET_OUT)/priv-app
LOCAL_SRC_FILES := GoogleServicesFramework.apk
LOCAL_REQUIRED_MODULES := privapp-permissions-google-system-ext.xml
include $(BUILD_PREBUILT)
```

Then define the apps in `<AOSP-source>/device/brcm/rpi4/device.mk`
```
# Google Services core apk
PRODUCT_PACKAGES += \
    GoogleServicesFramework \
    GmsCore \
    Phonesky
```

## Google Maps and Device ID
Now install the following apps:

### 1. Google Maps
A google app, used to prove that google services is installed.

Download an **arm64 google maps apk** and place it in `<AOSP-source>/vendor/prebuilt/app/GoogleMaps/GoogleMaps.apk`

Then create an `Android.mk` at the same place (`<AOSP-source>/vendor/prebuilt/app/GoogleMaps/Android.mk`) with the following content:
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := GoogleMaps
LOCAL_SRC_FILES := GoogleMaps.apk
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_CERTIFICATE := PRESIGNED
include $(BUILD_PREBUILT)
```

### 2. Device ID
This app is used to collect Google Service Framework ID. This ID can be used to register the android device with Google to allow it to use Google Services.

Device ID apk can be downloaded from the internet **deviceid.apk**, place it at `<AOSP-source>/vendor/prebuilt/app/DeviceId/DeviceId.apk`

And create an Android.mk in `<AOSP-source>/vendor/prebuilt/app/DeviceId/Android.mk`
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := DeviceId
LOCAL_SRC_FILES := DeviceId.apk
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_CERTIFICATE := PRESIGNED
include $(BUILD_PREBUILT)
```

Finally add Google Maps and Device ID to `<AOSP-source>/device/brcm/rpi4/device.mk`
```
# Prebuilt app apk
PRODUCT_PACKAGES += \
    DeviceId \
    GoogleMaps
```

Now build and boot AOSP on the device.

## Register Android device with Google
To use Google apps and services we have to register device with google. For more detail, read [https://www.google.com/android/uncertified](https://www.google.com/android/uncertified)

After booting, first **connect to wifi**.

Go to **Settings -> Google**
![](/assets/img/posts/google_services_aosp13/register1.png)

Now we can see that **the device isn’t Play Protect certified**.
![](/assets/img/posts/google_services_aosp13/register2.png)

Wait for a few minutes, open **Device ID** app.
![](/assets/img/posts/google_services_aosp13/register3.png)

You have to see a hex string Google Service Framework (GSF). That’s the string we use to register in [https://www.google.com/android/uncertified](https://www.google.com/android/uncertified)
![](/assets/img/posts/google_services_aosp13/register4.png)

Click on **Google Service Framework (GSF)** to copy that string.

Go to **Search**.
![](/assets/img/posts/google_services_aosp13/register5.png)

Search for **android uncertified**.
![](/assets/img/posts/google_services_aosp13/register6.png)

Paste above string and **register**.
![](/assets/img/posts/google_services_aosp13/register7.png)

Now wait for about 15 – 20 minutes, the warning **the device isn’t Play Protect certified** will disappear and you can sign in to **Google** in **Settings**
![](/assets/img/posts/google_services_aosp13/register8.png)

Now you can use Google Apps and Services, for example youtube, google maps, google play, …
![](/assets/img/posts/google_services_aosp13/register9.png)

That’s all about installing google services to AOSP 13 on Raspberry Pi 4. 

Give it a try and send me some feedback. Thank you !!