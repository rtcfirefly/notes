To fetch and build `systemd` as a custom package, you can use the following `custom_systemd.mk` example. This will download, configure, and build `systemd` from its source repository.

```makefile
# custom_systemd.mk

CUSTOM_SYSTEMD_VERSION = 249 # Replace with the version you want
CUSTOM_SYSTEMD_SITE = https://github.com/systemd/systemd/archive/refs/tags/v$(CUSTOM_SYSTEMD_VERSION).tar.gz
CUSTOM_SYSTEMD_LICENSE = LGPL-2.1+
CUSTOM_SYSTEMD_LICENSE_FILES = LICENSE.LGPL2.1
CUSTOM_SYSTEMD_DEPENDENCIES = host-pkgconf libcap libkmod libseccomp

define CUSTOM_SYSTEMD_BUILD_CMDS
	# Navigate to the source directory and run build commands
	(cd $(@D); ./autogen.sh)
	(cd $(@D); ./configure $(TARGET_CONFIGURE_OPTS))
	$(MAKE) $(TARGET_CONFIGURE_OPTS) -C $(@D)
endef

define CUSTOM_SYSTEMD_INSTALL_TARGET_CMDS
	# Install systemd binary to the target root filesystem
	# You can customize this part to install only the components you need
	$(INSTALL) -D -m 0755 $(@D)/src/core/systemd $(TARGET_DIR)/usr/sbin/systemd-namespace
	# Optionally, copy other required systemd utilities and libraries
	# $(INSTALL) -D -m 0755 $(@D)/src/libsystemd.so* $(TARGET_DIR)/usr/lib/
endef

$(eval $(generic-package))
```

Remember to include a `Config.in` file in the same directory as your `.mk` file so that the package can be selected in `make menuconfig`.

```makefile
# Config.in

config BR2_PACKAGE_CUSTOM_SYSTEMD
	bool "custom_systemd"
	depends on BR2_USE_MMU  # Depends on using Memory Management Unit
	select BR2_PACKAGE_LIBCAP
	select BR2_PACKAGE_LIBKMOD
	select BR2_PACKAGE_LIBSECCOMP
	help
	  Custom systemd package for PID namespace.
```

Finally, add a reference to this new package in Buildroot's main `Config.in`:

```makefile
# Add the following line to include your custom package
source "package/custom_systemd/Config.in"
```

Run `make menuconfig`, navigate to your custom package and enable it, then run `make` to build. This will compile `systemd` and install it in your target root filesystem as `systemd-namespace`.
