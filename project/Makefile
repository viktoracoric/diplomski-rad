# =========================================================
# FreeRTOS RISC-V QEMU Demo (clean build system)
# =========================================================

CROSS ?= riscv64-elf-
CC      = $(CROSS)gcc
OBJCOPY = $(CROSS)objcopy
AR      = $(CROSS)ar

# ---------------------------------------------------------
# Build directories
# ---------------------------------------------------------

BUILD_DIR = build

# ---------------------------------------------------------
# FreeRTOS paths
#
# Assumed structure:
#
# ~/project/
# ├── FreeRTOS/
# │   ├── include/
# │   ├── portable/
# │   ├── tasks.c
# │   └── ...
# └── RISC-V-Qemu-virt_GCC/
#     ├── Makefile
#     └── ...
# ---------------------------------------------------------

RTOS_SOURCE_DIR = ./FreeRTOS
DEMO_SOURCE_DIR = ./FreeRTOS/examples/template_configuration

# ---------------------------------------------------------
# Includes
# ---------------------------------------------------------

CPPFLAGS = \
        -D__riscv_float_abi_soft \
        -DportasmHANDLE_INTERRUPT=handle_trap \
        -I. \
        -I$(RTOS_SOURCE_DIR) \
        -I$(RTOS_SOURCE_DIR)/include \
        -I$(RTOS_SOURCE_DIR)/portable/GCC/RISC-V \
        -I$(RTOS_SOURCE_DIR)/portable/GCC/RISC-V/chip_specific_extensions/RV32I_CLINT_no_extensions

# ---------------------------------------------------------
# Compiler flags
# ---------------------------------------------------------

CFLAGS = \
	-mabi=ilp32 \
	-mcmodel=medany \
	-march=rv32imac_zicsr \
	-Wall \
	-fmessage-length=0 \
	-ffunction-sections \
	-fdata-sections \
	-fno-builtin-printf

# ---------------------------------------------------------
# Debug / release
# ---------------------------------------------------------

ifeq ($(DEBUG), 1)
	CFLAGS += -Og -ggdb3
else
	CFLAGS += -O2
endif

# ---------------------------------------------------------
# Linker flags
# ---------------------------------------------------------

LDFLAGS = \
	-nostartfiles \
	-Tfake_rom.lds \
	-march=rv32imac \
	-mabi=ilp32 \
	-mcmodel=medany \
	-Wl,--gc-sections \
	-Wl,--defsym=__stack_size=300 \
	-Wl,-Map=$(BUILD_DIR)/RTOSDemo.map

# ---------------------------------------------------------
# Source files
# ---------------------------------------------------------

C_SRCS = \
	main.c \
	main_blinky.c \
	riscv-virt.c \
	ns16550.c \
	$(RTOS_SOURCE_DIR)/event_groups.c \
	$(RTOS_SOURCE_DIR)/list.c \
	$(RTOS_SOURCE_DIR)/queue.c \
	$(RTOS_SOURCE_DIR)/stream_buffer.c \
	$(RTOS_SOURCE_DIR)/tasks.c \
	$(RTOS_SOURCE_DIR)/timers.c \
	$(RTOS_SOURCE_DIR)/portable/MemMang/heap_4.c \
	$(RTOS_SOURCE_DIR)/portable/GCC/RISC-V/port.c

ASM_SRCS = \
	start.S \
	vector.S \
	$(RTOS_SOURCE_DIR)/portable/GCC/RISC-V/portASM.S

SRCS = $(C_SRCS) $(ASM_SRCS)

# ---------------------------------------------------------
# Object files
# ---------------------------------------------------------

OBJS = \
	$(C_SRCS:%.c=$(BUILD_DIR)/%.o) \
	$(ASM_SRCS:%.S=$(BUILD_DIR)/%.o)

DEPS = \
	$(C_SRCS:%.c=$(BUILD_DIR)/%.d) \
	$(ASM_SRCS:%.S=$(BUILD_DIR)/%.d)

# ---------------------------------------------------------
# Target
# ---------------------------------------------------------

TARGET = $(BUILD_DIR)/RTOSDemo.elf

# ---------------------------------------------------------
# Build rules
# ---------------------------------------------------------

all: $(TARGET)

$(TARGET): $(OBJS) fake_rom.lds Makefile
	@mkdir -p $(BUILD_DIR)
	$(CC) $(LDFLAGS) $(OBJS) -o $@

# ---------------------------------------------------------
# Compile C
# ---------------------------------------------------------

$(BUILD_DIR)/%.o: %.c
	@mkdir -p $(dir $@)
	$(CC) $(CPPFLAGS) $(CFLAGS) -MMD -MP -c $< -o $@

# ---------------------------------------------------------
# Compile ASM
# ---------------------------------------------------------

$(BUILD_DIR)/%.o: %.S
	@mkdir -p $(dir $@)
	$(CC) $(CPPFLAGS) $(CFLAGS) -MMD -MP -c $< -o $@

# ---------------------------------------------------------
# Run in QEMU
# ---------------------------------------------------------

run: all
	qemu-system-riscv32 \
	        -machine virt \
	        -nographic \
	        -bios none \
	        -kernel $(TARGET)

# ---------------------------------------------------------
# Clean
# ---------------------------------------------------------

clean:
	rm -rf $(BUILD_DIR)

# ---------------------------------------------------------
# Dependencies
# ---------------------------------------------------------

-include $(DEPS)
