#
# Copyright (C) 2010-2012 Jo-Philipp Wich <xm@subsignal.org>
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=uhttpd
PKG_RELEASE:=34

PKG_BUILD_DIR := $(BUILD_DIR)/$(PKG_NAME)
PKG_CONFIG_DEPENDS := \
	CONFIG_PACKAGE_uhttpd_debug \
	CONFIG_PACKAGE_uhttpd-mod-lua \
	CONFIG_PACKAGE_uhttpd-mod-tls \
	CONFIG_PACKAGE_uhttpd-mod-tls_cyassl \
	CONFIG_PACKAGE_uhttpd-mod-tls_openssl \
	CONFIG_PACKAGE_uhttpd-mod-ubus

include $(INCLUDE_DIR)/package.mk

define Package/uhttpd/default
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Web Servers/Proxies
  TITLE:=uHTTPd - tiny, single threaded HTTP server
  MAINTAINER:=Jo-Philipp Wich <xm@subsignal.org>
endef

define Package/uhttpd
  $(Package/uhttpd/default)
  DEPENDS:=+libubox
endef

define Package/uhttpd/description
 uHTTPd is a tiny single threaded HTTP server with TLS, CGI and Lua
 support. It is intended as a drop-in replacement for the Busybox
 HTTP daemon.
endef

define Package/uhttpd/config
  config PACKAGE_uhttpd_debug
    bool "Build with debug messages"
    default n
endef


define Package/uhttpd-mod-tls
  $(Package/uhttpd/default)
  TITLE+= (TLS plugin)
  DEPENDS:=uhttpd +PACKAGE_uhttpd-mod-tls_cyassl:libcyassl +PACKAGE_uhttpd-mod-tls_openssl:libopenssl
endef

define Package/uhttpd-mod-tls/description
 The TLS plugin adds HTTPS support to uHTTPd.
endef

define Package/uhttpd-mod-tls/config
  choice
    depends on PACKAGE_uhttpd-mod-tls
    prompt "TLS Provider"
    default PACKAGE_uhttpd-mod-tls_cyassl

    config PACKAGE_uhttpd-mod-tls_cyassl
      bool "CyaSSL"

    config PACKAGE_uhttpd-mod-tls_openssl
      bool "OpenSSL"
  endchoice
endef

UHTTPD_TLS:=
TLS_CFLAGS:=
TLS_LDFLAGS:=

ifneq ($(CONFIG_PACKAGE_uhttpd-mod-tls_cyassl),)
  UHTTPD_TLS:=cyassl
  TLS_CFLAGS:=-I$(STAGING_DIR)/usr/include/cyassl -DTLS_IS_CYASSL
  TLS_LDFLAGS:=-lcyassl -lm
endif

ifneq ($(CONFIG_PACKAGE_uhttpd-mod-tls_openssl),)
  UHTTPD_TLS:=openssl
  TLS_CFLAGS:=-DTLS_IS_OPENSSL
  TLS_LDFLAGS:=-lssl
endif


define Package/uhttpd-mod-lua
  $(Package/uhttpd/default)
  TITLE+= (Lua plugin)
  DEPENDS:=uhttpd +liblua
endef

define Package/uhttpd-mod-lua/description
 The Lua plugin adds a CGI-like Lua runtime interface to uHTTPd.
endef


define Package/uhttpd-mod-ubus
  $(Package/uhttpd/default)
  TITLE+= (ubus plugin)
  DEPENDS:=uhttpd +libubus +libblobmsg-json
endef

define Package/uhttpd-mod-ubus/description
 The ubus plugin adds a HTTP/JSON RPC proxy for ubus and publishes the
 session.* namespace and procedures.
endef


TARGET_CFLAGS += $(TLS_CFLAGS) $(if $(CONFIG_PACKAGE_uhttpd_debug),-DDEBUG) -ggdb3
TARGET_LDFLAGS += -lubox -Wl,-rpath-link=$(STAGING_DIR)/usr/lib
MAKE_VARS += \
	FPIC="$(FPIC)" \
	LUA_SUPPORT="$(if $(CONFIG_PACKAGE_uhttpd-mod-lua),1)" \
	TLS_SUPPORT="$(if $(CONFIG_PACKAGE_uhttpd-mod-tls),1)" \
	UBUS_SUPPORT="$(if $(CONFIG_PACKAGE_uhttpd-mod-ubus),1)" \
	UHTTPD_TLS="$(UHTTPD_TLS)" \
	TLS_CFLAGS="$(TLS_CFLAGS)" \
	TLS_LDFLAGS="$(TLS_LDFLAGS)"

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
endef

define Package/uhttpd/conffiles
/etc/config/uhttpd
/etc/uhttpd.crt
/etc/uhttpd.key
endef

define Package/uhttpd/install
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/uhttpd.init $(1)/etc/init.d/uhttpd
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) ./files/uhttpd.config $(1)/etc/config/uhttpd
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/uhttpd $(1)/usr/sbin/uhttpd
endef

define Package/uhttpd-mod-tls/install
	$(INSTALL_DIR) $(1)/usr/lib
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/uhttpd_tls.so $(1)/usr/lib/
endef

define Package/uhttpd-mod-lua/install
	$(INSTALL_DIR) $(1)/usr/lib
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/uhttpd_lua.so $(1)/usr/lib/
endef

define Package/uhttpd-mod-ubus/install
	$(INSTALL_DIR) $(1)/usr/lib
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/uhttpd_ubus.so $(1)/usr/lib/
endef


$(eval $(call BuildPackage,uhttpd))
$(eval $(call BuildPackage,uhttpd-mod-tls))
$(eval $(call BuildPackage,uhttpd-mod-lua))
$(eval $(call BuildPackage,uhttpd-mod-ubus))
