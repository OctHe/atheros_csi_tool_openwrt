## Introduction

[Atheros CSI tool](https://wands.sg/research/wifi/AtherosCSI/) is used for collect CSI from atheros wireless card.
However, the Atheros csi tool only provide OpenWrt firmware for old routers and the new firmware does not be updated at all.

We wants to solve this problem by building the csi tool from OpenWrt source code for any new OpenWrt system.
We use OpenWrt v19.07.7 as an example.
The building process has been verified by [TP-Link Archer C7 AC 1750](https://openwrt.org/toh/tp-link/archer_c7) and [TP-Link TL-WDR 4310](https://openwrt.org/toh/tp-link/tl-wdr4310_v1) routers.

## Preparation

First, you must confirm that the OpenWrt system can be successfully compiled from the source code.
The build process of an original OpenWrt system can be found on the [official website](https://openwrt.org/docs/guide-developer/quickstart-build-images).
If you are new for OpenWrt and want to know the details of the source code, the overview of the source code can be found [here](https://openwrt.org/docs/guide-developer/overview).

## Patch the Source

The source code of the csi tool that can be installed to OpenWrt can be download from [GitHub](https://github.com/OctHe/atheros_csi_tool_openwrt).
Note that these files, which are organized from the [GitHub of Atheros CSI tool](https://github.com/wands-wireless/Atheros_CSI_tool_OpenWRT_src), only contain the source code of the Atheros CSI tool.
They do not contain the OpenWrt source code. 
So, you shoud copy these files into the OpenWrt source code.

Besides, two Makefiles should be revised.
The directory of these two files is 'openwrt/package/kernel/mac80211/" (Assume that openwrt is the directory of the OpenWrt source code).
The first file that need to be revised in the directory is 'openwrt/package/kernel/mac80211/Makefile'.
You should add the following two codes:

    $(CP) ./csi/ar9003_csi.c  $(PKG_BUILD_DIR)/drivers/net/wireless/ath/ath9k/
	$(CP) ./csi/ar9003_csi.h  $(PKG_BUILD_DIR)/drivers/net/wireless/ath/ath9k/

 After

    echo 'compat-wireless-$(PKG_VERSION)-$(PKG_RELEASE)-$(REVISION)' > $(PKG_BUILD_DIR)/compat_version

The new content in the revised Makefile is 

    echo 'compat-wireless-$(PKG_VERSION)-$(PKG_RELEASE)-$(REVISION)' > $(PKG_BUILD_DIR)/compat_version
    $(CP) ./csi/ar9003_csi.c  $(PKG_BUILD_DIR)/drivers/net/wireless/ath/ath9k/
	$(CP) ./csi/ar9003_csi.h  $(PKG_BUILD_DIR)/drivers/net/wireless/ath/ath9k/

The second file is "openwrt/package/kernel/mac80211/ath.mk".
The following part of the original file

    define KernelPackage/ath9k-common
      $(call KernelPackage/mac80211/Default)
      TITLE:=Atheros 802.11n wireless devices (common code for ath9k and ath9k_htc)
      URL:=https://wireless.wiki.kernel.org/en/users/drivers/ath9k
      HIDDEN:=1
      DEPENDS+= @PCI_SUPPORT||USB_SUPPORT||TARGET_ar71xx||TARGET_ath79 +kmod-ath +@DRIVER_11N_SUPPORT +@DRIVER_11W_SUPPORT
      FILES:= \
        $(PKG_BUILD_DIR)/drivers/net/wireless/ath/ath9k/ath9k_common.ko \
        $(PKG_BUILD_DIR)/drivers/net/wireless/ath/ath9k/ath9k_hw.ko
    endef

The revised content of ath.mk is

    define KernelPackage/ath9k-common
      $(call KernelPackage/mac80211/Default)
      TITLE:=Atheros 802.11n wireless devices (common code for ath9k and ath9k_htc)
      URL:=https://wireless.wiki.kernel.org/en/users/drivers/ath9k
      HIDDEN:=1
      DEPENDS+= @PCI_SUPPORT||USB_SUPPORT||TARGET_ar71xx||TARGET_ath79 +kmod-ath +@DRIVER_11N_SUPPORT +@DRIVER_11W_SUPPORT
      FILES:= \
        $(PKG_BUILD_DIR)/drivers/net/wireless/ath/ath9k/ath9k_common.ko \
        $(PKG_BUILD_DIR)/drivers/net/wireless/ath/ath9k/ath9k_hw.ko \
        $(PKG_BUILD_DIR)/drivers/net/wireless/ath/ath9k/ar9003_csi.ko
    endef


## Build from Source and System Upgrade

The build process can follows the [Atheros CSI tool Wiki](https://github.com/xieyaxiongfly/Atheros_CSI_tool_OpenWRT_src/wiki/Install-OpenWRT-version-of-Atheros-CSI-tool).

In our case, the router have been installed an OpenWrt before, so we use *sysupgrade* command in the OpenWrt to upgrade the system.
If you can find *sendData* and *recvCSI* commands in the terminal, the new system has been successfully installed.
