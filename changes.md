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
