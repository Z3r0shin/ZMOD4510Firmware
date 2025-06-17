# Makefile to build the no2_o3 example code
#
# Supported targets:
#  Windows:       Example application uses the ComBoard HAL
#  Raspberry Pi:  Example application uses the PiGPIO HAL
#
# Default build configuration:
#  Windows:      build for 64 bit OS - use flag ARCH=32 to build 32 bit
#  Raspberry Pi: required build configuration is selected automatically
#
# Windows alternative compilers:
#  Recommended compile environment on windows is w64devkit, which 
#  which provides gcc binary (but only works for 64 bit builds).
#
#  If an alternative compile environment is used, specify the the
#  compiler name, e.g. mingww32-gcc by to the makefile with flag
#  GCC=mingw32-gcc
#
#  Alternative compile environments must ensure, that the rm command
#  is available. Mingw32 provides msys for that which must be avaialble
#  in the windows search path, e.g. by setting it on command line:
#  set PATH=%PATH%;<PATH-TO-MSYS>/bin


TARGET_NAME := no2_o3-example

SOURCES := example.c               \
           sensors/zmod4xxx.c      \
           sensors/hs4xxx.c        \
           sensors/hs3xxx.c        \
           sensors/hsxxxx.c        \
           hal/zmod4xxx_hal.c      \
           hal/hal.c

LINKLIBS := _no2_o3 \
            _zmod4xxx_cleaning \

# include operating system specific files
ifeq ($(OS),Windows_NT)
  ifeq ($(ARCH),32)
    ARCHBITS:=32
    ARCHTYPE:=x86
  else
    ARCHBITS:=64
    ARCHTYPE:=x64
  endif

  SOURCES += hal/comboard/comboard.c \
             hal/comboard/escom.c  \
             hal/comboard/hicom.c
  
  TARGET := $(TARGET_NAME).exe

  SHARED_LIBS := hal/3rdParty/libusb/mingw$(ARCHBITS)/libusb-1.0.dll \
                 hal/3rdParty/ftdi/mingw$(ARCHBITS)/libmpsse.dll

  SHARED_LIB_FILES := $(notdir $(SHARED_LIBS))
  SHARED_LIB_DIRS  := $(dir    $(SHARED_LIBS))

  LDFLAGS := -L"../lib/Windows/$(ARCHTYPE)/mingw$(ARCHBITS)" \
             $(SHARED_LIB_DIRS:%=-L%)

  LINKLIBS += $(SHARED_LIB_FILES:lib%.dll=%)
  
  EXTRA_TARGETS := copyshared
else
  UNAME_S := $(shell uname -s)
  ifeq ($(UNAME_S),Linux)
		UNAME_M := $(shell uname -m)
    ifeq ($(UNAME_M),aarch64)
			ARCHBITS:=64
      LDFLAGS := -L"../lib/Raspberry Pi/ARMv8-A x64/aarch64-linux-gnu-gcc"
    else
      ARCHBITS:=32
      LDFLAGS := -L"../lib/Raspberry Pi/ARMv8-A x86/arm-linux-gnueabihf-gcc"
    endif
    LINKLIBS += pigpio m

    SOURCES += hal/raspi/rpi.c

    TARGET := $(TARGET_NAME)
  endif
endif


ifeq ($(GCC),)
  GCC := gcc
endif


BUILDDIR ?= build-$(ARCHBITS)

OBJECTS   := $(SOURCES:%.c=$(BUILDDIR)/%.o)
BUILDTREE := $(sort $(dir $(OBJECTS) $(DEFPFILES)))

CFLAGS = -g

$(BUILDDIR)/%.o : %.c
	$(GCC) -c $(CFLAGS) $(EXTRA_C_FLAGS) -I. -Ialgos -Isensors -Ihal -o $@ $<

all: clean $(BUILDTREE) $(BUILDDIR)/$(TARGET) $(EXTRA_TARGETS)

clean:
	rm -rf $(BUILDDIR)

# create build subdirectories
$(BUILDTREE):
	mkdir -p $@


$(BUILDDIR)/$(TARGET): $(OBJECTS)
	$(GCC) -o $@ $(LDFLAGS) $^ $(LINKLIBS:%=-l%)

.PHONY:
copyshared:
	cp $(SHARED_LIBS) $(BUILDDIR)
