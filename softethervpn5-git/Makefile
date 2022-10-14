# Based partially on the versions of el1n and Federico Di Marco

include $(TOPDIR)/rules.mk

PKG_NAME:=softethervpn5-git
PKG_VERSION:=2022-09-27
PKG_RELEASE:=$(AUTORELEASE)

PKG_LICENSE:=GPL-2.0
PKG_LICENSE_FILES:=COPYING

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/SoftEtherVPN/SoftEtherVPN.git
PKG_SOURCE_DATE:=2022-09-27
PKG_SOURCE_VERSION:=8ae0a932da1858abe26e07a4a03c18b04527ff64
PKG_MIRROR_HASH:=bf605d011279089ca4ccc2686f90bc2a5f9eb56569138b52e3c5e4befe09cb9c

HOST_BUILD_DIR:=$(BUILD_DIR_HOST)/SoftEtherVPN-$(PKG_VERSION)
PKG_BUILD_DIR:=$(BUILD_DIR)/SoftEtherVPN-$(PKG_VERSION)

HOST_BUILD_DEPENDS:=ncurses/host readline/host
PKG_BUILD_DEPENDS:=softethervpn5-git/host

include $(INCLUDE_DIR)/package.mk
include $(INCLUDE_DIR)/host-build.mk
include $(INCLUDE_DIR)/nls.mk
include $(INCLUDE_DIR)/cmake.mk

define Package/softethervpn5-git/Default
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=VPN
  TITLE:=softethervpn5-git v$(PKG_VERSION)
  URL:=http://www.softether.org/
endef

define Package/softethervpn5-git/Default/description
  SoftEther VPN supports SSL-VPN, OpenVPN, L2TP, EtherIP, L2TPv3 and IPsec as a single VPN software.
  SoftEther VPN is not only an alternative VPN server to existing VPN products (OpenVPN, IPsec and MS-SSTP),
  but has also original strong SSL-VPN protocol to penetrate any kinds of firewalls.
  Guide: https://wordpress.tirlins.com/2015/03/setting-up-softether-vpn-on-openwrt/
endef

define Package/softethervpn5-git-libs
  $(call Package/softethervpn5-git/Default)
  DEPENDS:=+libsodium +libpthread +librt +libreadline +libopenssl +libncurses +kmod-tun +zlib $(ICONV_DEPENDS)
  TITLE+= libs
  HIDDEN:=1
endef

define Package/softethervpn5-git-server
  $(call Package/softethervpn5-git/Default)
  TITLE+= server
  DEPENDS:= +softethervpn5-git-libs
endef
define Package/softethervpn5-git-server/description
  $(call Package/softethervpn5-git/Default/description)

  Provides the vpnserver (daemon).
endef

define Package/softethervpn5-git-bridge
  $(call Package/softethervpn5-git/Default)
  TITLE+= bridge
  DEPENDS:= +softethervpn5-git-libs  
endef
define Package/softethervpn5-git-bridge/description
  $(call Package/softethervpn5-git/Default/description)

  Provides the vpnbridge (daemon).
endef

define Package/softethervpn5-git-client
  $(call Package/softethervpn5-git/Default)
  TITLE+= client
  DEPENDS:= +softethervpn5-git-libs  
endef
define Package/softethervpn5-git-client/description
  $(call Package/softethervpn5-git/Default/description)

  Provides the vpnclient.
endef

export USE_MUSL=YES
# BUG: outdated host/include/elf.h
HOST_CFLAGS += $(FPIC) -DAT_HWCAP2=26
TARGET_CFLAGS += $(FPIC)
CMAKE_OPTIONS += -DICONV_LIB_PATH="$(ICONV_PREFIX)/lib"

# static build for host (hamcorebuilder), avoid -fpic on ncurses/host and shared libs can't be found on host
define Host/Prepare
	$(Host/Prepare/Default)
	$(SED) 's,SHARED,STATIC,g' $(HOST_BUILD_DIR)/src/Mayaqua/CMakeLists.txt
	$(SED) 's,SHARED,STATIC,g' $(HOST_BUILD_DIR)/src/Cedar/CMakeLists.txt
	$(SED) 's,readline,libreadline.a,g' $(HOST_BUILD_DIR)/src/Cedar/CMakeLists.txt
	$(SED) 's,libsodium REQUIRED,libsodium,g' $(HOST_BUILD_DIR)/src/Cedar/CMakeLists.txt
