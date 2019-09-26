# Makefile for MicroPython on ESP32.
#
# This is a simple, convenience wrapper around idf.py (which uses cmake).

# Select the board to build for, defaulting to GENERIC.
BOARD ?= GENERIC

# If the build directory is not given, make it reflect the board name.
BUILD ?= build-$(BOARD)

# Device serial settings.
PORT ?= /dev/ttyUSB0
BAUD ?= 460800
FLASH_MODE ?= dio
FLASH_FREQ ?= 40m
FLASH_SIZE ?= 4MB
CROSS_COMPILE ?= xtensa-esp32-elf-
OBJDUMP = $(CROSS_COMPILE)objdump

SDKCONFIG_COMBINED = $(BUILD)/sdkconfig.combined
SDKCONFIG_H = $(BUILD)/sdkconfig.h

# the git hash of the currently supported ESP IDF version
ESPIDF_SUPHASH_V3 := 6ccb4cf5b7d1fdddb8c2492f9cbc926abaf230df
ESPIDF_SUPHASH_V4 := 310beae373446ceb9a4ad9b36b5428d7fdf2705f

define print_supported_git_hash
$(info Supported git hash (v3.3): $(ESPIDF_SUPHASH_V3))
$(info Supported git hash (v4.0-beta1) (experimental): $(ESPIDF_SUPHASH_V4))
endef

# paths to ESP IDF and its components
ifeq ($(ESPIDF),)
ifneq ($(IDF_PATH),)
ESPIDF = $(IDF_PATH)
else
$(info The ESPIDF variable has not been set, please set it to the root of the esp-idf repository.)
$(info See README.md for installation instructions.)
$(call print_supported_git_hash)
$(error ESPIDF not set)
endif
endif

ESPCOMP = $(ESPIDF)/components
ESPTOOL ?= $(ESPCOMP)/esptool_py/esptool/esptool.py
ESPCOMP_KCONFIGS = $(shell find $(ESPCOMP) -name Kconfig)
ESPCOMP_KCONFIGS_PROJBUILD = $(shell find $(ESPCOMP) -name Kconfig.projbuild)

# verify the ESP IDF version
ESPIDF_CURHASH := $(shell git -C $(ESPIDF) show -s --pretty=format:'%H')

ifeq ($(ESPIDF_CURHASH),$(ESPIDF_SUPHASH_V3))
$(info Building with ESP IDF v3)
else ifeq ($(ESPIDF_CURHASH),$(ESPIDF_SUPHASH_V4))
$(info Building with ESP IDF v4)

PYPARSING_VERSION = $(shell python3 -c 'import pyparsing; print(pyparsing.__version__)')
ifneq ($(PYPARSING_VERSION),2.3.1)
$(info ** ERROR **)
$(info EDP IDF requires pyparsing version less than 2.4)
$(info You will need to set up a Python virtual environment with pyparsing 2.3.1)
$(info Please see README.md for more information)
$(error Incorrect pyparsing version)
endif
else
$(info ** WARNING **)
$(info The git hash of ESP IDF does not match the supported version)
$(info The build may complete and the firmware may work but it is not guaranteed)
$(info ESP IDF path:       $(ESPIDF))
$(info Current git hash:   $(ESPIDF_CURHASH))
$(call print_supported_git_hash)
endif

# pretty format of ESP IDF version, used internally by the IDF
IDF_VER := $(shell git -C $(ESPIDF) describe)

ifeq ($(shell which $(CC) 2> /dev/null),)
$(info ** ERROR **)
$(info Cannot find C compiler $(CC))
$(info Add the xtensa toolchain to your PATH. See README.md)
$(error C compiler missing)
endif

# include sdkconfig to get needed configuration values
include $(SDKCONFIG)

################################################################################
# Compiler and linker flags

INC += -I.
INC += -I$(TOP)
INC += -I$(TOP)/lib/mp-readline
INC += -I$(TOP)/lib/netutils
INC += -I$(TOP)/lib/timeutils
INC += -I$(BUILD)

