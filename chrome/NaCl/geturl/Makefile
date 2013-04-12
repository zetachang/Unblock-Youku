# Copyright (c) 2012 The Native Client Authors. All rights reserved.
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.

#
# GNU Make based build file.  For details on GNU Make see:
#   http://www.gnu.org/software/make/manual/make.html
#

#
# Get pepper directory for toolchain and includes.
#
# If NACL_SDK_ROOT is not set, then assume it can be found a two directories up,
# from the default example directory location.
#
THIS_MAKEFILE:=$(abspath $(lastword $(MAKEFILE_LIST)))
NACL_SDK_ROOT?=$(abspath $(dir $(THIS_MAKEFILE))../..)
CHROME_PATH?=Undefined

#
# Defaults
#
NACL_WARNINGS:=-Wno-long-long -Wall -Wswitch-enum -Werror -pedantic

#
# Project Settings
#
VALID_TOOLCHAINS:=newlib glibc pnacl
TOOLCHAIN?=newlib



#
# Default target
#
all:

#
# Alias for standard commands
#
CP:=python $(NACL_SDK_ROOT)/tools/oshelpers.py cp
MKDIR:=python $(NACL_SDK_ROOT)/tools/oshelpers.py mkdir
MV:=python $(NACL_SDK_ROOT)/tools/oshelpers.py mv
RM:=python $(NACL_SDK_ROOT)/tools/oshelpers.py rm


#
# Verify we selected a valid toolchain for this example
#
ifeq (,$(findstring $(TOOLCHAIN),$(VALID_TOOLCHAINS)))
$(warning Availbile choices are: $(VALID_TOOLCHAINS))
$(error Can not use TOOLCHAIN=$(TOOLCHAIN) on this example.)
endif


#
# Compute path to requested NaCl Toolchain
#
OSNAME:=$(shell python $(NACL_SDK_ROOT)/tools/getos.py)
TC_PATH:=$(abspath $(NACL_SDK_ROOT)/toolchain)


#
# Verify we have a valid NACL_SDK_ROOT by looking for the toolchain directory
#
ifeq (,$(wildcard $(TC_PATH)))
$(warning No valid NACL_SDK_ROOT at $(NACL_SDK_ROOT))
ifeq ($(origin NACL_SDK_ROOT), 'file')
$(error Override the default value via enviornment variable, or command-line.)
else
$(error Fix the NACL_SDK_ROOT specified in the environment or command-line.)
endif
endif


#
# Disable DOS PATH warning when using Cygwin based NaCl tools on Windows
#
CYGWIN ?= nodosfilewarning
export CYGWIN


#
# Defaults for TOOLS
#

NEWLIB_CC?=$(TC_PATH)/$(OSNAME)_x86_newlib/bin/i686-nacl-gcc -c
NEWLIB_CXX?=$(TC_PATH)/$(OSNAME)_x86_newlib/bin/i686-nacl-g++ -c
NEWLIB_LINK?=$(TC_PATH)/$(OSNAME)_x86_newlib/bin/i686-nacl-g++ -Wl,-as-needed
NEWLIB_LIB?=$(TC_PATH)/$(OSNAME)_x86_newlib/bin/i686-nacl-ar r
NEWLIB_DUMP?=$(TC_PATH)/$(OSNAME)_x86_newlib/x86_64-nacl/bin/objdump
NEWLIB_CCFLAGS?=-MMD -pthread $(NACL_WARNINGS) -idirafter $(NACL_SDK_ROOT)/include
NEWLIB_LDFLAGS?=-pthread

