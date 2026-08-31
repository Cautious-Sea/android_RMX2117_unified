# Build ROM Changes
Error/Change (No): 1
What is the Error: The device tree is configured for Cipher OS instead of LineageOS, prebuilt kernel is used instead of source, and vendor/hardware repos are not linked.
The Fix: Renamed product makefile to lineage_RMX2117, updated AndroidProducts.mk and lineage_RMX2117.mk for LineageOS. Removed prebuilt kernel flags in BoardConfig.mk and added source build flags. Added device/mediatek/sepolicy_vndr to BOARD_VENDOR_SEPOLICY_DIRS and SEPolicy.mk to device.mk. Included hardware/mediatek, hardware/oplus, vendor/mediatek/ims in PRODUCT_SOONG_NAMESPACES.
```diff
diff --git a/AndroidProducts.mk b/AndroidProducts.mk
index d04f8dd..3df72f5 100644
--- a/AndroidProducts.mk
+++ b/AndroidProducts.mk
@@ -15,9 +15,10 @@
 #
 
 PRODUCT_MAKEFILES := \
-    $(LOCAL_DIR)/cipher_RMX2117.mk
+    $(LOCAL_DIR)/lineage_RMX2117.mk
 
 COMMON_LUNCH_CHOICES := \
-    cipher_RMX2117-user \
-    cipher_RMX2117-userdebug \
-    cipher_RMX2117-eng
+    lineage_RMX2117-user \
+    lineage_RMX2117-bp4a-userdebug \
+    lineage_RMX2117-userdebug \
+    lineage_RMX2117-eng
diff --git a/BoardConfig.mk b/BoardConfig.mk
index ef0b36c..67ef959 100644
--- a/BoardConfig.mk
+++ b/BoardConfig.mk
@@ -63,7 +63,9 @@ BOARD_INCLUDE_RECOVERY_DTBO := true
 BOARD_INCLUDE_DTB_IN_BOOTIMG := true
 
 BOARD_KERNEL_IMAGE_NAME := Image.gz
-TARGET_PREBUILT_KERNEL := $(DEVICE_PATH)/prebuilt/kernel
+# TARGET_PREBUILT_KERNEL := $(DEVICE_PATH)/prebuilt/kernel
+TARGET_KERNEL_SOURCE := kernel/oplus/mt6853
+TARGET_KERNEL_CONFIG := mo-mt6853_defconfig
 BOARD_PREBUILT_DTBOIMAGE := $(DEVICE_PATH)/prebuilt/dtbo.img
 BOARD_PREBUILT_DTBIMAGE_DIR := $(DEVICE_PATH)/prebuilt/dtb
 TARGET_PREBUILT_DTB := $(DEVICE_PATH)/prebuilt/dtb/mt6853.dtb
@@ -142,6 +144,7 @@ TARGET_FS_CONFIG_GEN := $(DEVICE_PATH)/config.fs
 # Sepolicy
 TARGET_USES_PREBUILT_VENDOR_SEPOLICY := true
 TARGET_HAS_FUSEBLK_SEPOLICY_ON_VENDOR := true
+BOARD_VENDOR_SEPOLICY_DIRS += device/mediatek/sepolicy_vndr
 SYSTEM_EXT_PRIVATE_SEPOLICY_DIRS := $(DEVICE_PATH)/sepolicy/private
 SELINUX_IGNORE_NEVERALLOWS := true
 
diff --git a/device.mk b/device.mk
index b667ef8..b863794 100644
--- a/device.mk
+++ b/device.mk
@@ -21,6 +21,9 @@ $(call inherit-product, $(SRC_TARGET_DIR)/product/developer_gsi_keys.mk)
 # Inherit Vendor Blobs
 $(call inherit-product, vendor/realme/RMX2117/RMX2117-vendor.mk)
 
+# Inherit SEPolicy
+$(call inherit-product, device/mediatek/sepolicy_vndr/SEPolicy.mk)
+
 # Enable updating of APEXes
 $(call inherit-product, $(SRC_TARGET_DIR)/product/updatable_apex.mk)
 
@@ -34,7 +37,10 @@ PRODUCT_AAPT_PREF_CONFIG := xxhdpi
 
 # Soong namespaces
 PRODUCT_SOONG_NAMESPACES += \
-    $(DEVICE_PATH)
+    $(DEVICE_PATH) \
+    hardware/mediatek \
+    hardware/oplus \
+    vendor/mediatek/ims
 
 # Dynamic Partition
 PRODUCT_USE_DYNAMIC_PARTITIONS := true
diff --git a/cipher_RMX2117.mk b/lineage_RMX2117.mk
similarity index 100%
rename from cipher_RMX2117.mk
rename to lineage_RMX2117.mk
index 26b277e..bbe559a 100644
--- a/cipher_RMX2117.mk
+++ b/lineage_RMX2117.mk
@@ -22,14 +22,14 @@ $(call inherit-product, $(SRC_TARGET_DIR)/product/non_ab_device.mk)
 # Inherit from RMX2117 device
 $(call inherit-product, device/realme/RMX2117/device.mk)
 
-# Inherit some common cipherOS stuff.
-$(call inherit-product, vendor/cipher/config/common_full_phone.mk)
+# Inherit some common LineageOS stuff.
+$(call inherit-product, vendor/lineage/config/common_full_phone.mk)
 
 # Boot Animation
 TARGET_BOOT_ANIMATION_RES := 1080
-CIPHER_MAINTAINER := TechyMinati
+
 # Device identifier. This must come after all inclusions.
-PRODUCT_NAME := cipher_RMX2117
+PRODUCT_NAME := lineage_RMX2117
 PRODUCT_DEVICE := RMX2117
 PRODUCT_BRAND := realme
 PRODUCT_MODEL := realme Narzo 30 Pro 5G
```
Error/Change (No): 2
What is the Error: SEPolicy.mk was mistakenly included via $(call inherit-product, ...) in device.mk, but it should be included directly in BoardConfig.mk like device_oplus_op6893 does.
The Fix: Removed $(call inherit-product, device/mediatek/sepolicy_vndr/SEPolicy.mk) from device.mk and added include device/mediatek/sepolicy_vndr/SEPolicy.mk in BoardConfig.mk.
```diff
diff --git a/BoardConfig.mk b/BoardConfig.mk
index 67ef959..e92a297 100644
--- a/BoardConfig.mk
+++ b/BoardConfig.mk
@@ -145,6 +145,7 @@ TARGET_FS_CONFIG_GEN := $(DEVICE_PATH)/config.fs
 TARGET_USES_PREBUILT_VENDOR_SEPOLICY := true
 TARGET_HAS_FUSEBLK_SEPOLICY_ON_VENDOR := true
 BOARD_VENDOR_SEPOLICY_DIRS += device/mediatek/sepolicy_vndr
+include device/mediatek/sepolicy_vndr/SEPolicy.mk
 SYSTEM_EXT_PRIVATE_SEPOLICY_DIRS := $(DEVICE_PATH)/sepolicy/private
 SELINUX_IGNORE_NEVERALLOWS := true
 
diff --git a/device.mk b/device.mk
index b863794..875704d 100644
--- a/device.mk
+++ b/device.mk
@@ -21,9 +21,6 @@ $(call inherit-product, $(SRC_TARGET_DIR)/product/developer_gsi_keys.mk)
 # Inherit Vendor Blobs
 $(call inherit-product, vendor/realme/RMX2117/RMX2117-vendor.mk)
 
-# Inherit SEPolicy
-$(call inherit-product, device/mediatek/sepolicy_vndr/SEPolicy.mk)
-
 # Enable updating of APEXes
 $(call inherit-product, $(SRC_TARGET_DIR)/product/updatable_apex.mk)
 
```
Error/Change (No): 3
What is the Error: vendor/mediatek/ims was not included in the build system despite being in PRODUCT_SOONG_NAMESPACES.
The Fix: Inherited vendor/mediatek/ims/ims.mk in device.mk under the IMS section.
```diff
diff --git a/device.mk b/device.mk
index 875704d..253579e 100644
--- a/device.mk
+++ b/device.mk
@@ -116,8 +116,8 @@ PRODUCT_PACKAGES += \
 PRODUCT_PACKAGES += \
     libsuspend
 
-
 # IMS
+$(call inherit-product, vendor/mediatek/ims/ims.mk)
 PRODUCT_BOOT_JARS += \
     mediatek-common \
     mediatek-framework \
```
Error/Change (No): 4
What is the Error: Missing product packages for oplus and mediatek.
The Fix: Added sensors.oplus to PRODUCT_PACKAGES and inherited hardware/mediatek/frameworks/mediatek-frameworks.mk in device.mk.
```diff
diff --git a/device.mk b/device.mk
index 253579e..b9030b7 100644
--- a/device.mk
+++ b/device.mk
@@ -58,6 +58,10 @@ PRODUCT_PACKAGES += \
     init.mt6853.rc \
     fstab.mt6853
 
+# Sensors
+PRODUCT_PACKAGES += \
+    sensors.oplus
+
 # Overlays
 DEVICE_PACKAGE_OVERLAYS += \
     $(DEVICE_PATH)/overlay
@@ -131,3 +135,6 @@ PRODUCT_BOOT_JARS += \
 # MTK
 PRODUCT_PACKAGES += \
     MtkInCallService
+
+# Mediatek frameworks
+$(call inherit-product, hardware/mediatek/frameworks/mediatek-frameworks.mk)
```
Error/Change (No): 5
What is the Error: COMMON_LUNCH_CHOICES did not consistently follow the lineage_RMX2117-bp4a-* format.
The Fix: Updated lunch choices to lineage_RMX2117-bp4a-user, lineage_RMX2117-bp4a-userdebug, and lineage_RMX2117-bp4a-eng in AndroidProducts.mk.
```diff
diff --git a/AndroidProducts.mk b/AndroidProducts.mk
index 3df72f5..ad5cb3f 100644
--- a/AndroidProducts.mk
+++ b/AndroidProducts.mk
@@ -18,7 +18,6 @@ PRODUCT_MAKEFILES := \
     $(LOCAL_DIR)/lineage_RMX2117.mk
 
 COMMON_LUNCH_CHOICES := \
-    lineage_RMX2117-user \
+    lineage_RMX2117-bp4a-user \
     lineage_RMX2117-bp4a-userdebug \
-    lineage_RMX2117-userdebug \
-    lineage_RMX2117-eng
+    lineage_RMX2117-bp4a-eng
```
Error/Change (No): 6
What is the Error: module "MtkInCallService" variant "android_common": found in multiple namespaces(device/realme/RMX2117 and hardware/mediatek) when including in system partition
The Fix: Removed duplicate MtkInCallService from device/realme/RMX2117/app/InCallService since LineageOS has moved it to the common hardware/mediatek repository.
```diff
diff --git a/app/InCallService/Android.bp b/app/InCallService/Android.bp
deleted file mode 100644
index 649bdc0009cd..000000000000
--- a/app/InCallService/Android.bp
+++ /dev/null
@@ -1,31 +0,0 @@
-/*
- * Copyright (C) 2022 bengris32
- * Copyright (C) 2022 LineageOS
- *
- * Licensed under the Apache License, Version 2.0 (the "License");
- * you may not use this file except in compliance with the License.
- * You may obtain a copy of the License at
- *
- * http://www.apache.org/licenses/LICENSE-2.0
- *
- * Unless required by applicable law or agreed to in writing, software
- * distributed under the License is distributed on an "AS IS" BASIS,
- * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
- * See the License for the specific language governing permissions and
- * limitations under the License.
- */
-
-android_app {
-    name: "MtkInCallService",
-
-    srcs: ["src/**/*.java"],
-    resource_dirs: ["res"],
-
-    certificate: "platform",
-    platform_apis: true,
-    privileged: true,
-
-    optimize: {
-        enabled: false,
-    }
-}
diff --git a/app/InCallService/AndroidManifest.xml b/app/InCallService/AndroidManifest.xml
deleted file mode 100644
index cb5c2bc4040f..000000000000
--- a/app/InCallService/AndroidManifest.xml
+++ /dev/null
@@ -1,25 +0,0 @@
-<?xml version="1.0" encoding="utf-8"?>
-<manifest xmlns:android="http://schemas.android.com/apk/res/android"
-    package="org.lineageos.mediatek.incallservice"
-    android:versionCode="1"
-    android:versionName="1.0"
-    android:sharedUserId="android.uid.system">
-
-    <application
-        android:label="@string/app_name"
-        android:persistent="true">
-        <receiver
-            android:directBootAware="true"
-            android:exported="true"    
-            android:name="org.lineageos.mediatek.incallservice.OnLockedBootCompleteReceiver">
-            <intent-filter>
-                <action android:name="android.intent.action.LOCKED_BOOT_COMPLETED" />
-                <category android:name="android.intent.category.DEFAULT" />
-            </intent-filter>
-        </receiver>
-        <service
-            android:directBootAware="true"
-            android:name="org.lineageos.mediatek.incallservice.VolumeChangeService">
-        </service>
-    </application>
-</manifest>
diff --git a/app/InCallService/res/values/strings.xml b/app/InCallService/res/values/strings.xml
deleted file mode 100644
index 2be002f1d017..000000000000
--- a/app/InCallService/res/values/strings.xml
+++ /dev/null
@@ -1,4 +0,0 @@
-<?xml version="1.0" encoding="utf-8"?>
-<resources>
-    <string name="app_name">Mediatek In-Call Service</string>
-</resources>
diff --git a/app/InCallService/src/org/lineageos/mediatek/incallservice/CallStateListener.java b/app/InCallService/src/org/lineageos/mediatek/incallservice/CallStateListener.java
deleted file mode 100644
index 5c42a91940ef..000000000000
--- a/app/InCallService/src/org/lineageos/mediatek/incallservice/CallStateListener.java
+++ /dev/null
@@ -1,31 +0,0 @@
-/*
- * Copyright (C) 2023 The LineageOS Project
- *
- * SPDX-License-Identifier: Apache-2.0
- */
-
-package org.lineageos.mediatek.incallservice;
-
-import android.media.AudioManager;
-
-import android.telephony.TelephonyManager;
-import android.telephony.TelephonyCallback;
-
-import android.util.Log;
-
-public class CallStateListener extends TelephonyCallback implements TelephonyCallback.CallStateListener {
-    private static final String LOG_TAG = "MtkInCallService";
-    private AudioManager mAudioManager;
-
-    public CallStateListener(AudioManager audioManager) {
-        mAudioManager = audioManager;
-    }
-
-    @Override
-    public void onCallStateChanged(int state) {
-        if (state == TelephonyManager.CALL_STATE_OFFHOOK || state == TelephonyManager.CALL_STATE_RINGING) {
-            Log.d(LOG_TAG, "CallStateListener: CALL_STATE_OFFHOOK, setting gain.");
-            GainUtils.setGainLevel(mAudioManager.getStreamVolume(AudioManager.STREAM_VOICE_CALL));
-        }
-    }
-}
diff --git a/app/InCallService/src/org/lineageos/mediatek/incallservice/GainUtils.java b/app/InCallService/src/org/lineageos/mediatek/incallservice/GainUtils.java
deleted file mode 100644
index 482717476019..000000000000
--- a/app/InCallService/src/org/lineageos/mediatek/incallservice/GainUtils.java
+++ /dev/null
@@ -1,31 +0,0 @@
-package org.lineageos.mediatek.incallservice;
-
-import android.media.AudioDeviceInfo;
-import android.os.SystemProperties;
-import android.media.AudioSystem;
-import android.util.Log;
-
-public class GainUtils {
-    public static final String LOG_TAG = "MediatekInCallService";
-
-    public static void setGainLevel(int audioDevice, int gainIndex, int streamType) {
-        int maxStep = SystemProperties.getInt("ro.config.vc_call_vol_steps", 7);
-        String parameters = String.format("volumeDevice=%d;volumeIndex=%d;volumeStreamType=%d",
-                                          audioDevice, 
-                                          Math.round(
-                                            (15.0 / Math.log(maxStep + 1.0))
-                                            * Math.log(Math.min(maxStep, gainIndex) + 1.0)),
-                                          streamType);
-        Log.d(LOG_TAG, "Setting audio parameters to: " + parameters);
-        AudioSystem.setParameters(parameters);
-    }
-
-    /**
-     * Sets the gain level for built-in earpiece and bluetooth SCO devices.
-     * @param gainIndex The gain level to set.
-     */
-    public static void setGainLevel(int gainIndex) {
-        GainUtils.setGainLevel(AudioDeviceInfo.TYPE_BUILTIN_EARPIECE, gainIndex, AudioSystem.STREAM_VOICE_CALL);
-        GainUtils.setGainLevel(AudioDeviceInfo.TYPE_BLUETOOTH_SCO, gainIndex, AudioSystem.STREAM_VOICE_CALL);
-    }
-}
diff --git a/app/InCallService/src/org/lineageos/mediatek/incallservice/OnLockedBootCompleteReceiver.java b/app/InCallService/src/org/lineageos/mediatek/incallservice/OnLockedBootCompleteReceiver.java
deleted file mode 100644
index 218f4371b1bb..000000000000
--- a/app/InCallService/src/org/lineageos/mediatek/incallservice/OnLockedBootCompleteReceiver.java
+++ /dev/null
@@ -1,19 +0,0 @@
-package org.lineageos.mediatek.incallservice;
-
-import android.content.BroadcastReceiver;
-import android.content.Intent;
-import android.content.Context;
-
-import android.util.Log;
-
-public class OnLockedBootCompleteReceiver extends BroadcastReceiver {
-    private static final String LOG_TAG = "MediatekInCallService";
-
-    @Override
-    public void onReceive(final Context context, Intent intent) {
-        Log.i(LOG_TAG, "onBoot");
-
-        Intent sIntent = new Intent(context, VolumeChangeService.class);
-        context.startService(sIntent);
-    }
-}
diff --git a/app/InCallService/src/org/lineageos/mediatek/incallservice/VolumeChangeReceiver.java b/app/InCallService/src/org/lineageos/mediatek/incallservice/VolumeChangeReceiver.java
deleted file mode 100644
index 98d1d39d493f..000000000000
--- a/app/InCallService/src/org/lineageos/mediatek/incallservice/VolumeChangeReceiver.java
+++ /dev/null
@@ -1,41 +0,0 @@
-package org.lineageos.mediatek.incallservice;
-
-import android.content.Intent;
-import android.content.Context;
-import android.content.BroadcastReceiver;
-
-import android.media.AudioManager;
-import android.media.AudioSystem;
-import android.media.AudioDeviceInfo;
-
-import android.util.Log;
-
-public class VolumeChangeReceiver extends BroadcastReceiver {
-    public static final String LOG_TAG = "MediatekInCallService";
-
-    private AudioManager mAudioManager;
-
-    public VolumeChangeReceiver(AudioManager audioManager) {
-        mAudioManager = audioManager;
-    }
-
-    private void handleVolumeStateChange(Intent intent) {
-        if (intent.getIntExtra(AudioManager.EXTRA_VOLUME_STREAM_TYPE, -1) == AudioManager.STREAM_VOICE_CALL) {
-            AudioDeviceInfo callDevice = mAudioManager.getCommunicationDevice();
-
-            // Try to get volumeIndex
-            int volumeIndex = intent.getIntExtra(AudioManager.EXTRA_VOLUME_STREAM_VALUE, -1);
-            if (volumeIndex < 0) {
-                Log.w(LOG_TAG, "Could not get volumeIndex!");
-                return;
-            }
-
-            GainUtils.setGainLevel(callDevice.getPort().type(), volumeIndex, AudioSystem.STREAM_VOICE_CALL);
-        }
-    }
-
-    @Override
-    public void onReceive(Context context, Intent intent) {
-            handleVolumeStateChange(intent);
-    }
-}
diff --git a/app/InCallService/src/org/lineageos/mediatek/incallservice/VolumeChangeService.java b/app/InCallService/src/org/lineageos/mediatek/incallservice/VolumeChangeService.java
deleted file mode 100644
index 0870315e86d4..000000000000
--- a/app/InCallService/src/org/lineageos/mediatek/incallservice/VolumeChangeService.java
+++ /dev/null
@@ -1,54 +0,0 @@
-package org.lineageos.mediatek.incallservice;
-
-import android.media.AudioManager;
-
-import android.telephony.TelephonyManager;
-import android.telephony.TelephonyCallback;
-
-import android.content.Intent;
-import android.content.IntentFilter;
-import android.content.Context;
-import android.app.Service;
-import android.os.IBinder;
-
-import android.util.Log;
-
-public class VolumeChangeService extends Service {
-    public static final String LOG_TAG = "MediatekInCallService";
-
-    private Context mContext;
-    private VolumeChangeReceiver mVolumeChangeReceiver;
-    private CallStateListener mCallStateListener;
-
-    @Override
-    public IBinder onBind(Intent intent) {
-        return null;
-    }
-
-    @Override
-    public void onDestroy() {
-        super.onDestroy();
-    }
-
-    @Override
-    public int onStartCommand(Intent intent, int flags, int startid) {
-        mContext = this;
-
-        AudioManager audioManager = (AudioManager) mContext.getSystemService(Context.AUDIO_SERVICE);
-        TelephonyManager telephonyManager = (TelephonyManager) mContext.getSystemService(Context.TELEPHONY_SERVICE);
-        mVolumeChangeReceiver = new VolumeChangeReceiver(audioManager);
-        mCallStateListener = new CallStateListener(audioManager);
-
-        Log.i(LOG_TAG, "Service is starting...");
-
-        this.registerReceiver(mVolumeChangeReceiver,
-                               new IntentFilter(AudioManager.VOLUME_CHANGED_ACTION));
-
-        telephonyManager.registerTelephonyCallback(getMainExecutor(), mCallStateListener);
-
-        // Restore gain levels on service start.
-        GainUtils.setGainLevel(audioManager.getStreamVolume(AudioManager.STREAM_VOICE_CALL));
-
-        return START_STICKY;
-    }
-}
```
Error/Change (No): 7
What is the Error: module "mediatek-common" variant "android_common": found in multiple namespaces and module "ImsService" variant "android_common": found in multiple namespaces
The Fix: Removed duplicate mediatek-common and ImsService prebuilts from vendor/realme/RMX2117 since they are now provided by the common hardware/mediatek and vendor/mediatek/ims repositories.
```diff
diff --git a/proprietary-files.txt b/proprietary-files.txt
index 8c8a4802a3eb..cfb4f414996f 100644
--- a/proprietary-files.txt
+++ b/proprietary-files.txt
@@ -29,7 +29,6 @@ lib64/libvt_avsync.so
 lib64/libmtk_vt_wrapper.so
 lib64/libvsim-adaptor-client.so
 lib/libvsim-adaptor-client.so
--framework/mediatek-common.jar
 -framework/mediatek-framework.jar
 framework/mediatek-gwsd.jar
 framework/mediatek-gwsdv2.jar
@@ -41,6 +40,3 @@ framework/mediatek-ims-extension-plugin.jar
 -framework/mediatek-telephony-common.jar
 system_ext/lib/vendor.mediatek.hardware.videotelephony@1.0.so
 system_ext/lib64/vendor.mediatek.hardware.videotelephony@1.0.so
-
-# ImsService - from plato-user 14 UP1A.230620.001 V14.0.7.0.ULQMIXM release-keys
--priv-app/ImsService/ImsService.apk|62e6a0c457812366623630dffef27b0bf5ca7fad
diff --git a/Android.bp b/Android.bp
index 4184bce..c85d48c 100644
--- a/Android.bp
+++ b/Android.bp
@@ -5,22 +5,7 @@
 soong_namespace {
 }
 
-android_app_import {
-	name: "ImsService",
-	owner: "realme",
-	apk: "proprietary/priv-app/ImsService/ImsService.apk",
-	certificate: "platform",
-	dex_preopt: {
-		enabled: false,
-	},
-	privileged: true,
-}
 
-dex_import {
-	name: "mediatek-common",
-	owner: "realme",
-	jars: ["proprietary/framework/mediatek-common.jar"],
-}
 
 dex_import {
 	name: "mediatek-framework",
diff --git a/RMX2117-vendor.mk b/RMX2117-vendor.mk
index da723e1..47b7d4c 100644
--- a/RMX2117-vendor.mk
+++ b/RMX2117-vendor.mk
@@ -36,8 +36,6 @@ PRODUCT_COPY_FILES += \
     vendor/realme/RMX2117/proprietary/system_ext/lib64/vendor.mediatek.hardware.videotelephony@1.0.so:$(TARGET_COPY_OUT_SYSTEM_EXT)/lib64/vendor.mediatek.hardware.videotelephony@1.0.so
 
 PRODUCT_PACKAGES += \
-    ImsService \
-    mediatek-common \
     mediatek-framework \
     mediatek-gwsd \
     mediatek-gwsdv2 \
diff --git a/proprietary/framework/mediatek-common.jar b/proprietary/framework/mediatek-common.jar
deleted file mode 100644
index efd4015..0000000
Binary files a/proprietary/framework/mediatek-common.jar and /dev/null differ
diff --git a/proprietary/priv-app/ImsService/ImsService.apk b/proprietary/priv-app/ImsService/ImsService.apk
deleted file mode 100644
index a95219d..0000000
Binary files a/proprietary/priv-app/ImsService/ImsService.apk and /dev/null differ
```

