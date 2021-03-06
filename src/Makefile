#! gmake

#
# Copyright (C) 2006 Laurent Bessard
# 
# This file is part of canfestival, a library implementing the canopen
# stack
# 
# This library is free software; you can redistribute it and/or
# modify it under the terms of the GNU Lesser General Public
# License as published by the Free Software Foundation; either
# version 2.1 of the License, or (at your option) any later version.
# 
# This library is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
# Lesser General Public License for more details.
# 
# You should have received a copy of the GNU Lesser General Public
# License along with this library; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA
# 

# [fj] this file was manually adjusted after CANFestival ./configure
CC = $(CROSS_COMPILE)gcc
PREFIX = $(DESTDIR)/usr/can
BINUTILS_PREFIX = $(CROSS_COMPILE)

#ifneq ($(KERNELRELEASE),)
# Kbuild part of Makefile
obj-m := canfestival.o
canfestival-objs := $(OBJS)

#else
# Normal Makefile
PROG_CFLAGS = -mfloat-abi=soft -mfpu=neon -mtune=cortex-a8 --sysroot=$(SDKTARGETSYSROOT) -fPIC
OS_NAME = Linux
ARCH_NAME = armv7a
TARGET = unix
CAN_DRIVER = can_socket
TIMERS_DRIVER = timers_unix
ENABLE_LSS = 0

INCLUDES = -I../include -I../include/$(TARGET) -I../include/$(TIMERS_DRIVER) -I../drivers/$(TARGET) -I$(SDKTARGETSYSROOT)/usr/include

OBJS = $(TARGET)_objacces.o $(TARGET)_lifegrd.o $(TARGET)_sdo.o\
	    $(TARGET)_pdo.o $(TARGET)_sync.o $(TARGET)_nmtSlave.o $(TARGET)_nmtMaster.o \
	    $(TARGET)_states.o $(TARGET)_timer.o $(TARGET)_dcf.o $(TARGET)_emcy.o \
	    ../drivers/unix/unix.o ../drivers/timers_unix/timers_unix.o \
	    ../drivers/can_socket/can_socket.o 


ifeq ($(ENABLE_LSS),1)
OBJS += $(TARGET)_lss.o
endif

# # # # Target specific paramters # # # #

ifeq ($(TARGET),hcs12)
OPT_CFLAGS = -Os
PROGDEFINES = -mnoshort -Wall -Wmissing-prototypes -fno-strict-aliasing
endif

ifeq ($(TARGET),unix)
OPT_CFLAGS = -g3 -gdwarf-2
#-O2
endif

# # # # Options # # # # 

all: canfestival

ifeq ($(TIMERS_DRIVER), timers_kernel)
OBJS := $(shell echo $(OBJS) | sed "s:$(TARGET)_::g")
OBJS += symbols.o
OBJS += ../drivers/unix/libcanfestival_$(TARGET).o
EXTRA_CFLAGS := $(shell echo $(INCLUDES) | sed "s:-I:-I$(src)/:g")
EXTRA_CFLAGS += $(PROG_CFLAGS)
KERNELDIR := $(HOME)/duagon/linux/linux-current/ti-sdk-am335x-evm-05.04.01.00/board-support/linux-3.2-psp04.06.00.07.sdk
export OBJS
export EXTRA_CFLAGS

canfestival:
	@echo " "
	@echo "*********************************************"
	@echo "**Building [libcanfestival.o]"
	@echo "*********************************************"
	$(MAKE) -C $(KERNELDIR) M=$(shell pwd) modules

clean:
	$(MAKE) -C $(KERNELDIR) M=$(shell pwd) clean
	rm -f Module.symvers

# [fj] as it is cross-compiled, no need to install these
#install:
#	$(MAKE) -C $(KERNELDIR) M=$(shell pwd) modules_install
#	mkdir -p $(PREFIX)/include/canfestival
#	cp ../include/*.h $(PREFIX)/include/canfestival
#
#uninstall:
#	rm -rf $(PREFIX)/include/canfestival

else
CFLAGS = $(OPT_CFLAGS)

canfestival: libcanfestival.a

libcanfestival.a: $(OBJS)
	@echo " "
	@echo "*********************************************"
	@echo "**Building [libcanfestival.a]"
	@echo "*********************************************"
	$(BINUTILS_PREFIX)ar rc $@ $(OBJS)
	$(BINUTILS_PREFIX)ranlib $@

$(TARGET)_%o: %c
	@echo " "
	@echo "*********************************************"
	@echo "**Compiling $< -> $@"
	@echo "*********************************************"
	$(CC) $(CFLAGS) $(PROG_CFLAGS) ${PROGDEFINES} $(INCLUDES) -o $@ -c $<

# [fj] as it is cross-compiled, no need to install these
#install: libcanfestival.a
#	mkdir -p $(PREFIX)/lib/
#	mkdir -p $(PREFIX)/include/canfestival
#	cp libcanfestival.a $(PREFIX)/lib/
#	cp ../include/*.h $(PREFIX)/include/canfestival
#
#uninstall:
#	rm -f $(PREFIX)/lib/libcanfestival.a
#	rm -rf $(PREFIX)/include/canfestival

clean:
	rm -f $(OBJS) libcanfestival.a

endif
mrproper: clean

#endif
