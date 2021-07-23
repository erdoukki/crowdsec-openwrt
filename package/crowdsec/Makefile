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

PKG_BUILD_DEPENDS:=golang/host
PKG_BUILD_PARALLEL:=1
PKG_USE_MIPS16:=0

GO_PKG:=github.com/crowdsecurity/crowdsec

include $(INCLUDE_DIR)/package.mk
#include ../../lang/golang/golang-package.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

define Package/crowdsec/Default
  TITLE:=CrowdSec is a free, modern & collaborative behavior detection engine
  URL:=https://github.com/crowdsecurity/crowdsec/
  DEPENDS:=$(GO_ARCH_DEPENDS)
  HOST_DEPENDS:=jq
  PKG_CONFIG_DEPENDS:= \
  	@CONFIG_EXTRA_BINUTILS_CONFIG_OPTIONS="--enable-gold --enable-plugins"
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
  CrowdSec is a free, modern & collaborative behavior detection engine, 
  coupled with a global IP reputation network. It stacks on fail2ban's 
  philosophy but is IPV6 compatible and 60x faster (Go vs Python), uses 
  Grok patterns to parse logs and YAML scenario to identify behaviors. 
  CrowdSec is engineered for modern Cloud / Containers / VM based 
  infrastructures (by decoupling detection and remediation). 
  Once detected you can remedy threats with various bouncers 
  (firewall block, nginx http 403, Captchas, etc.) while the 
  aggressive IP can be sent to CrowdSec for curation before 
  being shared among all users to further improve everyone's security.
endef

define Package/crowdsec/description
$(call Package/crowdsec/Default/description)

This package contains the main program.
endef

define Package/golang-crowdsec-dev/description
$(call Package/crowdsec/Default/description)

This package provides the source files for the program.
endef

GO_PKG_INSTALL_ALL:=1
MAKE_VARS += $(GO_PKG_VARS) BUILD_VERSION=v$(PKG_VERSION)
MAKE_FLAGS += release --debug

define Build/Compile
	+$(MAKE_VARS) \
	$(GO_PKG_VARS) \
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
		$(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/config/config.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/config/dev.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/config/user.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/config/acquis.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/config/profiles.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/config/simulation.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/config/local_api_credentials.yaml \
		$(1)/etc/crowdsec
	$(INSTALL_DATA) \
		$(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/config/online_api_credentials.yaml \
		$(1)/etc/crowdsec

	$(CP) \
		$(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/config/patterns/* \
		$(1)/etc/crowdsec/patterns

	$(INSTALL_DIR) $(1)/srv/crowdsec/data/
	
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) \
		./files/crowdsec.initd \
		$(1)/etc/init.d/crowdsec
endef

define Package/crowdsec/postinst
#!/bin/sh
if [ -z "$${IPKG_INSTROOT}" ]; then
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
  rm -f /usr/bin/crowdsec
  rm -f /usr/bin/cscli
fi
endef

$(eval $(call GoBinPackage,crowdsec))
$(eval $(call BuildPackage,crowdsec))

$(eval $(call GoSrcPackage,golang-crowdsec-dev))
$(eval $(call BuildPackage,golang-crowdsec-dev))

