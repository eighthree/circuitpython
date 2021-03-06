# Select the board to build for: if not given on the command line,
# then default to PYBV10.
BOARD ?= feather_huzzah
ifeq ($(wildcard boards/$(BOARD)/.),)
$(error Invalid BOARD specified)
endif

# If the build directory is not given, make it reflect the board name.
BUILD ?= build-$(BOARD)

include ../../py/mkenv.mk

# qstr definitions (must come before including py.mk)
QSTR_DEFS = qstrdefsport.h #$(BUILD)/pins_qstr.h

MICROPY_PY_USSL = 1
MICROPY_SSL_AXTLS = 1
MICROPY_FATFS = 1
MICROPY_PY_BTREE = 1
BTREE_DEFS_EXTRA = -DDEFPSIZE=1024 -DMINCACHE=3

FROZEN_DIR ?= scripts
FROZEN_MPY_DIR ?= modules

# include py core make definitions
include $(TOP)/py/py.mk

FWBIN = $(BUILD)/firmware.bin
PORT ?= /dev/ttyACM0
BAUD ?= 115200
FLASH_MODE ?= qio
FLASH_SIZE ?= detect
CROSS_COMPILE = xtensa-lx106-elf-
ESP_SDK = $(shell $(CC) -print-sysroot)/usr
ESPTOOL = esptool.py

INC += -I.
INC += -I$(TOP)
INC += -I$(BUILD)
INC += -I$(ESP_SDK)/include

# UART for "os" messages. 0 is normal UART as used by MicroPython REPL,
# 1 is debug UART (tx only), -1 to disable.
UART_OS = 0

CFLAGS_XTENSA = -fsingle-precision-constant -Wdouble-promotion \
	-D__ets__ -DICACHE_FLASH \
	-fno-inline-functions \
	-Wl,-EL -mlongcalls -mtext-section-literals -mforce-l32 \
	-DLWIP_OPEN_SRC

CFLAGS = $(INC) -Wall -Wpointer-arith -Werror -Wno-strict-aliasing -std=gnu99 -nostdlib -DUART_OS=$(UART_OS) \
	$(CFLAGS_XTENSA) $(CFLAGS_MOD) $(COPT) $(CFLAGS_EXTRA)

LDSCRIPT = esp8266.ld
LDFLAGS = -nostdlib -T $(LDSCRIPT) -Map=$(@:.elf=.map) --cref
LIBS = -L$(ESP_SDK)/lib -lmain -ljson -llwip_open -lpp -lnet80211 -lwpa -lphy -lnet80211 $(LDFLAGS_MOD)

LIBGCC_FILE_NAME = $(shell $(CC) $(CFLAGS) -print-libgcc-file-name)
LIBS += -L$(dir $(LIBGCC_FILE_NAME)) -lgcc

# Debugging/Optimization
ifeq ($(DEBUG), 1)
CFLAGS += -g
COPT = -O0
else
CFLAGS += -fdata-sections -ffunction-sections
COPT += -Os -DNDEBUG
LDFLAGS += --gc-sections
endif

SRC_C = \
	strtoll.c \
	main.c \
	help.c \
	esp_mphal.c \
	esp_init_data.c \
	gccollect.c \
	lexerstr32.c \
	uart.c \
	esppwm.c \
	espneopixel.c \
	intr.c \
	modpyb.c \
	modmachine.c \
	machine_pin.c \
	machine_pwm.c \
	machine_rtc.c \
	machine_adc.c \
	machine_uart.c \
	machine_wdt.c \
	machine_hspi.c \
	modesp.c \
	modnetwork.c \
	ets_alt_task.c \
	fatfs_port.c \
	posix_helpers.c \
	hspi.c \
	boards/$(BOARD)/pins.c \
	supervisor/stub/stack.c \
	supervisor/shared/translate.c \
	$(SRC_MOD)

SRC_COMMON_HAL = \
	microcontroller/__init__.c \
	microcontroller/Pin.c \
	microcontroller/Processor.c \
	analogio/__init__.c \
	analogio/AnalogIn.c \
	analogio/AnalogOut.c \
	digitalio/__init__.c \
	digitalio/DigitalInOut.c \
	pulseio/__init__.c \
	pulseio/PulseIn.c \
	pulseio/PulseOut.c \
	pulseio/PWMOut.c \
	busio/__init__.c \
	busio/SPI.c \
	busio/UART.c \
	multiterminal/__init__.c \
	neopixel_write/__init__.c \
	os/__init__.c \
	time/__init__.c \
	board/__init__.c


# These don't have corresponding files in each port but are still located in
# shared-bindings to make it clear what the contents of the modules are.
SRC_BINDINGS_ENUMS = \
	digitalio/Direction.c \
	digitalio/DriveMode.c \
	digitalio/Pull.c \
	math/__init__.c \
	microcontroller/RunMode.c \
	util.c

SRC_COMMON_HAL_EXPANDED = $(addprefix shared-bindings/, $(SRC_COMMON_HAL)) \
                          $(addprefix shared-bindings/, $(SRC_BINDINGS_ENUMS)) \
                          $(addprefix common-hal/, $(SRC_COMMON_HAL))
SRC_SHARED_MODULE = \
	bitbangio/__init__.c \
	bitbangio/I2C.c \
	bitbangio/OneWire.c \
	bitbangio/SPI.c \
	busio/I2C.c \
	busio/OneWire.c \
	multiterminal/__init__.c \
	os/__init__.c \
	random/__init__.c \
	struct/__init__.c