INC_ESPCOMP += -I$(ESPCOMP)/bootloader_support/include
INC_ESPCOMP += -I$(ESPCOMP)/bootloader_support/include_bootloader
INC_ESPCOMP += -I$(ESPCOMP)/console
INC_ESPCOMP += -I$(ESPCOMP)/driver/include
INC_ESPCOMP += -I$(ESPCOMP)/driver/include/driver
INC_ESPCOMP += -I$(ESPCOMP)/efuse/include
INC_ESPCOMP += -I$(ESPCOMP)/efuse/esp32/include
INC_ESPCOMP += -I$(ESPCOMP)/esp32/include
INC_ESPCOMP += -I$(ESPCOMP)/espcoredump/include
INC_ESPCOMP += -I$(ESPCOMP)/soc/include
INC_ESPCOMP += -I$(ESPCOMP)/soc/esp32/include
INC_ESPCOMP += -I$(ESPCOMP)/heap/include
INC_ESPCOMP += -I$(ESPCOMP)/log/include
INC_ESPCOMP += -I$(ESPCOMP)/newlib/platform_include
INC_ESPCOMP += -I$(ESPCOMP)/newlib/include
INC_ESPCOMP += -I$(ESPCOMP)/nvs_flash/include
INC_ESPCOMP += -I$(ESPCOMP)/freertos/include
INC_ESPCOMP += -I$(ESPCOMP)/esp_ringbuf/include
INC_ESPCOMP += -I$(ESPCOMP)/esp_event/include
INC_ESPCOMP += -I$(ESPCOMP)/tcpip_adapter/include
INC_ESPCOMP += -I$(ESPCOMP)/lwip/lwip/src/include
INC_ESPCOMP += -I$(ESPCOMP)/lwip/port/esp32/include
INC_ESPCOMP += -I$(ESPCOMP)/lwip/include/apps
INC_ESPCOMP += -I$(ESPCOMP)/mbedtls/mbedtls/include
INC_ESPCOMP += -I$(ESPCOMP)/mbedtls/port/include
INC_ESPCOMP += -I$(ESPCOMP)/mdns/include
INC_ESPCOMP += -I$(ESPCOMP)/mdns/private_include
INC_ESPCOMP += -I$(ESPCOMP)/spi_flash/include
INC_ESPCOMP += -I$(ESPCOMP)/ulp/include
INC_ESPCOMP += -I$(ESPCOMP)/vfs/include
INC_ESPCOMP += -I$(ESPCOMP)/xtensa-debug-module/include
INC_ESPCOMP += -I$(ESPCOMP)/wpa_supplicant/include
INC_ESPCOMP += -I$(ESPCOMP)/wpa_supplicant/port/include
INC_ESPCOMP += -I$(ESPCOMP)/app_trace/include
INC_ESPCOMP += -I$(ESPCOMP)/app_update/include
INC_ESPCOMP += -I$(ESPCOMP)/pthread/include
INC_ESPCOMP += -I$(ESPCOMP)/smartconfig_ack/include
INC_ESPCOMP += -I$(ESPCOMP)/sdmmc/include
#INC_ESPCOMP += -I$(ESPCOMP)/can/include

ifeq ($(ESPIDF_CURHASH),$(ESPIDF_SUPHASH_V4))
INC_ESPCOMP += -I$(ESPCOMP)/esp_common/include
INC_ESPCOMP += -I$(ESPCOMP)/esp_eth/include
INC_ESPCOMP += -I$(ESPCOMP)/esp_event/private_include
INC_ESPCOMP += -I$(ESPCOMP)/esp_rom/include
INC_ESPCOMP += -I$(ESPCOMP)/esp_wifi/include
INC_ESPCOMP += -I$(ESPCOMP)/esp_wifi/esp32/include
INC_ESPCOMP += -I$(ESPCOMP)/lwip/include/apps/sntp
INC_ESPCOMP += -I$(ESPCOMP)/spi_flash/private_include
INC_ESPCOMP += -I$(ESPCOMP)/wpa_supplicant/include/esp_supplicant
INC_ESPCOMP += -I$(ESPCOMP)/xtensa/include
INC_ESPCOMP += -I$(ESPCOMP)/xtensa/esp32/include
else
INC_ESPCOMP += -I$(ESPCOMP)/ethernet/include
INC_ESPCOMP += -I$(ESPCOMP)/expat/expat/expat/lib
INC_ESPCOMP += -I$(ESPCOMP)/expat/port/include
INC_ESPCOMP += -I$(ESPCOMP)/json/include
INC_ESPCOMP += -I$(ESPCOMP)/json/port/include
INC_ESPCOMP += -I$(ESPCOMP)/micro-ecc/micro-ecc
INC_ESPCOMP += -I$(ESPCOMP)/nghttp/port/include
INC_ESPCOMP += -I$(ESPCOMP)/nghttp/nghttp2/lib/includes
endif