Error/Change (No): 8
What is the Error: build/soong/fsgen/Android.bp:41:1: module "lineage_RMX2117_generated_system_image" variant "android_common" (created by module "soong_filesystem_creator" variant "android_common"): vintf_fragments: Module android.hardware.biometrics.fingerprint@2.3-service.RMX2117 is referenced by soong-defined filesystem lineage_RMX2117_generated_system_image with property vintf_fragments(device/realme/RMX2117/fingerprint/android.hardware.biometrics.fingerprint@2.3-service.RMX2117.xml) in use. Use vintf_fragment_modules property instead.
The Fix: Replaced `vintf_fragments` with `vintf_fragment_modules` and created `vintf_fragment` modules for `android.hardware.biometrics.fingerprint@2.3-service.RMX2117.xml` and `bluetooth_audio_system.xml` in their respective `Android.bp` files.
```diff
diff --git a/bluetooth/audio/hal/Android.bp b/bluetooth/audio/hal/Android.bp
index 5bc57ed3baf6..28113c3af0df 100644
--- a/bluetooth/audio/hal/Android.bp
+++ b/bluetooth/audio/hal/Android.bp
@@ -1,6 +1,6 @@
 cc_binary {
     name: "android.hardware.bluetooth.audio-service-system",
-    vintf_fragments: ["bluetooth_audio_system.xml"],
+    vintf_fragment_modules: ["bluetooth_audio_system.xml"],
     init_rc: ["android.hardware.bluetooth.audio-service-system.rc"],
     relative_install_path: "hw",
     srcs: [
@@ -38,3 +38,8 @@ cc_binary {
         "android.hardware.audio@7.1-impl-system",
     ],
 }
+
+vintf_fragment {
+    name: "bluetooth_audio_system.xml",
+    src: "bluetooth_audio_system.xml",
+}
diff --git a/fingerprint/Android.bp b/fingerprint/Android.bp
index 5a6833e4b292..cad6ade06b72 100644
--- a/fingerprint/Android.bp
+++ b/fingerprint/Android.bp
@@ -2,7 +2,7 @@ cc_binary {
     name: "android.hardware.biometrics.fingerprint@2.3-service.RMX2117",
     defaults: ["hidl_defaults"],
     init_rc: ["android.hardware.biometrics.fingerprint@2.3-service.RMX2117.rc"],
-    vintf_fragments: ["android.hardware.biometrics.fingerprint@2.3-service.RMX2117.xml"],
+    vintf_fragment_modules: ["android.hardware.biometrics.fingerprint@2.3-service.RMX2117.xml"],
     relative_install_path: "hw",
     srcs: [
         "BiometricsFingerprint.cpp",
@@ -24,3 +24,9 @@ cc_binary {
         "vendor.oplus.hardware.biometrics.fingerprint@2.1",
     ],
 }
+
+vintf_fragment {
+    name: "android.hardware.biometrics.fingerprint@2.3-service.RMX2117.xml",
+    src: "android.hardware.biometrics.fingerprint@2.3-service.RMX2117.xml",
+    vendor: true,
+}
```