SRC_SHARED_MODULE_EXPANDED = $(addprefix shared-bindings/, $(SRC_SHARED_MODULE)) \
                             $(addprefix shared-module/, $(SRC_SHARED_MODULE))

EXTMOD_SRC_C = $(addprefix extmod/,\
	modlwip.c \
	modonewire.c \
        )

LIB_SRC_C = $(addprefix lib/,\
	libc/string0.c \
	libm/math.c \
	libm/fmodf.c \
	libm/nearbyintf.c \
	libm/ef_sqrt.c \
	libm/kf_rem_pio2.c \
	libm/kf_sin.c \
	libm/kf_cos.c \
	libm/kf_tan.c \
	libm/ef_rem_pio2.c \
	libm/sf_sin.c \
	libm/sf_cos.c \
	libm/sf_tan.c \
	libm/sf_frexp.c \
	libm/sf_modf.c \
	libm/sf_ldexp.c \
	libm/asinfacosf.c \
	libm/atanf.c \
	libm/atan2f.c \
	mp-readline/readline.c \
	netutils/netutils.c \
	timeutils/timeutils.c \
	utils/buffer_helper.c \
	utils/context_manager_helpers.c \
	utils/pyexec.c \
	utils/interrupt_char.c \
	utils/sys_stdio_mphal.c \
	)

ifeq ($(MICROPY_FATFS), 1)
LIB_SRC_C += \
	lib/oofatfs/ff.c \
	lib/oofatfs/option/unicode.c
endif

DRIVERS_SRC_C = $(addprefix drivers/,\
	bus/softspi.c \
	)

SRC_S = \
	gchelper.s \

OBJ =
OBJ += $(PY_O)
OBJ += $(addprefix $(BUILD)/, $(SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_S:.s=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_COMMON_HAL_EXPANDED:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_SHARED_MODULE_EXPANDED:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(STM_SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(EXTMOD_SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(LIB_SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(DRIVERS_SRC_C:.c=.o))

# List of sources for qstr extraction
SRC_QSTR += $(SRC_C) $(SRC_COMMON_HAL_EXPANDED) $(SRC_SHARED_MODULE_EXPANDED) $(STM_SRC_C) $(EXTMOD_SRC_C) $(DRIVERS_SRC_C)
# Append any auto-generated sources that are needed by sources listed in SRC_QSTR
SRC_QSTR_AUTO_DEPS +=

all: $(BUILD)/libaxtls.a $(FWBIN)

CONFVARS_FILE = $(BUILD)/confvars

ifeq ($(wildcard $(CONFVARS_FILE)),)
$(shell $(MKDIR) -p $(BUILD))
$(shell echo $(FROZEN_DIR) $(UART_OS) > $(CONFVARS_FILE))
else ifneq ($(shell cat $(CONFVARS_FILE)), $(FROZEN_DIR) $(UART_OS))
$(shell echo $(FROZEN_DIR) $(UART_OS) > $(CONFVARS_FILE))
endif

$(BUILD)/uart.o: $(CONFVARS_FILE)

FROZEN_EXTRA_DEPS = $(CONFVARS_FILE)

.PHONY: deploy

deploy: $(FWBIN)
	$(ECHO) "Writing $< to the board"
	$(Q)$(ESPTOOL) --port $(PORT) --baud $(BAUD) write_flash --verify --flash_size=$(FLASH_SIZE) --flash_mode=$(FLASH_MODE) 0 $<

erase:
	$(ECHO) "Erase flash"
	$(Q)$(ESPTOOL) --port $(PORT) --baud $(BAUD) erase_flash

reset:
	echo -e "\r\nimport machine; machine.reset()\r\n" >$(PORT)

$(FWBIN): $(BUILD)/firmware.elf
	$(ECHO) "Create $@"
	$(Q)$(ESPTOOL) elf2image $^
	$(Q)$(PYTHON) makeimg.py $(BUILD)/firmware.elf-0x00000.bin $(BUILD)/firmware.elf-0x[0-5][1-f]000.bin $@


$(BUILD)/firmware.elf: $(OBJ)
	$(STEPECHO) "LINK $@"
	$(Q)$(LD) $(LDFLAGS) -o $@ $^ $(LIBS)
	$(Q)$(SIZE) $@

512k:
	$(MAKE) LDSCRIPT=esp8266_512k.ld CFLAGS_EXTRA='-DMP_CONFIGFILE="<mpconfigport_512k.h>"' MICROPY_FATFS=0 MICROPY_PY_BTREE=0

ota:
	rm -f $(BUILD)/firmware.elf $(BUILD)/firmware.elf*.bin
	$(MAKE) LDSCRIPT=esp8266_ota.ld FWBIN=$(BUILD)/firmware-ota.bin

include $(TOP)/py/mkrules.mk

axtls: $(BUILD)/libaxtls.a

$(BUILD)/libaxtls.a:
	cd $(TOP)/lib/axtls; cp config/upyconfig config/.config
	cd $(TOP)/lib/axtls; $(MAKE) ssl/version.h
	cd $(TOP)/lib/axtls; $(MAKE) oldconfig -B
	cd $(TOP)/lib/axtls; $(MAKE) clean
	cd $(TOP)/lib/axtls; $(MAKE) all CC="$(CC)" LD="$(LD)" AR="$(AR)" CFLAGS_EXTRA="$(CFLAGS_XTENSA) -Dabort=abort_ -DRT_MAX_PLAIN_LENGTH=1024 -DRT_EXTRA=4096"
	cp $(TOP)/lib/axtls/_stage/libaxtls.a $@

clean-modules:
	git clean -f -d modules
	rm -f $(BUILD)/frozen*.c
