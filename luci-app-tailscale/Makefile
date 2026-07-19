# SPDX-License-Identifier: GPL-3.0-only
#
# Copyright (C) 2024 asvow

include $(TOPDIR)/rules.mk

LUCI_TITLE:=LuCI for Tailscale
# Upstream declares +tailscale. That package builds tailscaled from Go source, which
# is impractical to compile inside the SDK/CI for this feed — the image installs the
# official prebuilt 'tailscale' package instead, so it's not a build dependency here.
LUCI_DEPENDS:=
LUCI_PKGARCH:=all

PKG_VERSION:=1.2.6
PKG_RELEASE:=1

# This app ships its own tailscale init script + config (a richer, UI-managed schema
# than the stock tailscale package's). Both packages would otherwise own
# /etc/init.d/tailscale and /etc/config/tailscale — apk forbids that. So we ship ours
# under /usr/share and drop them into place from postinst (overwriting the stock ones).
define Package/luci-app-tailscale/postinst
#!/bin/sh
[ -n "$${IPKG_INSTROOT}" ] || {
	cp -f /usr/share/luci-app-tailscale/tailscale.init /etc/init.d/tailscale
	chmod 0755 /etc/init.d/tailscale
	if [ ! -f /etc/config/tailscale.luci-app-installed ]; then
		cp -f /usr/share/luci-app-tailscale/tailscale.config /etc/config/tailscale
		touch /etc/config/tailscale.luci-app-installed
	fi
	rm -f /tmp/luci-indexcache /tmp/luci-modulecache/* 2>/dev/null
	killall -HUP rpcd 2>/dev/null
	exit 0
}
endef

include $(TOPDIR)/feeds/luci/luci.mk

# call BuildPackage - OpenWrt buildroot signature