endef

define Host/Compile
	$(call Host/Compile/Default,hamcorebuilder)
endef

define Host/Install
	$(INSTALL_DIR) $(STAGING_DIR_HOSTPKG)/bin/
	$(INSTALL_BIN) $(HOST_BUILD_DIR)/src/hamcorebuilder/hamcorebuilder $(STAGING_DIR_HOSTPKG)/bin/
endef

# disable SSE detection, disables BLAKE2 lib
define Build/Prepare
	$(call Build/Prepare/Default)
	$(SED) 's,cmake_host_system_information,#cmake_host_system_information,g' $(PKG_BUILD_DIR)/src/Cedar/CMakeLists.txt
endef

define Build/Compile
	$(call Build/Compile/Default,vpnserver vpnbridge vpnclient vpncmd hamcore-archive-build)
endef

define Package/softethervpn5-git-libs/install
	$(INSTALL_DIR) $(1)/usr/lib
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libcedar.so $(1)/usr/lib/
	$(CP) $(PKG_INSTALL_DIR)/usr/lib/libmayaqua.so $(1)/usr/lib/
	$(INSTALL_DIR) $(1)/usr/libexec/softethervpn
	$(CP) $(PKG_INSTALL_DIR)/usr/libexec/softether/vpncmd/hamcore.se2 $(1)/usr/libexec/softethervpn/
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/libexec/softether/vpncmd/vpncmd $(1)/usr/libexec/softethervpn/
	$(INSTALL_DIR) $(1)/usr/bin
	if [ -f $(PKG_INSTALL_DIR)/usr/bin/list_cpu_features ]; then \
		$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/list_cpu_features $(1)/usr/bin/ ; fi 
	$(INSTALL_BIN) files/launcher.sh $(1)/usr/libexec/softethervpn/
	$(INSTALL_DATA) files/dummy $(1)/usr/libexec/softethervpn/lang.config
	$(INSTALL_DIR) $(1)/usr/bin
	$(LN) ../../usr/libexec/softethervpn/launcher.sh $(1)/usr/bin/vpncmd
endef

define Package/softethervpn5-git-server/install
	$(INSTALL_DIR) $(1)/usr/libexec/softethervpn
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/libexec/softether/vpnserver/vpnserver $(1)/usr/libexec/softethervpn/
	$(INSTALL_DATA) files/dummy $(1)/usr/libexec/softethervpn/vpn_server.config
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) files/vpnserver.init $(1)/etc/init.d/softethervpnserver
endef

define Package/softethervpn5-git-bridge/install
	$(INSTALL_DIR) $(1)/usr/libexec/softethervpn
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/libexec/softether/vpnbridge/vpnbridge $(1)/usr/libexec/softethervpn/
	$(INSTALL_DATA) files/dummy $(1)/usr/libexec/softethervpn/vpn_bridge.config
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) files/vpnbridge.init $(1)/etc/init.d/softethervpnbridge
endef

define Package/softethervpn5-git-client/install
	$(INSTALL_DIR) $(1)/usr/libexec/softethervpn
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/libexec/softether/vpnclient/vpnclient $(1)/usr/libexec/softethervpn/
	$(INSTALL_DATA) files/dummy $(1)/usr/libexec/softethervpn/vpn_client.config
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) files/vpnclient.init $(1)/etc/init.d/softethervpnclient
endef

define Package/softethervpn5-git-server/conffiles
/usr/libexec/softethervpn/vpn_server.config
/usr/libexec/softethervpn/lang.config
endef

define Package/softethervpn5-git-bridge/conffiles
/usr/libexec/softethervpn/vpn_bridge.config
/usr/libexec/softethervpn/lang.config
endef

define Package/softethervpn5-git-client/conffiles
/usr/libexec/softethervpn/vpn_client.config
/usr/libexec/softethervpn/lang.config
endef

$(eval $(call HostBuild))
$(eval $(call BuildPackage,softethervpn5-git-libs))
$(eval $(call BuildPackage,softethervpn5-git-server))
$(eval $(call BuildPackage,softethervpn5-git-bridge))
$(eval $(call BuildPackage,softethervpn5-git-client))
