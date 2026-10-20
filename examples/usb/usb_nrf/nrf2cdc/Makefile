# ===================================================================================
# Project:  NRF2CDC for CH55x (USB2NRF)
# Author:   Stefan Wagner
# Source:   https://github.com/wagiminator/CH552-USB-NRF
# ==================================================================================
# SDK adaptation: Cesar Bautista
# License:  CC BY-SA 3.0; see LICENSE
# ===================================================================================
# Type "make help" in the command line.
# ===================================================================================

# Files and directories
MAINFILE   = main.c
TARGET     = main
INCLUDE    = src
TOOLS      = tools
BUILD_DIR  = build

# Microcontroller settings
FREQ_SYS   = 16000000
XRAM_LOC   = 0x0100
XRAM_SIZE  = 0x0300
CODE_SIZE  = 0x3800

# Toolchain
CC         = sdcc
OBJCOPY    = objcopy
PACK_HEX   = packihx
ISPTOOL   ?= python3 $(TOOLS)/chprog.py $(BUILD_DIR)/$(TARGET).bin

# Compiler flags
CFLAGS  = -mmcs51 --model-small --no-xinit-opt -DF_CPU=$(FREQ_SYS) -I$(INCLUDE) -I.
CFLAGS += --xram-size $(XRAM_SIZE) --xram-loc $(XRAM_LOC) --code-size $(CODE_SIZE)

# Source files and object files
CFILES  = $(MAINFILE) $(wildcard $(INCLUDE)/*.c)
RFILES  = $(patsubst %.c,$(BUILD_DIR)/%.rel,$(notdir $(CFILES)))

# Generated files to clean
CLEAN_FILES = *.ihx *.lk *.map *.mem *.lst *.rel *.rst *.sym *.asm *.adb

# Show help
help:
	@echo "Available commands:"
	@echo "make all     -> Build binary and Intel HEX files"
	@echo "make hex     -> Generate $(TARGET).hex"
	@echo "make bin     -> Generate $(TARGET).bin"
	@echo "make flash   -> Upload firmware to CH55x"
	@echo "make clean   -> Remove generated files"

# Create the output directory
$(BUILD_DIR):
	@mkdir -p $(BUILD_DIR)

# Compile main.c
$(BUILD_DIR)/main.rel: main.c | $(BUILD_DIR)
	@echo "Compiling $< ..."
	@$(CC) -c $(CFLAGS) -o $@ $<

# Compile src/*.c
$(BUILD_DIR)/%.rel: $(INCLUDE)/%.c | $(BUILD_DIR)
	@echo "Compiling $< ..."
	@$(CC) -c $(CFLAGS) -o $@ $<

# Link objects
$(BUILD_DIR)/$(TARGET).ihx: $(RFILES)
	@echo "Linking IHX..."
	@$(CC) $(RFILES) $(CFLAGS) -o $@

# Generate HEX
$(BUILD_DIR)/$(TARGET).hex: $(BUILD_DIR)/$(TARGET).ihx
	@echo "Generating HEX..."
	@$(PACK_HEX) $< > $@

# Generate BIN
$(BUILD_DIR)/$(TARGET).bin: $(BUILD_DIR)/$(TARGET).ihx
	@echo "Generating BIN..."
	@$(OBJCOPY) -I ihex -O binary $< $@

# Upload to CH55x
flash: $(BUILD_DIR)/$(TARGET).bin size removetemp
	@echo "Uploading to CH55x..."
	@$(ISPTOOL)

# Standard targets
all: $(BUILD_DIR)/$(TARGET).bin $(BUILD_DIR)/$(TARGET).hex size

hex: $(BUILD_DIR)/$(TARGET).hex size removetemp

bin: $(BUILD_DIR)/$(TARGET).bin size removetemp

bin-hex: $(BUILD_DIR)/$(TARGET).bin $(BUILD_DIR)/$(TARGET).hex size removetemp

install: flash

# Report memory usage
size:
	@echo "------------------"
	@echo "FLASH: $(shell awk '$$1 == "ROM/EPROM/FLASH" {print $$4}' $(BUILD_DIR)/$(TARGET).mem) bytes"
	@echo "IRAM:  $(shell awk '$$1 == "Stack" {print 248-$$10}' $(BUILD_DIR)/$(TARGET).mem) bytes"
	@echo "XRAM:  $(shell awk '$$1 == "EXTERNAL" {print $(XRAM_LOC)+$$5}' $(BUILD_DIR)/$(TARGET).mem) bytes"
	@echo "------------------"

# Clean generated files
removetemp:
	@echo "Removing temporary files..."
	@rm -f $(addprefix $(BUILD_DIR)/,$(CLEAN_FILES))

clean:
	@echo "Cleaning all generated files..."
	@rm -rf $(BUILD_DIR)
