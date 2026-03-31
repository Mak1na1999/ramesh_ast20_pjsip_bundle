include $(TOPDIR)/rules.mk

PKG_NAME:=asterisk20-gateway-bundle
PKG_VERSION:=20.5.0
PKG_RELEASE:=1

PKG_SOURCE:=asterisk-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://downloads.asterisk.org/pub/telephony/asterisk/releases/
PKG_HASH:=skip

# The specific driver you requested
QUECTEL_IPK_URL:=https://github.com/RoEdAl/asterisk-chan-quectel/releases/download/v2023.12.13/asterisk-chan-quectel_2023.12.13_aarch64_cortex-a53.ipk

PKG_FIXUP:=autoreconf
PKG_BUILD_PARALLEL:=1

include $(INCLUDE_DIR)/package.mk

define Package/asterisk20-gateway-bundle
  SECTION:=net
  CATEGORY:=Network
  TITLE:=Asterisk 20 GSM Gateway for Flint 2
  # Core dependencies for SIM7600 and Asterisk 20
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

define Build/Prepare
	$(call Build/Prepare/Default)
	# Download the driver IPK directly into the build directory
	curl -L $(QUECTEL_IPK_URL) -o $(PKG_BUILD_DIR)/quectel_driver.ipk
	# Extract the chan_quectel.so from the IPK's data archive
	mkdir -p $(PKG_BUILD_DIR)/ext_modules
	( cd $(PKG_BUILD_DIR)/ext_modules && \
	  ar x ../quectel_driver.ipk && \
	  tar -xzf data.tar.gz )
endef

define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR) $(MAKE_FLAGS) ASTCFLAGS="-D_GNU_SOURCE"
endef

define Package/asterisk20-gateway-bundle/install
	$(INSTALL_DIR) $(1)/usr/sbin $(1)/usr/lib/asterisk/modules $(1)/etc/asterisk
	
	# Install Compiled Asterisk Components
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/main/asterisk $(1)/usr/sbin/
	$(CP) $(PKG_BUILD_DIR)/channels/chan_pjsip.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/res/res_pjsip*.so $(1)/usr/lib/asterisk/modules/
	$(CP) $(PKG_BUILD_DIR)/res/res_pjproject.so $(1)/usr/lib/asterisk/modules/
	
	# Install the Pre-compiled Quectel Module from the extracted folder
	$(CP) $(PKG_BUILD_DIR)/ext_modules/usr/lib/asterisk/modules/chan_quectel.so $(1)/usr/lib/asterisk/modules/
	
	# Copy your custom config files
	$(CP) ./files/* $(1)/etc/asterisk/ 2>/dev/null || true
endef

$(eval $(call BuildPackage,asterisk20-gateway-bundle))
