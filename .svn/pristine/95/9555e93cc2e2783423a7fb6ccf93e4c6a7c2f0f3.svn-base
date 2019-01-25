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

#all: canfestival install
# [fj] here only builds (in development computer); install is a separate step
all: canfestival

canfestival: driver
	$(MAKE) -C src $@

driver:
	$(MAKE) -C drivers $@

#install: canfestival driver
# [fj] here only installs (in target device); build is a separate step
install:
# [fj] will only install libcanfestival_can_socket.so to targetNFS/usr/can
	$(MAKE) -C drivers $@
#	$(MAKE) -C src $@
#	ldconfig

uninstall:
	$(MAKE) -C drivers $@
#	$(MAKE) -C src $@

clean:
	$(MAKE) -C src $@
	$(MAKE) -C drivers $@
	
mrproper: clean
	$(MAKE) -C src $@
	$(MAKE) -C drivers $@

