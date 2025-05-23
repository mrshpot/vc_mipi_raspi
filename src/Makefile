################################################################################
# Makefile
#
# Mipi Module Sensor Driver Vision Components GmbH Ettlingen 2019-2022
#
# Author: Michael Steinel, MBE (Martin Engel)
#
################################################################################
DTC ?= dtc
DTC_FLAGS ?= -@ -I dts -O dtb -Wno-unit_address_vs_reg



SENSOR_NAMES := vc-mipi-bcm2712-cam0
SENSOR_NAMES += vc-mipi-bcm2712-cam1
SENSOR_NAMES += vc-mipi-common-memory-contiguous


################################################################################

UBUNTU_CONFIG		= /boot/firmware/config.txt
UBUNTU_LAYOUT		= $(shell [ -f $(UBUNTU_CONFIG) ] && echo 1 || echo 0 )
ifeq ($(UBUNTU_LAYOUT), 1)
BOOT_OVERLAYS_DIR	= /boot/firmware/overlays
else
BOOT_OVERLAYS_DIR	= /boot/overlays
endif

KERNEL_I2C_DIR := /lib/modules/$(shell uname -r)/kernel/drivers/media/i2c/

SENSORDRIVERS := vc_mipi_core/vc_mipi_core.ko
SENSORDRIVERS += vc_mipi_core/vc_mipi_modules.ko
SENSORDRIVERS += vc_mipi_camera/vc_mipi_camera.ko
SENSOR_DRIVER := vc_mipi_core
SENSOR_DRIVER += vc_mipi_modules
SENSOR_DRIVER += vc_mipi_camera
DEVICETREES	:= $(foreach sentype,$(SENSOR_NAMES),overlays/$(sentype).dtbo)
DEVICETREES_RM := $(foreach sentype,$(SENSOR_NAMES),$(sentype).dtbo)

obj-m := vc_mipi_core/
obj-m += vc_mipi_camera/

.PHONY: all copy-overlays



dtbo: $(DEVICETREES)
	@echo "\n### DTBOs will be installed in $(BOOT_OVERLAYS_DIR) ###\n"


%.dtbo: %-overlay.dts
	$(DTC) $(DTC_FLAGS) -o $@ $<

# Add rule to build .dtbo from .dts
overlays/%.dtbo: overlays/%.dts
	$(DTC) $(DTC_FLAGS) -o $@ $<

overlays/vc-mipi-bcm2712-cam0.dtbo: overlays/vc-mipi-bcm2712-cam0-overlay.dts
	dtc -W no-unit_address_vs_reg -@ -I dts -O dtb  -o $@  $<

KERNEL_HEADERS := /home/vc/linux

# Debug information
$(info KERNEL_HEADERS=$(KERNEL_HEADERS))
$(info Checking for required files in $(KERNEL_HEADERS)...)

# Check for required files
REQUIRED_FILES := scripts/Makefile.build scripts/basic/Makefile include/config/auto.conf.cmd
MISSING_FILES := $(foreach file,$(REQUIRED_FILES),$(if $(wildcard $(KERNEL_HEADERS)/$(file)),,$(file)))

ifeq ($(MISSING_FILES),)
$(info All required files are present.)
else
endif

ifeq ($(wildcard $(KERNEL_HEADERS)),)
$(error Kernel headers not found. Please install them with 'sudo apt-get install raspberrypi-kernel-headers')
endif

build:
	$(MAKE) -C $(KERNEL_HEADERS) M=$(PWD) modules

all: $(DEVICETREES) build

clean:
	$(MAKE) -C $(KERNEL_HEADERS) M=$(PWD) clean
	-rm -f $(SENSORDRIVERS)
	-rm -f $(DEVICETREES)

reload:
	-sudo modprobe -r bcm2835-unicam
	-sudo modprobe -r rp1_cfe 
	-sudo modprobe -r vc_mipi_camera
	-sudo modprobe -r vc_mipi_modules
	-sudo modprobe -r vc_mipi_core

	-sudo modprobe bcm2835-unicam
	-sudo modprobe rp1_cfe verbose_debug=1
	-sudo modprobe vc_mipi_core
	-sudo modprobe vc_mipi_modules
	-sudo modprobe vc_mipi_camera
unload:
	-sudo modprobe -r vc_mipi_camera
	-sudo modprobe -r vc_mipi_modules
	-sudo modprobe -r vc_mipi_core


dev:	clean all install reload

install: all
	sudo install -p -m 644  $(SENSORDRIVERS)	/lib/modules/$(shell uname -r)/kernel/drivers/media/i2c/
	sudo install -p -m 644  $(DEVICETREES)	 $(BOOT_OVERLAYS_DIR)/
	sudo /sbin/depmod -a $(shell uname -r)
	sudo cp overlays/*.txt /boot/firmware/
	sudo mkdir -p /usr/local/bin
	sudo cp set_rpi5_pipeline.sh /usr/local/bin
	sudo chmod +x /usr/local/bin/set_rpi5_pipeline.sh
	sudo cp 99-camera-setup.rules /etc/udev/rules.d/
	sudo cp vc-config /usr/local/bin/
	sudo chmod +x /usr/local/bin/vc-config
	grep -qxF "include config_vc-mipi-driver-bcm2712.txt" /boot/firmware/config.txt || echo "include config_vc-mipi-driver-bcm2712.txt" | sudo tee -a /boot/firmware/config.txt
uninstall:
	sudo rm -f $(foreach sentype,$(SENSOR_NAMES),$(KERNEL_I2C_DIR)/$(sentype).ko)
	sudo rm -f $(foreach sencommon,$(SENSOR_DRIVER),$(KERNEL_I2C_DIR)/$(sencommon).ko)
	sudo rm -f $(foreach dtbo,$(DEVICETREES_RM),$(BOOT_OVERLAYS_DIR)/$(dtbo))
	sudo /sbin/depmod -a $(shell uname -r)
	@echo "--------------------------------------"
	@echo
	@echo "  Call  debian/postrm  for deactivating the configuration file in your /boot/config.txt"
	@echo "  You may remove the configuration file /boot/config_vc-mipi-driver-bcm2712.txt for cleanup"
	@echo
	@echo "--------------------------------------"


$(KERNEL_BUILD_DIR):
	@echo "--------------------------------------"
	@echo
	@echo "  Error:  Cannot continue due to the absence of the directory"
	@echo
	@echo "	$(KERNEL_BUILD_DIR)"
	@echo
	@echo "  You can try to fix it by installing the kernel header files package."
	@echo "  The following command may do so:"
	@echo
	@echo "	sudo  apt-get install  raspberrypi-kernel-headers"
	@echo
	@echo "--------------------------------------"
	@false


