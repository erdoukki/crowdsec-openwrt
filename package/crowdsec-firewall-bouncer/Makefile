#
# Copyright (C) 2021 Gerald Kerma
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=crowdsec-firewall-bouncer
PKG_VERSION:=0.0.13
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/crowdsecurity/cs-firewall-bouncer
PKG_SOURCE_VERSION:=f302e9820077fee858ffe29b3adbd980d9c34107
PKG_SOURCE_DATE:=20210705
PKG_MIRROR_HASH:=7490a99288246bbc883384c47b441bb2f3867724a9f1a0e94d352fe36d47c220

PKG_LICENSE:=MIT
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=Gerald Kerma <gandalf@gk2.net>

PKG_BUILD_DEPENDS:=golang/host jq/host
PKG_BUILD_PARALLEL:=1
PKG_USE_MIPS16:=0

GO_PKG:=github.com/crowdsecurity/cs-firewall-bouncer

include $(INCLUDE_DIR)/package.mk
#include ../../lang/golang/golang-package.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

define Package/crowdsec-firewall-bouncer/Default
  TITLE:=Crowdsec bouncer written in golang for firewalls.
  URL:=https://github.com/crowdsecurity/cs-firewall-bouncer/
  DEPENDS:=$(GO_ARCH_DEPENDS) \
  	+crowdsec \
  	+jq
endef

define Package/crowdsec-firewall-bouncer
$(call Package/crowdsec-firewall-bouncer/Default)
  SECTION:=net
  CATEGORY:=Network
endef

define Package/golang-crowdsec-firewall-bouncer-dev
$(call Package/crowdsec-firewall-bouncer/Default)
$(call GoPackage/GoSubMenu)
  TITLE+= (source files)
  PKGARCH:=all
endef

define Package/crowdsec-firewall-bouncer/Default/description
  Crowdsec bouncer written in golang for firewalls.

  crowdsec-firewall-bouncer will fetch new and old decisions
  from a CrowdSec API to add them in a blocklist used by supported firewalls.
endef

define Package/crowdsec-firewall-bouncer/description
$(call Package/crowdsec-firewall-bouncer/Default/description)

  This package contains the main program.
endef

define Package/golang-crowdsec-firewall-bouncer-dev/description
$(call Package/crowdsec-firewall-bouncer/Default/description)

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

define Package/crowdsec-firewall-bouncer/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(GO_PKG_BUILD_DIR)/src/$(GO_PKG)/crowdsec-firewall-bouncer $(1)/usr/bin/

	$(INSTALL_DIR) $(1)/etc/crowdsec/bouncers
	$(INSTALL_DATA) \
		$(PKG_BUILD_DIR)/config/crowdsec-firewall-bouncer.yaml \
		$(1)/etc/crowdsec/bouncers

	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) \
		./files/crowdsec-firewall-bouncer.initd \
		$(1)/etc/init.d/crowdsec-firewall-bouncer

	$(INSTALL_DIR) $(1)/etc
	$(INSTALL_BIN) \
		./files/crowdsec-firewall-bouncer.firewall \
		$(1)/etc/firewall.cs
endef

define Package/crowdsec-firewall-bouncer/postinst
#!/bin/sh
if [ -z "$${IPKG_INSTROOT}" ]; then
  # Default firewall backend is iptables instead of nftables
  FW_BACKEND="iptables"
  API_KEY=""

  ## CheckFirewall
  iptables="true"
  which iptables > /dev/null
  FW_BACKEND=""
  if [[ $$? != 0 ]]; then 
    echo "iptables is not present"
    iptables="false"
  else 
    FW_BACKEND="iptables"
    echo "iptables found"
  fi

  nftables="true"
  which nft > /dev/null
  if [[ $$? != 0 ]]; then 
    echo "nftables is not present"
    nftables="false"
  else
    FW_BACKEND="nftables" 
    echo "nftables found"
  fi

  if [ "$$nftables" = "true" -a "$$iptables" = "true" ]; then
    echo "Found nftables(default) and iptables..."
  fi

  if [ "$$FW_BACKEND" = "iptables" ]; then
    which ipset > /dev/null
    if [[ $$? != 0 ]]; then
      echo "ipset not found, install it !"
    fi
  fi

  if ! uci show crowdsec | grep -q api_key; then
    ## Gen&ConfigApiKey
    SUFFIX=`tr -dc A-Za-z0-9 </dev/urandom | head -c 8`
    API_KEY=`/usr/bin/cscli bouncers add cs-firewall-bouncer-$${SUFFIX} -o raw`
    sed -i "s,^\(\s*api_key\s*:\s*\).*\$$,\1$$API_KEY," /etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml
    uci set "crowdsec.crowdsec.api_key=$$API_KEY"
    echo -e "Adding the following UCI config:\n $$(uci changes)"
    uci commit
  fi
  
  ## ConfigBackend
  sed -i "s,^\(\s*mode\s*:\s*\).*\$$,\1$$FW_BACKEND," /etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml
  uci set "crowdsec.crowdsec.backend=$$FW_BACKEND"
  echo -e "Adding the following UCI config:\n $$(uci changes)"
  uci commit

  # unfortunately, UCI doesn't provide a nice way to add an anonymous section only if it doesn't already exist
  if ! uci show firewall | grep -q firewall.cs; then
    name="$$(uci add firewall include)"
    uci set "firewall.$${name}.path=/etc/firewall.cs"
    uci set "firewall.$${name}.enabled=1"
    uci set "firewall.$${name}.reload=1"
    echo -e "Adding the following UCI config:\n $$(uci changes)"
    uci commit
  fi

  ##
  /etc/init.d/firewall restart
fi
endef

define Package/crowdsec-firewall-bouncer/prerm
#!/bin/sh
if [ -z "$${IPKG_INSTROOT}" ]; then
  /etc/init.d/crowdsec-firewall-bouncer stop
  /etc/init.d/crowdsec-firewall-bouncer disable
  rm -f /usr/bin/crowdsec-firewall-bouncer
  rm -f /etc/firewall.cs
  rm -f /etc/init.d/crowdsec-firewall-bouncer
fi
endef

define Package/crowdsec/conffiles
  /etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml
endef

$(eval $(call GoBinPackage,crowdsec-firewall-bouncer))
$(eval $(call BuildPackage,crowdsec-firewall-bouncer))

$(eval $(call GoSrcPackage,golang-crowdsec-firewall-bouncer-dev))
$(eval $(call BuildPackage,golang-crowdsec-firewall-bouncer-dev))