# these flags are common to C and C++ compilation
CFLAGS_COMMON = -Os -ffunction-sections -fdata-sections -fstrict-volatile-bitfields \
	-mlongcalls -nostdlib \
	-Wall -Werror -Wno-error=unused-function -Wno-error=unused-but-set-variable \
	-Wno-error=unused-variable -Wno-error=deprecated-declarations \
	-DESP_PLATFORM

CFLAGS_BASE = -std=gnu99 $(CFLAGS_COMMON) -DMBEDTLS_CONFIG_FILE='"mbedtls/esp_config.h"' -DHAVE_CONFIG_H
CFLAGS = $(CFLAGS_BASE) $(INC) $(INC_ESPCOMP)
CFLAGS += -DIDF_VER=\"$(IDF_VER)\"
CFLAGS += $(CFLAGS_MOD) $(CFLAGS_EXTRA)
CFLAGS += -I$(BOARD_DIR)

ifeq ($(ESPIDF_CURHASH),$(ESPIDF_SUPHASH_V4))
CFLAGS += -DMICROPY_ESP_IDF_4=1
endif

PYTHON ?= python3

GIT_SUBMODULES = lib/berkeley-db-1.xx

.PHONY: all clean deploy erase submodules FORCE

CMAKE_ARGS =

ifdef USER_C_MODULES
	CMAKE_ARGS += -DUSER_C_MODULES=${USER_C_MODULES}
endif

################################################################################
# List of MicroPython source and object files

SRC_C = \
	main.c \
	uart.c \
	gccollect.c \
	mphalport.c \
	fatfs_port.c \
	help.c \
	modutime.c \
	moduos.c \
	machine_timer.c \
	machine_pin.c \
	machine_touchpad.c \
	machine_adc.c \
	machine_dac.c \
	machine_i2c.c \
	machine_pwm.c \
	machine_uart.c \
	machine_can.c \
	modmachine.c \
	modnetwork.c \
	network_lan.c \
	network_ppp.c \
	modsocket.c \
	modesp.c \
	esp32_partition.c \
	esp32_ulp.c \
	modesp32.c \
	espneopixel.c \
	machine_hw_spi.c \
	machine_wdt.c \
	mpthreadport.c \
	machine_rtc.c \
	machine_sdcard.c \
	$(wildcard $(BOARD_DIR)/*.c) \
	$(SRC_MOD)

EXTMOD_SRC_C = $(addprefix extmod/,\
	modonewire.c \
	)

LIB_SRC_C = $(addprefix lib/,\
	mp-readline/readline.c \
	netutils/netutils.c \
	timeutils/timeutils.c \
	utils/pyexec.c \
	utils/interrupt_char.c \
	utils/sys_stdio_mphal.c \
	)

ifdef FROZEN_MANIFEST
       IDFPY_FLAGS += -D MICROPY_FROZEN_MANIFEST=$(FROZEN_MANIFEST)
endif

all:
	idf.py $(IDFPY_FLAGS) build
	@$(PYTHON) makeimg.py \
		$(BUILD)/sdkconfig \
		$(BUILD)/bootloader/bootloader.bin \
		$(BUILD)/partition_table/partition-table.bin \
    		$(BUILD)/micropython.bin \
    		$(BUILD)/firmware.bin

$(BUILD)/bootloader/bootloader.bin $(BUILD)/partition_table/partition-table.bin $(BUILD)/micropython.bin: FORCE

clean:
	idf.py $(IDFPY_FLAGS) fullclean

deploy:
	idf.py $(IDFPY_FLAGS) -p $(PORT) -b $(BAUD) flash

erase:
	idf.py $(IDFPY_FLAGS) -p $(PORT) -b $(BAUD) erase_flash

submodules:
	git submodule update --init $(addprefix ../../,$(GIT_SUBMODULES))