Error/Change (No): 9
What is the Error: module "ImsService" variant "android_common": found in multiple namespaces(vendor/mediatek/ims and vendor/realme/RMX2117) when including in system partition
The Fix: Removed duplicate ImsService definition from vendor/realme/RMX2117 Android.bp and RMX2117-vendor.mk, and removed the apk from priv-app.
```diff
--- a/vendor/realme/RMX2117/Android.bp
+++ b/vendor/realme/RMX2117/Android.bp
@@ -7,12 +7,2 @@
 
-android_app_import {
-	name: "ImsService",
-	owner: "realme",
-	apk: "proprietary/priv-app/ImsService/ImsService.apk",
-	certificate: "platform",
-	dex_preopt: {
-		enabled: false,
-	},
-	privileged: true,
-}
 
--- a/vendor/realme/RMX2117/RMX2117-vendor.mk
+++ b/vendor/realme/RMX2117/RMX2117-vendor.mk
@@ -38,3 +38,2 @@
 PRODUCT_PACKAGES += \
-    ImsService \
     mediatek-common \
```

Error/Change (No): 10
What is the Error: module "android.hardware.biometrics.fingerprint@2.3-service.RMX2117" variant "android_arm64_armv8-2a-dotprod": Module "android.hardware.biometrics.fingerprint@2.3-service.RMX2117"("system") and Vintf_fragment "android.hardware.biometrics.fingerprint@2.3-service.RMX2117.xml"("vendor") are installed to different partitions.
The Fix: Removed `vendor: true` from the `vintf_fragment` for the fingerprint service to install it to the system partition, aligning with the GSI build where we do not build the vendor partition.
```diff
diff --git a/fingerprint/Android.bp b/fingerprint/Android.bp
index cad6ade..8a1e2f3 100644
--- a/fingerprint/Android.bp
+++ b/fingerprint/Android.bp
@@ -27,5 +27,4 @@ cc_binary {
 vintf_fragment {
     name: "android.hardware.biometrics.fingerprint@2.3-service.RMX2117.xml",
     src: "android.hardware.biometrics.fingerprint@2.3-service.RMX2117.xml",
-    vendor: true,
 }
```
Error/Change (No): 11
What is the Error: MODULE.linux.STATIC_LIBRARIES.vendor.oplus.hardware.biometrics.fingerprint@2.1 already defined by device/realme/RMX2117/interfaces/biometrics/fingerprint/2.1.
The Fix: Removed duplicate device-specific interfaces directory since the shared hardware/oplus/interfaces repository already defines them.
```diff
diff --git a/interfaces/Android.bp b/interfaces/Android.bp
deleted file mode 100644
index face203b267f..000000000000
--- a/interfaces/Android.bp
+++ /dev/null
@@ -1,4 +0,0 @@
-hidl_package_root {
-    name: "vendor.oplus.hardware",
-    path: "device/realme/RMX2117/interfaces",
-}
diff --git a/interfaces/biometrics/fingerprint/2.1/Android.bp b/interfaces/biometrics/fingerprint/2.1/Android.bp
deleted file mode 100644
index b3b70c12492f..000000000000
--- a/interfaces/biometrics/fingerprint/2.1/Android.bp
+++ /dev/null
@@ -1,15 +0,0 @@
-// This file is autogenerated by hidl-gen -Landroidbp.
-
-hidl_interface {
-    name: "vendor.oplus.hardware.biometrics.fingerprint@2.1",
-    root: "vendor.oplus.hardware",
-    srcs: [
-        "IBiometricsFingerprint.hal",
-        "IBiometricsFingerprintClientCallback.hal",
-        "types.hal",
-    ],
-    interfaces: [
-        "android.hidl.base@1.0",
-    ],
-    gen_java: false,
-}
diff --git a/interfaces/biometrics/fingerprint/2.1/IBiometricsFingerprint.hal b/interfaces/biometrics/fingerprint/2.1/IBiometricsFingerprint.hal
deleted file mode 100644
index 8fe81f19bdc1..000000000000
--- a/interfaces/biometrics/fingerprint/2.1/IBiometricsFingerprint.hal
+++ /dev/null
@@ -1,73 +0,0 @@
-/*
- * Copyright (C) 2017 The Android Open Source Project
- *
- * Licensed under the Apache License, Version 2.0 (the "License");
- * you may not use this file except in compliance with the License.
- * You may obtain a copy of the License at
- *
- *      http://www.apache.org/licenses/LICENSE-2.0
- *
- * Unless required by applicable law or agreed to in writing, software
- * distributed under the License is distributed on an "AS IS" BASIS,
- * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
- * See the License for the specific language governing permissions and
- * limitations under the License.
- */
-
-package vendor.oplus.hardware.biometrics.fingerprint@2.1;
-
-import IBiometricsFingerprintClientCallback;
-
-interface IBiometricsFingerprint {
-
-  setNotify(IBiometricsFingerprintClientCallback clientCallback) generates (uint64_t deviceId);
-
-  preEnroll() generates (uint64_t authChallenge);
-
-  enroll(uint8_t[69] hat, uint32_t gid, uint32_t timeoutSec) generates (RequestStatus debugErrno);
-
-  postEnroll() generates (RequestStatus debugErrno);
-
-  getAuthenticatorId() generates (uint64_t AuthenticatorId);
-
-  cancel() generates (RequestStatus debugErrno);
-
-  enumerate() generates (RequestStatus debugErrno);
-
-  remove(uint32_t gid, uint32_t fid) generates (RequestStatus debugErrno);
-
-  setActiveGroup(uint32_t gid, string storePath) generates (RequestStatus debugErrno);
-
-  authenticate(uint64_t operationId, uint32_t gid) generates (RequestStatus debugErrno);
-
-  pauseEnroll() generates (RequestStatus debugErrno);
-  
-  pauseIdentify() generates (RequestStatus debugErrno);
-  
-  continueEnroll() generates (RequestStatus debugErrno);
-  
-  setScreenState(FingerprintScreenState ScreenState);
-  
-  getAlikeyStatus() generates (RequestStatus debugErrno);
-  
-  continueIdentify() generates (RequestStatus debugErrno);
-  
-  authenticateAsType(uint64_t auth, uint32_t type, FingerprintAuthType AuthType) generates (RequestStatus debugErrno);
-
-  getEngineeringInfo(uint32_t info) generates (RequestStatus debugErrno);
-
-  sendFingerprintCmd(int32_t cmd, vec<int8_t> CmdId) generates (RequestStatus debugErrno);
-
-  dynamicallyConfigLog(uint32_t log) generates (RequestStatus debugErrno);
-  
-  setTouchEventListener() generates (RequestStatus debugErrno);
-  
-  getEnrollmentTotalTimes() generates (RequestStatus debugErrno);
-  
-  cleanUp() generates (RequestStatus debugErrno);
-  
-  touchUp() generates (RequestStatus debugErrno);
-  
-  touchDown() generates (RequestStatus debugErrno);
-  
-};
diff --git a/interfaces/biometrics/fingerprint/2.1/IBiometricsFingerprintClientCallback.hal b/interfaces/biometrics/fingerprint/2.1/IBiometricsFingerprintClientCallback.hal
deleted file mode 100644
index 66c62e8f64d4..000000000000
--- a/interfaces/biometrics/fingerprint/2.1/IBiometricsFingerprintClientCallback.hal
+++ /dev/null
@@ -1,44 +0,0 @@
-/*
- * Copyright (C) 2017 The Android Open Source Project
- *
- * Licensed under the Apache License, Version 2.0 (the "License");
- * you may not use this file except in compliance with the License.
- * You may obtain a copy of the License at
- *
- *      http://www.apache.org/licenses/LICENSE-2.0
- *
- * Unless required by applicable law or agreed to in writing, software
- * distributed under the License is distributed on an "AS IS" BASIS,
- * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
- * See the License for the specific language governing permissions and
- * limitations under the License.
- */
-
-package vendor.oplus.hardware.biometrics.fingerprint@2.1;
-
-/* This HAL interface communicates asynchronous results from the
-   fingerprint driver in response to user actions on the fingerprint sensor
-*/
-interface IBiometricsFingerprintClientCallback {
-
-    oneway onEnrollResult(uint64_t deviceId, uint32_t fingerId, uint32_t groupId, uint32_t remaining);
-
-    oneway onAcquired(uint64_t deviceId, FingerprintAcquiredInfo acquiredInfo, int32_t vendorCode);
-
-    oneway onAuthenticated(uint64_t deviceId, uint32_t fingerId, uint32_t groupId, vec<uint8_t> token);
-
-    oneway onError(uint64_t deviceId, FingerprintError error, int32_t vendorCode);
-
-    oneway onRemoved(uint64_t deviceId, uint32_t fingerId, uint32_t groupId, uint32_t remaining);
-
-    oneway onEnumerate(uint64_t deviceId, uint32_t fingerId, uint32_t groupId, uint32_t remaining);
-
-    oneway onTouchUp(uint64_t deviceId);
-    oneway onTouchDown(uint64_t deviceId);
-    oneway onSyncTemplates(uint64_t deviceId, vec<uint32_t> fingerId, uint32_t remaining);
-    oneway onFingerprintCmd(int32_t deviceId, vec<uint32_t> groupId, uint32_t remaining);
-    oneway onImageInfoAcquired(uint32_t type, uint32_t quality, uint32_t match_score);
-    oneway onMonitorEventTriggered(uint32_t type, string data);
-    oneway onEngineeringInfoUpdated(uint32_t length, vec<uint32_t> keys, vec<string> values);
-
-};
diff --git a/interfaces/biometrics/fingerprint/2.1/types.hal b/interfaces/biometrics/fingerprint/2.1/types.hal
deleted file mode 100644
index 21efeb776305..000000000000
--- a/interfaces/biometrics/fingerprint/2.1/types.hal
+++ /dev/null
@@ -1,103 +0,0 @@
-/*
- * Copyright (C) 2017 The Android Open Source Project
- *
- * Licensed under the Apache License, Version 2.0 (the "License");
- * you may not use this file except in compliance with the License.
- * You may obtain a copy of the License at
- *
- *      http://www.apache.org/licenses/LICENSE-2.0
- *
- * Unless required by applicable law or agreed to in writing, software
- * distributed under the License is distributed on an "AS IS" BASIS,
- * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
- * See the License for the specific language governing permissions and
- * limitations under the License.
- */
-
-package vendor.oplus.hardware.biometrics.fingerprint@2.1;
-
-enum RequestStatus : int32_t {
-  SYS_UNKNOWN = 1,
-  SYS_OK = 0,
-  SYS_ENOENT = -2,
-  SYS_EINTR = -4,
-  SYS_EIO = -5,
-  SYS_EAGAIN = -11,
-  SYS_ENOMEM = -12,
-  SYS_EACCES = -13,
-  SYS_EFAULT = -14,
-  SYS_EBUSY = -16,
-  SYS_EINVAL = -22,
-  SYS_ENOSPC = -28,
-  SYS_ETIMEDOUT = -110,
-};
-
-enum FingerprintError : int32_t {
-  ERROR_NO_ERROR = 0,
-  ERROR_HW_UNAVAILABLE = 1,
-  ERROR_UNABLE_TO_PROCESS = 2,
-  ERROR_TIMEOUT = 3,
-  ERROR_NO_SPACE = 4,
-  ERROR_CANCELED = 5,
-  ERROR_UNABLE_TO_REMOVE = 6,
-  ERROR_LOCKOUT = 7,
-  ERROR_VENDOR = 8
-};
-
-enum FingerprintAcquiredInfo : int32_t {
-  ACQUIRED_GOOD = 0,
-  ACQUIRED_PARTIAL = 1,
-  ACQUIRED_INSUFFICIENT = 2,
-  ACQUIRED_IMAGER_DIRTY = 3,
-  ACQUIRED_TOO_SLOW = 4,
-  ACQUIRED_TOO_FAST = 5,
-  ACQUIRED_VENDOR = 6
-};
-
-struct FingerprintFingerId {
-  uint32_t gid;
-  uint32_t fid;
-};
-
-struct FingerprintEnroll {
-  FingerprintFingerId finger;
-  uint32_t samplesRemaining;  uint64_t msg;
-};
-
-struct FingerprintIterator {
-  FingerprintFingerId finger;
-  uint32_t remainingTemplates;
-};
-
-typedef FingerprintIterator FingerprintEnumerated;
-typedef FingerprintIterator FingerprintRemoved;
-
-struct FingerprintAcquired {
-  FingerprintAcquiredInfo acquiredInfo;
-};
-
-struct FingerprintAuthenticated {
-  FingerprintFingerId finger;
-  uint8_t[69] hat;
-};
-
-enum FingerprintMsgType : int32_t {
-  ERROR = -1,
-  ACQUIRED = 1,
-  TEMPLATE_ENROLLING = 3,
-  TEMPLATE_REMOVED = 4,
-  AUTHENTICATED = 5,
-  TEMPLATE_ENUMERATING = 6,
-};
-
-enum FingerprintScreenState : int32_t {
-  FINGERPRINT_SCREEN_OFF = 0,
-  FINGERPRINT_SCREEN_ON = 1,
-};
-
-enum FingerprintAuthType : int32_t {
-  TYPE_KEYGUARD = 1,
-  TYPE_PAY = 2,
-  TYPE_OTHER = 3,
-};
-
```

