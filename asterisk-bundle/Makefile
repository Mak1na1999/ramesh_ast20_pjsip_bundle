include $(TOPDIR)/rules.mk

PKG_NAME:=asterisk20-gateway-bundle
PKG_VERSION:=20.5.0
PKG_RELEASE:=1

PKG_SOURCE:=asterisk-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://downloads.asterisk.org/pub/telephony/asterisk/releases/
PKG_HASH:=skip

CHAN_QUECTEL_URL:=https://github.com/RoEdAl/asterisk-chan-quectel.git

PKG_FIXUP:=autoreconf
PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk

define Package/asterisk20-gateway-bundle
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Asterisk 20 GSM Gateway for OpenWrt 24.10
  DEPENDS:=+libedit +libopenssl +libncurses +libpthread +libuuid +libxml2 +sqlite3-cli +libsqlite3 +libtirpc +libcap +kmod-usb-serial-option +kmod-usb-serial-wwan +usbutils +libiconv +alsa-lib
endef

# Using bundled PJSIP and Jansson to avoid version conflicts in 24.10
CONFIGURE_ARGS += \
	--with-pjproject-bundled \
	--with-jansson-bundled \
	--with-sqlite3 \
	--with-ssl \
	--without-oss \
	--disable-xmldoc \
	--with-libedit \
	--with-gsm \
	--with-iconv \
	--with-srv \
	--with-asound \
	--with-tonezone \
	--without-dahdi

define Build/Prepare
	$(call Build/Prepare/Default)
	git clone $(CHAN_QUECTEL_URL) $(PKG_BUILD_DIR)/chan-quectel-src
endef

define Build/Compile
	# 1. Build Asterisk Core first (Required for headers)
	$(MAKE) -C $(PKG_BUILD_DIR) $(MAKE_FLAGS) ASTCFLAGS="-D_GNU_SOURCE"
	
	# 2. Create a symlink so the driver finds the headers where it expects them
	ln -sf $(PKG_BUILD_DIR)/include $(PKG_BUILD_DIR)/chan-quectel-src/include
	
	# 3. Build chan-quectel
	$(MAKE) -C $(PKG_BUILD_DIR)/chan-quectel-src \
		$(MAKE_FLAGS) \
		ASTERISK_INCLUDE_DIR=$(PKG_BUILD_DIR)/include \
		all
endef

define Package/asterisk20-gateway-bundle/install
    $(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/asterisk.init $(1)/etc/init.d/asterisk
	$(INSTALL_DIR) $(1)/usr/sbin $(1)/usr/lib/asterisk/modules $(1)/etc/asterisk
	
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/main/asterisk $(1)/usr/sbin/
	
	# Install PJSIP and Core Modules
	$(CP) $(PKG_BUILD_DIR)/channels/chan_pjsip.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/res/res_pjsip*.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/res/res_pjproject.so $(1)/usr/lib/asterisk/modules/
    $(CP) $(PKG_BUILD_DIR)/codecs/codec_gsm.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/codecs/codec_alaw.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/codecs/codec_ulaw.so $(1)/usr/lib/asterisk/modules/
	
	# Install Source-Compiled Quectel Module
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/chan-quectel-src/chan_quectel.so $(1)/usr/lib/asterisk/modules/
	
	# Copy local configs from /files folder
	$(CP) ./files/* $(1)/etc/asterisk/ 2>/dev/null || true
endef

$(eval $(call BuildPackage,asterisk20-gateway-bundle))
