#
# Copyright 2010 Clozure Associates
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

SVN_REVISION := "$(shell svnversion || echo unknown)"

# See <http://people.debian.org/~wookey/bootstrap.html>
#CROSS=aarch64-linux-gnu-
VPATH = ../
RM = /bin/rm
AS = ${CROSS}as
CC = ${CROSS}gcc
M4 = m4
ASFLAGS = 
M4FLAGS = -DLINUX -DARM64
CDEFINES = -DLINUX -DARM64 -D_REENTRANT -D_GNU_SOURCE -DUSE_FUTEX -DSVN_REVISION=$(SVN_REVISION)
CDEBUG = -g
COPT = -O2
# Once in a while, -Wformat says something useful.  The odds are against that,
# however.
WFORMAT = -Wno-format
PLATFORM_H = platform-linuxarm64.h


# If the linker supports a "--hash-style=" option, use traditional
# Sysv hash tables.  (If it doesn't support that option, assume
# that traditional hash tables will be used by default.)
ld_has_hash_style = $(shell $(LD) --help | grep "hash-style=")
ifeq ($(ld_has_hash_style),)
HASH_STYLE=
else
HASH_STYLE="-Wl,--hash-style=sysv"
endif



.s.o:
	$(M4) $(M4FLAGS) -I../ $< | $(AS)  $(ASFLAGS) -o $@
.c.o:
	$(CC) -include ../$(PLATFORM_H) -c $< $(CDEFINES) $(CDEBUG) $(COPT)   $(WFORMAT)  -o $@

SPOBJ = pad.o  arm64-spentry.o
ASMOBJ = arm64-asmutils.o imports.o

COBJ  = pmcl-kernel.o gc-common.o arm64-gc.o bits.o  arm64-exceptions.o \
	image.o thread_manager.o lisp-debug.o memory.o unix-calls.o

DEBUGOBJ = lispdcmd.o plprint.o plsym.o albt.o arm_print.o
KERNELOBJ= $(COBJ) arm64-asmutils.o  imports.o

SPINC =	lisp.s m4macros.m4 arm64-constants.s arm64-macros.s errors.s arm64-uuo.s \
	lisp_globals.s

CHEADERS = area.h bits.h arm64-constants.h lisp-errors.h gc.h lisp.h \
	lisp-exceptions.h lisp_globals.h macros.h memprotect.h image.h \
	threads.h arm64-exceptions.h $(PLATFORM_H)

# Subprims linked into the kernel ?
# Yes:

KSPOBJ = $(SPOBJ)
all:	../../armcl64


# No:

# KSPOBJ=
# all:	../../armcl64 ../../subprims.so

OSLIBS = -ldl -lm -lpthread -lrt


../../armcl64:	$(KSPOBJ) $(KERNELOBJ) $(DEBUGOBJ)
	$(CC)  $(CDEBUG)  -Wl,--export-dynamic $(HASH_STYLE) -mfloat-abi=$(FLOAT_ABI) -o $@ -T ./armlinux.x $(KSPOBJ) $(KERNELOBJ) $(DEBUGOBJ) -Wl,--no-as-needed $(OSLIBS)


$(SPOBJ): $(SPINC)
$(ASMOBJ): $(SPINC)
$(COBJ): $(CHEADERS)
$(DEBUGOBJ): $(CHEADERS) lispdcmd.h


cclean:
	$(RM) -f $(KERNELOBJ) $(DEBUGOBJ) ../../armcl

clean:	cclean
	$(RM) -f $(SPOBJ)

strip:	../../armcl
	strip -g ../../armcl