Error/Change (No): 12
What is the Error: module "android.hardware.biometrics.fingerprint@2.3-service.RMX2117" depends on undefined module "vendor.oplus.hardware.biometrics.fingerprint@2.1" because it cannot resolve the hardware/oplus namespace.
The Fix: Added an imports list to the soong_namespace in device/realme/RMX2117/Android.bp to explicitly import hardware/oplus, hardware/mediatek, and vendor/mediatek/ims namespaces.
```diff
diff --git a/Android.bp b/Android.bp
index 9515b2573200..bd42cc0e7fd2 100644
--- a/Android.bp
+++ b/Android.bp
@@ -1,2 +1,7 @@
 soong_namespace {
+    imports: [
+        "hardware/oplus",
+        "hardware/mediatek",
+        "vendor/mediatek/ims",
+    ],
 }
```


Error/Change (No): 13
What is the Error: The build failed with `multiple rules generate out/target/product/RMX2117/symbols/system/framework/arm64/boot-mediatek-common.oat [-w dupbuild=err]`. This was because `mediatek-common` was added multiple times to `PRODUCT_BOOT_JARS` and `PRODUCT_PACKAGES`.
The Fix: Removed redundant `mediatek-common` entries from `device/realme/RMX2117/device.mk` and `vendor/realme/RMX2117/RMX2117-vendor.mk` since it is already included via `hardware/mediatek/frameworks/mediatek-frameworks.mk`.
```diff
diff --git a/device.mk b/device.mk
index 73d6e612f167..d6ec118a7714 100644
--- a/device.mk
+++ b/device.mk
@@ -126,7 +126,6 @@ PRODUCT_PACKAGES += \
 # IMS
 $(call inherit-product, vendor/mediatek/ims/ims.mk)
 PRODUCT_BOOT_JARS += \
-    mediatek-common \
     mediatek-framework \
     mediatek-ims-base \
     mediatek-ims-common \
diff --git a/RMX2117-vendor.mk b/RMX2117-vendor.mk
index 4a9a24a1f355..47b7d4c3c0a0 100644
--- a/RMX2117-vendor.mk
+++ b/RMX2117-vendor.mk
@@ -36,7 +36,6 @@ PRODUCT_COPY_FILES += \
     vendor/realme/RMX2117/proprietary/system_ext/lib64/vendor.mediatek.hardware.videotelephony@1.0.so:$(TARGET_COPY_OUT_SYSTEM_EXT)/lib64/vendor.mediatek.hardware.videotelephony@1.0.so
 
 PRODUCT_PACKAGES += \
-    mediatek-common \
     mediatek-framework \
     mediatek-gwsd \
     mediatek-gwsdv2 \
```