GLIBC_CC?=$(TC_PATH)/$(OSNAME)_x86_glibc/bin/i686-nacl-gcc -c
GLIBC_CXX?=$(TC_PATH)/$(OSNAME)_x86_glibc/bin/i686-nacl-g++ -c
GLIBC_LINK?=$(TC_PATH)/$(OSNAME)_x86_glibc/bin/i686-nacl-g++ -Wl,-as-needed
GLIBC_LIB?=$(TC_PATH)/$(OSNAME)_x86_glibc/bin/i686-nacl-ar r
GLIBC_DUMP?=$(TC_PATH)/$(OSNAME)_x86_glibc/x86_64-nacl/bin/objdump
GLIBC_PATHS:=-L $(TC_PATH)/$(OSNAME)_x86_glibc/x86_64-nacl/lib32
GLIBC_PATHS+=-L $(TC_PATH)/$(OSNAME)_x86_glibc/x86_64-nacl/lib
GLIBC_CCFLAGS?=-MMD -pthread $(NACL_WARNINGS) -idirafter $(NACL_SDK_ROOT)/include
GLIBC_LDFLAGS?=-pthread

PNACL_CC?=$(TC_PATH)/$(OSNAME)_x86_pnacl/newlib/bin/pnacl-clang -c
PNACL_CXX?=$(TC_PATH)/$(OSNAME)_x86_pnacl/newlib/bin/pnacl-clang++ -c
PNACL_LINK?=$(TC_PATH)/$(OSNAME)_x86_pnacl/newlib/bin/pnacl-clang++
PNACL_LIB?=$(TC_PATH)/$(OSNAME)_x86_pnacl/newlib/bin/pnacl-ar r
PNACL_DUMP?=$(TC_PATH)/$(OSNAME)_x86_pnacl/newlib/bin/objdump
PNACL_CCFLAGS?=-MMD -pthread $(NACL_WARNINGS) -idirafter $(NACL_SDK_ROOT)/include
PNACL_LDFLAGS?=-pthread
TRANSLATE:=$(TC_PATH)/$(OSNAME)_x86_pnacl/newlib/bin/pnacl-translate



#
# NMF Manifiest generation
#
# Use the python script create_nmf to scan the binaries for dependencies using
# objdump.  Pass in the (-L) paths to the default library toolchains so that we
# can find those libraries and have it automatically copy the files (-s) to
# the target directory for us.
NMF:=python $(NACL_SDK_ROOT)/tools/create_nmf.py


#
# Verify we can find the Chrome executable if we need to launch it.
#
.PHONY: CHECK_FOR_CHROME
CHECK_FOR_CHROME:
ifeq (,$(wildcard $(CHROME_PATH)))
	$(warning No valid Chrome found at CHROME_PATH=$(CHROME_PATH))
	$(error Set CHROME_PATH via an environment variable, or command-line.)
else
	$(warning Using chrome at: $(CHROME_PATH))
endif

#
# Per target object lists
#
GETURL_OBJS:=geturl geturl_handler

#
# Rules for newlib toolchain
#
newlib:
	$(MKDIR) newlib
newlib/Debug: | newlib
	$(MKDIR) newlib/Debug
newlib/Release: | newlib
	$(MKDIR) newlib/Release

