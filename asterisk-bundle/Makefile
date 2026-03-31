include $(TOPDIR)/rules.mk

PKG_NAME:=asterisk20-pjsip-bundle
PKG_VERSION:=20.5.0
PKG_RELEASE:=1

PKG_SOURCE:=asterisk-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://downloads.asterisk.org/pub/telephony/asterisk/releases/
PKG_HASH:=skip

PKG_FIXUP:=autoreconf
PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk

define Package/asterisk20-pjsip-bundle
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Asterisk 20 with Forced Bundled PJSIP
  DEPENDS:=+libedit +libopenssl +libncurses +libpthread +libuuid +libxml2 \
           +sqlite3-cli +libsqlite3 +libtirpc +libcap +libiconv +alsa-lib
endef

# We keep all the Gateway flags (iconv, srv, etc.) so the CORE supports GSM
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
	--with-tonezone

define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR) $(MAKE_FLAGS) ASTCFLAGS="-D_GNU_SOURCE"
endef

define Package/asterisk20-pjsip-bundle/install
	$(INSTALL_DIR) $(1)/usr/sbin $(1)/usr/lib/asterisk/modules $(1)/etc/asterisk $(1)/etc/init.d
	
	# Install Binary
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/main/asterisk $(1)/usr/sbin/
	
	# Install ALL PJSIP and Core modules into this single IPK
	$(CP) $(PKG_BUILD_DIR)/channels/chan_pjsip.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/res/res_pjsip*.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/res/res_pjproject.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/res/res_rtp_asterisk.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/res/res_sorcery*.so $(1)/usr/lib/asterisk/modules/
	
	# Install Codecs (Necessary for GSM gateway)
	$(CP) $(PKG_BUILD_DIR)/codecs/codec_gsm.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/codecs/codec_alaw.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/codecs/codec_ulaw.so $(1)/usr/lib/asterisk/modules/

	# Configs & Init Script
	$(CP) ./files/*.conf $(1)/etc/asterisk/ 2>/dev/null || true
	$(INSTALL_BIN) ./files/asterisk.init $(1)/etc/init.d/asterisk 2>/dev/null || true
endef

$(eval $(call BuildPackage,asterisk20-pjsip-bundle))