Error/Change (No): 14
What is the Error: `ccache` failed to run during compilation with the error `ccache: error: Not a directory`.
The Fix: Removed the invalid `~/.cache/ccache` regular file and created the correct directory structure so `ccache` can properly store and access cache files.
```diff
# No code diff, as this was a filesystem fix: rm ~/.cache/ccache && mkdir -p ~/.cache/ccache
```

Error/Change (No): 15
What is the Error: The kernel compilation failed with an `exec format error` because the clang-19 prebuilt binary (`clang.real`) was not a valid ELF executable. It was a Git LFS pointer text file because `git lfs` wasn't initialized before syncing the prebuilts repo.
The Fix: Initialized Git LFS (`git lfs install`) and checked out the binary files (`git lfs checkout`) in `prebuilts/clang/host/linux-x86/clang-r530567`.
```diff
# No code diff, as this was a Git LFS repository initialization and checkout command to fetch the toolchain binaries.
```

Error/Change (No): 16
What is the Error: device/realme/RMX2117/sepolicy/private/kpoc_charger.te:1:ERROR 'Duplicate declaration of type' at token ';' on line 90116: type kpoc_charger, domain;
The Fix: Removed duplicate kpoc_charger and kpoc_charger_exec type declarations from the device tree since they are provided by the MediaTek vendor sepolicy.
```diff
@@ -1,5 +1,4 @@
-type kpoc_charger, domain;
-type kpoc_charger_exec, system_file_type, exec_type, file_type;
+
 # Move to system partition
 typeattribute kpoc_charger coredomain;
 init_daemon_domain(kpoc_charger)
```

