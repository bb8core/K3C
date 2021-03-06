#
# Copyright (C) 2006-2010 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=iproute2
PKG_VERSION:=3.12.0
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=http://devresources.linux-foundation.org/dev/iproute2/download/
PKG_SOURCE_URL:=https://www.kernel.org/pub/linux/utils/net/iproute2/
#PKG_MD5SUM:=8a3b6bc77c2ecf752284aa4a6fc630a6
PKG_BUILD_PARALLEL:=1

PKG_BUILD_DIR:=$(BUILD_DIR)/iproute2-$(PKG_VERSION)

include $(INCLUDE_DIR)/package.mk

define Package/tc-latest/Default
  SECTION:=net
  CATEGORY:=Network
  URL:=http://linux-net.osdl.org/index.php/tc-latest
endef

define Package/ip-latest
$(call Package/tc-latest/Default)
  SUBMENU:=Routing and Redirection
  TITLE:=Routing control utility
endef

define Package/ip-latest/conffiles
/ramdisk/etc/tc-latest/rt_tables
endef

define Package/tc-latest
$(call Package/tc-latest/Default)
  TITLE:=Traffic control utility with codel support
  DEPENDS:=+kmod-sched
endef

define Package/genl-latest
$(call Package/tc-latest/Default)
  TITLE:=General netlink utility frontend
endef

define Package/ss-latest
$(call Package/tc-latest/Default)
  TITLE:=Socket statistics utility
endef

define Build/Configure
	$(SED) "s,-I/usr/include/db3,," $(PKG_BUILD_DIR)/Makefile
	$(SED) "s,^KERNEL_INCLUDE.*,KERNEL_INCLUDE=$(LINUX_DIR)/include," \
		$(PKG_BUILD_DIR)/Makefile
	$(SED) "s,^LIBC_INCLUDE.*,LIBC_INCLUDE=$(STAGING_DIR)/include," \
		$(PKG_BUILD_DIR)/Makefile
	# For now disable compiling of the misc directory because it seems to fail
	rm -rf $(PKG_BUILD_DIR)/misc
	$(SED) "s, misc,," $(PKG_BUILD_DIR)/Makefile
	echo "static const char SNAPSHOT[] = \"$(PKG_VERSION)-$(PKG_RELEASE)-openwrt\";" \
		> $(PKG_BUILD_DIR)/include/SNAPSHOT.h
endef

MAKE_FLAGS += \
	EXTRA_CCOPTS="$(TARGET_CFLAGS)" \
	KERNEL_INCLUDE="$(LINUX_DIR)/include" \
	FPIC="$(FPIC)"

define Build/Compile
	$(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR)/netem \
		HOSTCC="$(HOSTCC)"  EXTRA_CCOPTS="$(TARGET_CFLAGS)" \
		CFLAGS="-D_GNU_SOURCE -O2 -Wstrict-prototypes -Wall \
		-I../include -DRESOLVE_HOSTNAMES"
	$(MAKE_VARS) $(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR) $(MAKE_FLAGS) all
	$(MAKE_VARS) $(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR)/tc $(MAKE_FLAGS) tc
	$(MAKE_VARS) $(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR)/ip $(MAKE_FLAGS) ip
endef



define Package/tc-latest/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/tc/tc $(1)/usr/sbin/tc-latest
endef


$(eval $(call BuildPackage,ip-latest))
$(eval $(call BuildPackage,tc-latest))
$(eval $(call BuildPackage,genl-latest))
$(eval $(call BuildPackage,ss-latest))