# Include header dependency files.
-include newlib/Debug/*.d
-include newlib/Release/*.d

PPAPI_DEBUG:=$(abspath newlib/Debug/geturl_<ARCH>.nexe);application/x-ppapi-debug
newlib/Debug/geturl_x86_32.o : geturl.cc $(THIS_MAKE) | newlib/Debug
	$(NEWLIB_CXX) -o $@ $< -g -O0 -m32 $(NEWLIB_CCFLAGS) $(GETURL_CXXFLAGS)  

newlib/Debug/geturl_handler_x86_32.o : geturl_handler.cc $(THIS_MAKE) | newlib/Debug
	$(NEWLIB_CXX) -o $@ $< -g -O0 -m32 $(NEWLIB_CCFLAGS) $(GETURL_CXXFLAGS)  


GETURL_NEWLIB_DEBUG_x86_32_O:=$(patsubst %,newlib/Debug/%_x86_32.o,$(GETURL_OBJS))
newlib/Debug/geturl_x86_32.nexe : $(GETURL_NEWLIB_DEBUG_x86_32_O)
	$(NEWLIB_LINK) -o $@ $^ -g -m32 $(NEWLIB_LDFLAGS) $(GETURL_LDFLAGS) -L$(NACL_SDK_ROOT)/lib/$(OSNAME)_x86_32_newlib/Debug -Wl,--start-group -lppapi_cpp -lppapi -lpthread -Wl,--end-group

newlib/Debug/geturl_x86_64.o : geturl.cc $(THIS_MAKE) | newlib/Debug
	$(NEWLIB_CXX) -o $@ $< -g -O0 -m64 $(NEWLIB_CCFLAGS) $(GETURL_CXXFLAGS)  

newlib/Debug/geturl_handler_x86_64.o : geturl_handler.cc $(THIS_MAKE) | newlib/Debug
	$(NEWLIB_CXX) -o $@ $< -g -O0 -m64 $(NEWLIB_CCFLAGS) $(GETURL_CXXFLAGS)  


GETURL_NEWLIB_DEBUG_x86_64_O:=$(patsubst %,newlib/Debug/%_x86_64.o,$(GETURL_OBJS))
newlib/Debug/geturl_x86_64.nexe : $(GETURL_NEWLIB_DEBUG_x86_64_O)
	$(NEWLIB_LINK) -o $@ $^ -g -m64 $(NEWLIB_LDFLAGS) $(GETURL_LDFLAGS) -L$(NACL_SDK_ROOT)/lib/$(OSNAME)_x86_64_newlib/Debug -Wl,--start-group -lppapi_cpp -lppapi -lpthread -Wl,--end-group


ALL_TARGETS+=newlib/Debug/geturl.nmf
newlib/Debug/geturl.nmf : newlib/Debug/geturl_x86_32.nexe newlib/Debug/geturl_x86_64.nexe
	$(NMF) -D $(NEWLIB_DUMP) -o $@ $^ -t newlib -s newlib/Debug

PPAPI_RELEASE:=$(abspath newlib/Release/geturl_x86_64.nexe);application/x-ppapi-release
newlib/Release/geturl_x86_32.o : geturl.cc $(THIS_MAKE) | newlib/Release
	$(NEWLIB_CXX) -o $@ $< -O2 -m32 $(NEWLIB_CCFLAGS) $(GETURL_CXXFLAGS)  

newlib/Release/geturl_handler_x86_32.o : geturl_handler.cc $(THIS_MAKE) | newlib/Release
	$(NEWLIB_CXX) -o $@ $< -O2 -m32 $(NEWLIB_CCFLAGS) $(GETURL_CXXFLAGS)  


GETURL_NEWLIB_RELEASE_x86_32_O:=$(patsubst %,newlib/Release/%_x86_32.o,$(GETURL_OBJS))
newlib/Release/geturl_x86_32.nexe : $(GETURL_NEWLIB_RELEASE_x86_32_O)
	$(NEWLIB_LINK) -o $@ $^ -m32 $(NEWLIB_LDFLAGS) $(GETURL_LDFLAGS) -L$(NACL_SDK_ROOT)/lib/$(OSNAME)_x86_32_newlib/Release  -Wl,--start-group -lppapi_cpp -lppapi -lpthread -Wl,--end-group

newlib/Release/geturl_x86_64.o : geturl.cc $(THIS_MAKE) | newlib/Release
	$(NEWLIB_CXX) -o $@ $< -O2 -m64 $(NEWLIB_CCFLAGS) $(GETURL_CXXFLAGS)  

newlib/Release/geturl_handler_x86_64.o : geturl_handler.cc $(THIS_MAKE) | newlib/Release
	$(NEWLIB_CXX) -o $@ $< -O2 -m64 $(NEWLIB_CCFLAGS) $(GETURL_CXXFLAGS)  


GETURL_NEWLIB_RELEASE_x86_64_O:=$(patsubst %,newlib/Release/%_x86_64.o,$(GETURL_OBJS))
newlib/Release/geturl_x86_64.nexe : $(GETURL_NEWLIB_RELEASE_x86_64_O)
	$(NEWLIB_LINK) -o $@ $^ -m64 $(NEWLIB_LDFLAGS) $(GETURL_LDFLAGS) -L$(NACL_SDK_ROOT)/lib/$(OSNAME)_x86_64_newlib/Release  -Wl,--start-group -lppapi_cpp -lppapi -lpthread -Wl,--end-group


ALL_TARGETS+=newlib/Release/geturl.nmf
newlib/Release/geturl.nmf : newlib/Release/geturl_x86_32.nexe newlib/Release/geturl_x86_64.nexe
	$(NMF) -D $(NEWLIB_DUMP) -o $@ $^ -t newlib -s newlib/Release


#
# Rules for glibc toolchain
#
glibc:
	$(MKDIR) glibc
glibc/Debug: | glibc
	$(MKDIR) glibc/Debug
glibc/Release: | glibc
	$(MKDIR) glibc/Release

# Include header dependency files.
-include glibc/Debug/*.d
-include glibc/Release/*.d

PPAPI_DEBUG:=$(abspath glibc/Debug/geturl_<ARCH>.nexe);application/x-ppapi-debug
glibc/Debug/geturl_x86_32.o : geturl.cc $(THIS_MAKE) | glibc/Debug
	$(GLIBC_CXX) -o $@ $< -g -O0 -m32 $(GLIBC_CCFLAGS) $(GETURL_CXXFLAGS)  

glibc/Debug/geturl_handler_x86_32.o : geturl_handler.cc $(THIS_MAKE) | glibc/Debug
	$(GLIBC_CXX) -o $@ $< -g -O0 -m32 $(GLIBC_CCFLAGS) $(GETURL_CXXFLAGS)  


GETURL_GLIBC_DEBUG_x86_32_O:=$(patsubst %,glibc/Debug/%_x86_32.o,$(GETURL_OBJS))
glibc/Debug/geturl_x86_32.nexe : $(GETURL_GLIBC_DEBUG_x86_32_O)
	$(GLIBC_LINK) -o $@ $^ -g -m32 $(GLIBC_LDFLAGS) $(GETURL_LDFLAGS) -L$(NACL_SDK_ROOT)/lib/$(OSNAME)_x86_32_glibc/Debug -Wl,--start-group -lppapi_cpp -lppapi -lpthread -Wl,--end-group

glibc/Debug/geturl_x86_64.o : geturl.cc $(THIS_MAKE) | glibc/Debug
	$(GLIBC_CXX) -o $@ $< -g -O0 -m64 $(GLIBC_CCFLAGS) $(GETURL_CXXFLAGS)  

glibc/Debug/geturl_handler_x86_64.o : geturl_handler.cc $(THIS_MAKE) | glibc/Debug
	$(GLIBC_CXX) -o $@ $< -g -O0 -m64 $(GLIBC_CCFLAGS) $(GETURL_CXXFLAGS)  


GETURL_GLIBC_DEBUG_x86_64_O:=$(patsubst %,glibc/Debug/%_x86_64.o,$(GETURL_OBJS))
glibc/Debug/geturl_x86_64.nexe : $(GETURL_GLIBC_DEBUG_x86_64_O)
	$(GLIBC_LINK) -o $@ $^ -g -m64 $(GLIBC_LDFLAGS) $(GETURL_LDFLAGS) -L$(NACL_SDK_ROOT)/lib/$(OSNAME)_x86_64_glibc/Debug -Wl,--start-group -lppapi_cpp -lppapi -lpthread -Wl,--end-group


ALL_TARGETS+=glibc/Debug/geturl.nmf
glibc/Debug/geturl.nmf : glibc/Debug/geturl_x86_32.nexe glibc/Debug/geturl_x86_64.nexe
	$(NMF) -D $(GLIBC_DUMP) -o $@ $(GLIBC_PATHS) $^ -t glibc -s glibc/Debug $(GLIBC_REMAP)

PPAPI_RELEASE:=$(abspath glibc/Release/geturl_x86_64.nexe);application/x-ppapi-release
glibc/Release/geturl_x86_32.o : geturl.cc $(THIS_MAKE) | glibc/Release
	$(GLIBC_CXX) -o $@ $< -O2 -m32 $(GLIBC_CCFLAGS) $(GETURL_CXXFLAGS)  

glibc/Release/geturl_handler_x86_32.o : geturl_handler.cc $(THIS_MAKE) | glibc/Release
	$(GLIBC_CXX) -o $@ $< -O2 -m32 $(GLIBC_CCFLAGS) $(GETURL_CXXFLAGS)  


GETURL_GLIBC_RELEASE_x86_32_O:=$(patsubst %,glibc/Release/%_x86_32.o,$(GETURL_OBJS))
glibc/Release/geturl_x86_32.nexe : $(GETURL_GLIBC_RELEASE_x86_32_O)
	$(GLIBC_LINK) -o $@ $^ -m32 $(GLIBC_LDFLAGS) $(GETURL_LDFLAGS) -L$(NACL_SDK_ROOT)/lib/$(OSNAME)_x86_32_glibc/Release  -Wl,--start-group -lppapi_cpp -lppapi -lpthread -Wl,--end-group

glibc/Release/geturl_x86_64.o : geturl.cc $(THIS_MAKE) | glibc/Release
	$(GLIBC_CXX) -o $@ $< -O2 -m64 $(GLIBC_CCFLAGS) $(GETURL_CXXFLAGS)  

glibc/Release/geturl_handler_x86_64.o : geturl_handler.cc $(THIS_MAKE) | glibc/Release
	$(GLIBC_CXX) -o $@ $< -O2 -m64 $(GLIBC_CCFLAGS) $(GETURL_CXXFLAGS)  


GETURL_GLIBC_RELEASE_x86_64_O:=$(patsubst %,glibc/Release/%_x86_64.o,$(GETURL_OBJS))
glibc/Release/geturl_x86_64.nexe : $(GETURL_GLIBC_RELEASE_x86_64_O)
	$(GLIBC_LINK) -o $@ $^ -m64 $(GLIBC_LDFLAGS) $(GETURL_LDFLAGS) -L$(NACL_SDK_ROOT)/lib/$(OSNAME)_x86_64_glibc/Release  -Wl,--start-group -lppapi_cpp -lppapi -lpthread -Wl,--end-group


ALL_TARGETS+=glibc/Release/geturl.nmf
glibc/Release/geturl.nmf : glibc/Release/geturl_x86_32.nexe glibc/Release/geturl_x86_64.nexe
	$(NMF) -D $(GLIBC_DUMP) -o $@ $(GLIBC_PATHS) $^ -t glibc -s glibc/Release $(GLIBC_REMAP)


#
# Rules for pnacl toolchain
#
pnacl:
	$(MKDIR) pnacl
pnacl/Debug: | pnacl
	$(MKDIR) pnacl/Debug
pnacl/Release: | pnacl
	$(MKDIR) pnacl/Release

# Include header dependency files.
-include pnacl/Debug/*.d
-include pnacl/Release/*.d

PPAPI_DEBUG:=$(abspath pnacl/Debug/geturl.pexe);application/x-ppapi-debug
pnacl/Debug/geturl_PNACL.o : geturl.cc $(THIS_MAKE) | pnacl/Debug
	$(PNACL_CXX) -o $@ $< -g -O0  $(PNACL_CCFLAGS) $(GETURL_CXXFLAGS)  

pnacl/Debug/geturl_handler_PNACL.o : geturl_handler.cc $(THIS_MAKE) | pnacl/Debug
	$(PNACL_CXX) -o $@ $< -g -O0  $(PNACL_CCFLAGS) $(GETURL_CXXFLAGS)  


GETURL_PNACL_DEBUG_PNACL_O:=$(patsubst %,pnacl/Debug/%_PNACL.o,$(GETURL_OBJS))
pnacl/Debug/geturl.pexe : $(GETURL_PNACL_DEBUG_PNACL_O)
	$(PNACL_LINK) -o $@ $^ -g $(PNACL_LDFLAGS) $(GETURL_LDFLAGS) -L$(NACL_SDK_ROOT)/lib/$(OSNAME)_PNACL_pnacl/Debug -lppapi_cpp -lppapi -lpthread

pnacl/Debug/geturl_x86_32.nexe : pnacl/Debug/geturl.pexe
	$(TRANSLATE) -arch x86-32 $< -o $@

pnacl/Debug/geturl_x86_64.nexe : pnacl/Debug/geturl.pexe
	$(TRANSLATE) -arch x86-64 $< -o $@

pnacl/Debug/geturl_arm.nexe : pnacl/Debug/geturl.pexe
	$(TRANSLATE) -arch arm $< -o $@


ALL_TARGETS+=pnacl/Debug/geturl.nmf
pnacl/Debug/geturl.nmf : pnacl/Debug/geturl_x86_32.nexe pnacl/Debug/geturl_x86_64.nexe pnacl/Debug/geturl_arm.nexe
	$(NMF) -D $(PNACL_DUMP) -o $@ $^ -t pnacl -s pnacl/Debug

PPAPI_RELEASE:=$(abspath pnacl/Release/geturl.pexe);application/x-ppapi-release
pnacl/Release/geturl_PNACL.o : geturl.cc $(THIS_MAKE) | pnacl/Release
	$(PNACL_CXX) -o $@ $< -O2  $(PNACL_CCFLAGS) $(GETURL_CXXFLAGS)  

pnacl/Release/geturl_handler_PNACL.o : geturl_handler.cc $(THIS_MAKE) | pnacl/Release
	$(PNACL_CXX) -o $@ $< -O2  $(PNACL_CCFLAGS) $(GETURL_CXXFLAGS)  


GETURL_PNACL_RELEASE_PNACL_O:=$(patsubst %,pnacl/Release/%_PNACL.o,$(GETURL_OBJS))
pnacl/Release/geturl.pexe : $(GETURL_PNACL_RELEASE_PNACL_O)
	$(PNACL_LINK) -o $@ $^ $(PNACL_LDFLAGS) $(GETURL_LDFLAGS) -L$(NACL_SDK_ROOT)/lib/$(OSNAME)_PNACL_pnacl/Release -lppapi_cpp -lppapi -lpthread

pnacl/Release/geturl_x86_32.nexe : pnacl/Release/geturl.pexe
	$(TRANSLATE) -arch x86-32 $< -o $@

pnacl/Release/geturl_x86_64.nexe : pnacl/Release/geturl.pexe
	$(TRANSLATE) -arch x86-64 $< -o $@

pnacl/Release/geturl_arm.nexe : pnacl/Release/geturl.pexe
	$(TRANSLATE) -arch arm $< -o $@


ALL_TARGETS+=pnacl/Release/geturl.nmf
pnacl/Release/geturl.nmf : pnacl/Release/geturl_x86_32.nexe pnacl/Release/geturl_x86_64.nexe pnacl/Release/geturl_arm.nexe
	$(NMF) -D $(PNACL_DUMP) -o $@ $^ -t pnacl -s pnacl/Release

#
# Target to remove temporary files
#
.PHONY: clean
clean:
	$(RM) -fr newlib/Debug
	$(RM) -fr newlib/Release
	$(RM) -fr glibc/Debug
	$(RM) -fr glibc/Release
	$(RM) -fr pnacl/Debug
	$(RM) -fr pnacl/Release


all: $(ALL_TARGETS)




RUN: all
	python ../httpd.py

CONFIG?=Debug
PAGE?=index_$(TOOLCHAIN)_$(CONFIG).html

LAUNCH: CHECK_FOR_CHROME all
ifeq (,$(wildcard $(PAGE)))
	$(warning No valid HTML page found at $(PAGE))
	$(error Make sure TOOLCHAIN and CONFIG are properly set)
endif
	$(CHROME_PATH) $(NEXE_ARGS) --register-pepper-plugins="$(PPAPI_DEBUG),$(PPAPI_RELEASE)" localhost:5103/$(PAGE)


