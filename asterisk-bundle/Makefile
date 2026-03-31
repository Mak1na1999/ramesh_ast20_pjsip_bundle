include $(TOPDIR)/rules.mk

PKG_NAME:=asterisk20-gateway-bundle
PKG_VERSION:=20.5.0
PKG_RELEASE:=1

PKG_SOURCE:=asterisk-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://downloads.asterisk.org/pub/telephony/asterisk/releases/
PKG_HASH:=skip

# Source for the Quectel/SIMCom driver
CHAN_QUECTEL_URL:=https://github.com/RoEdAl/asterisk-chan-quectel.git

PKG_FIXUP:=autoreconf
PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk

define Package/asterisk20-gateway-bundle
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Asterisk 20 GSM Gateway (Source Built)
  DEPENDS:=+libedit +libopenssl +libncurses +libpthread +libuuid +libxml2 +sqlite3-cli +libsqlite3 +libtirpc +libcap +kmod-usb-serial-option +kmod-usb-serial-wwan +usbutils
endef

CONFIGURE_ARGS += \
	--with-pjproject-bundled \
	--with-jansson-bundled \
	--with-sqlite3 \
	--with-ssl \
	--without-oss \
	--disable-xmldoc \
	--with-libedit \
	--with-gsm
	--with-asound \
    --with-iconv \
    --with-srv \
    --with-daudi \
    --with-tonezone

define Build/Prepare
	$(call Build/Prepare/Default)
	# Clone the driver source into a subdirectory
	git clone $(CHAN_QUECTEL_URL) $(PKG_BUILD_DIR)/chan-quectel-src
endef

define Build/Compile
	# 1. Compile Asterisk Core
	$(MAKE) -C $(PKG_BUILD_DIR) $(MAKE_FLAGS) ASTCFLAGS="-D_GNU_SOURCE"
	
	# 2. Compile chan-quectel against the freshly built Asterisk headers
	$(MAKE) -C $(PKG_BUILD_DIR)/chan-quectel-src \
		$(MAKE_FLAGS) \
		ASTERISK_INCLUDE_DIR=$(PKG_BUILD_DIR)/include \
		all
endef

define Package/asterisk20-gateway-bundle/install
	$(INSTALL_DIR) $(1)/usr/sbin $(1)/usr/lib/asterisk/modules $(1)/etc/asterisk
	
	# Install Asterisk Binaries
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/main/asterisk $(1)/usr/sbin/
	
	# Install PJSIP Modules
	$(CP) $(PKG_BUILD_DIR)/channels/chan_pjsip.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/res/res_pjsip*.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/res/res_pjproject.so $(1)/usr/lib/asterisk/modules/
	
	# Install the Source-Compiled Quectel Module
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/chan-quectel-src/chan_quectel.so $(1)/usr/lib/asterisk/modules/
	
	# Install Configs from your 'files' directory
	$(CP) ./files/* $(1)/etc/asterisk/ 2>/dev/null || true
endef

$(eval $(call BuildPackage,asterisk20-gateway-bundle))
