#
# Copyright (C) 2021 Gerald Kerma
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=crowdsec
PKG_VERSION:=1.1.1
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/crowdsecurity/crowdsec
PKG_SOURCE_VERSION:=73e0bbaf93070f4a640eb5a22212b5dcf26699de
PKG_SOURCE_DATE:=20210707
PKG_MIRROR_HASH:=940e6e122097468ad99fcee2db569afda94fc2a65837c74b0ee71bbb5c64518c

PKG_LICENSE:=MIT
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=Gerald Kerma <gandalf@gk2.net>

PKG_BUILD_DEPENDS:=golang/host jq/host
PKG_BUILD_PARALLEL:=1
PKG_USE_MIPS16:=0

GO_PKG:=github.com/crowdsecurity/crowdsec
GO_PKG_BUILD_PKG:=github.com/crowdsecurity/crowdsec

include $(INCLUDE_DIR)/package.mk
#include ../../lang/golang/golang-package.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

define Package/crowdsec/Default
  TITLE:=CrowdSec is a free, modern & collaborative behavior detection engine
  URL:=https://github.com/crowdsecurity/crowdsec/
  DEPENDS:=$(GO_ARCH_DEPENDS) \
  	+jq
endef

define Package/crowdsec
$(call Package/crowdsec/Default)
  SECTION:=net
  CATEGORY:=Network
endef

define Package/golang-crowdsec-dev
$(call Package/crowdsec/Default)
$(call GoPackage/GoSubMenu)
  TITLE+= (source files)
  PKGARCH:=all
endef

define Package/crowdsec/Default/description
  Crowdsec - An open-source, lightweight agent to detect 
  and respond to bad behaviours. 
  It also automatically benefits from our global 
  community-wide IP reputation database 
endef

define Package/crowdsec/description
$(call Package/crowdsec/Default/description)

  This package contains the main program.
endef

define Package/golang-crowdsec-dev/description
$(call Package/crowdsec/Default/description)

  This package provides the source files for the program.
endef

#--debug basic, verbose, implicit, jobs, all, or makefile.
PKG_JOBS += static --debug=basic
GOARCH=$(GO_ARCH)
GO_PKG_BUILD_VARS += GO111MODULE=auto
GO_PKG_INSTALL_ALL:=1
MAKE_VARS += $(GO_PKG_VARS)
MAKE_FLAGS += BUILD_VERSION=v$(PKG_VERSION)

define Build/Compile
	+$(MAKE_VARS) \
	$(MAKE) $(PKG_JOBS) -C $(GO_PKG_BUILD_DIR)/src/$(GO_PKG) \
		$(MAKE_FLAGS)
endef

define Package/crowdsec/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/cmd/crowdsec/crowdsec $(1)/usr/bin/
	$(INSTALL_BIN) $(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/cmd/crowdsec-cli/cscli $(1)/usr/bin/

	$(INSTALL_DIR) $(1)/etc/crowdsec
	$(INSTALL_DIR) $(1)/etc/crowdsec/scenarios
	$(INSTALL_DIR) $(1)/etc/crowdsec/postoverflows
	$(INSTALL_DIR) $(1)/etc/crowdsec/collections
	$(INSTALL_DIR) $(1)/etc/crowdsec/patterns
	$(INSTALL_DIR) $(1)/etc/crowdsec/hub
	
	$(INSTALL_DATA) \
		$(PKG_BUILD_DIR)/config/config.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(PKG_BUILD_DIR)/config/dev.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(PKG_BUILD_DIR)/config/user.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(PKG_BUILD_DIR)/config/acquis.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(PKG_BUILD_DIR)/config/profiles.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(PKG_BUILD_DIR)/config/simulation.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(PKG_BUILD_DIR)/config/local_api_credentials.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(PKG_BUILD_DIR)/config/online_api_credentials.yaml \
		$(1)/etc/crowdsec

	$(CP) \
		$(PKG_BUILD_DIR)/config/patterns/* \
		$(1)/etc/crowdsec/patterns

	$(INSTALL_DIR) $(1)/srv/crowdsec/data/
	
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) \
		./files/crowdsec.initd \
		$(1)/etc/init.d/crowdsec
		
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) \
		./files/crowdsec.config \
		$(1)/etc/config/crowdsec
endef

define Package/crowdsec/postinst
#!/bin/sh
if [ -z "$${IPKG_INSTROOT}" ]; then
  /etc/init.d/crowdsec enable
  /etc/init.d/crowdsec restart
  cscli -c /etc/crowdsec/config.yaml machines add --auto
  cscli -c /etc/crowdsec/config.yaml capi register
  cscli hub update
  cscli parsers install "crowdsecurity/whitelists"
  /etc/init.d/crowdsec restart
fi
endef

define Package/crowdsec/prerm
#!/bin/sh
if [ -z "$${IPKG_INSTROOT}" ]; then
  /etc/init.d/crowdsec stop
  /etc/init.d/crowdsec disable
  rm -f /usr/bin/crowdsec
  rm -f /usr/bin/cscli
  rm -f /etc/init.d/crowdsec
fi
endef

define Package/crowdsec/conffiles
  /etc/crowdsec/config/config.yaml
  /etc/crowdsec/config/local_api_credentials.yaml
  /etc/config/crowdsec
endef

$(eval $(call GoBinPackage,crowdsec))
$(eval $(call BuildPackage,crowdsec))

$(eval $(call GoSrcPackage,golang-crowdsec-dev))
$(eval $(call BuildPackage,golang-crowdsec-dev))

