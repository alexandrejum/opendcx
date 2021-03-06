# Copyright (c) 2016 DreamWorks Animation LLC. 
#
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
# *       Redistributions of source code must retain the above
#         copyright notice, this list of conditions and the following
#         disclaimer.
# *       Redistributions in binary form must reproduce the above
#         copyright notice, this list of conditions and the following
#         disclaimer in the documentation and/or other materials
#         provided with the distribution.
# *       Neither the name of DreamWorks Animation nor the names of its
#         contributors may be used to endorse or promote products
#         derived from this software without specific prior written
#         permission.
# 
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
# Makefile for OpenDCX examples
#


# Targets:
#   all
#   install             install the example commands into EXAMPLES_DESTDIR
#   clean               delete generated files from the local directory
#   depend              recompute source file header dependencies
#
# Options:
#   shared=no           link executables against static OpenDCX libraries
#                       (default=yes: link against shared libraries)
#   debug=yes           build with debugging symbols and without optimization


#
# The following variables must be defined, either here or on the command line
# (e.g., "make install DESTDIR=/usr/local"):
#

# The directory into which to install files
# (it is recommended that this match the DESTDIR in the OpenDCX
# library makefile)
DESTDIR := /tmp/OpenDCX

# The directory into which to install the example executables
EXAMPLES_DESTDIR := $(DESTDIR)/examples

ifndef OPENDCX_INCL_DIR
    # The parent directory of the OpenDCX/ header directory
    OPENDCX_INCL_DIR := $(DESTDIR)/include
endif
ifndef OPENDCX_LIB_DIR
    # The directory containing libOpenDCX
    OPENDCX_LIB_DIR := $(DESTDIR)/lib
endif


# The version of OpenEXR should match the version of OpenDCX.
# i.e. OpenDCX 2.2 requires OpenEXR 2.2
ifndef OPENEXR_INCL_DIR
    # The parent directory of the OpenEXR header directory
    OPENEXR_INCL_DIR := /usr/include
endif
ifndef OPENEXR_LIB_DIR
    # The directory containing IlmImf & IlmImfUtil libs
    OPENEXR_LIB_DIR := /usr/lib64
endif

# static or shared lib linking
ifeq (no,$(strip $(shared)))
    OPENDCX_LIBS := $(OPENDCX_LIB_DIR)/libOpenDCX.a
    OPENEXR_LIBS := \
        $(OPENEXR_LIB_DIR)/libIlmImfUtil.a \
        $(OPENEXR_LIB_DIR)/libIlmImf.a \
        $(OPENEXR_LIB_DIR)/libImath.a \
        $(OPENEXR_LIB_DIR)/libIlmThread.a \
        $(OPENEXR_LIB_DIR)/libIex.a \
        $(OPENEXR_LIB_DIR)/libHalf.a
else
    OPENDCX_LIBS := -lOpenDCX
    OPENEXR_LIBS := \
        -lIlmImfUtil \
        -lIlmImf \
        -lImath \
        -lIlmThread \
        -lIex \
        -lHalf
endif #shared


#
# Ideally, users shouldn't need to change anything below this line.
#

SHELL = /bin/bash

# Turn off implicit rules for speed
.SUFFIXES:

ifneq (,$(INSTALL_DIR))
    $(warning Warning: $$(INSTALL_DIR) is no longer used; set $$(DESTDIR) instead.)
endif

# Determine the platform.
ifeq ("$(OS)","Windows_NT")
    WINDOWS_NT := 1
else
    UNAME_S := $(shell uname -s)
    ifeq ("$(UNAME_S)","Linux")
        LINUX := 1
    else
        ifeq ("$(UNAME_S)","Darwin")
            MBSD := 1
        endif
    endif
endif

ifeq (yes,$(strip $(debug)))
    OPTIMIZE := -g
else
    OPTIMIZE := -O3 -DNDEBUG
endif

INCLDIRS := \
    -I$(OPENDCX_INCL_DIR) \
    -I$(OPENEXR_INCL_DIR)
#

CXXFLAGS += $(OPTIMIZE) $(INCLDIRS)
#CXXFLAGS += -std=c++0x

# enable unresolved symbol warnings:
LDFLAGS  += -Wl,-z,defs

ifeq (32,$(strip $(bits)))
    CXXFLAGS += -m32 -DBits32_
else
    CXXFLAGS += -m64 -DBits64_
endif

ifdef LINUX
    CXXFLAGS += -DLINUX
    ifeq (32,$(strip $(bits)))
        CXXFLAGS += -DLINUX_32
    else
        CXXFLAGS += -DLINUX_64
    endif
endif
ifdef MBSD
    # Darwin ld treats undefined symbols as errors by default;
    # change to runtime resolution, like Linux.
    CXXFLAGS += -undefined dynamic_lookup
endif

ifeq (no,$(strip $(shared)))
    LIBS += $(OPENDCX_LIBS) $(OPENEXR_LIBS)
else
    LIBS += \
        -Wl,-rpath,$(OPENEXR_LIB_DIR) -L$(OPENEXR_LIB_DIR) $(OPENEXR_LIBS) \
        -Wl,-rpath,$(OPENDCX_LIB_DIR) -L$(OPENDCX_LIB_DIR) $(OPENDCX_LIBS)
endif #shared

LIBS += \
    -ldl -lm -lz

#
ifdef LINUX
    LIBS += -lrt
    LIBS_RPATH += -lrt
endif

# Examples source - each .cpp becomes a command
EXAMPLES_SRC_NAMES := \
    deepTransformExample.cpp \
    raySampleAccumulatorExample.cpp \
    readDeepPixelExample.cpp \
#

# Source code shared between examples:
EXAMPLES_SHARED_SRC_NAMES := \
#

# File to build dependecies for
DEPEND_FILES := \
    $(EXAMPLES_SRC_NAMES) \
    $(EXAMPLES_SHARED_SRC_NAMES) \
#


EXAMPLES_OBJ_NAMES := \
    $(EXAMPLES_SHARED_SRC_NAMES:.cpp=.o) \
    $(EXAMPLES_SRC_NAMES:.cpp=.o)
EXAMPLES_CMD_NAMES := \
    $(EXAMPLES_SRC_NAMES:.cpp=)

DEPEND := dependencies

# Get the list of dependencies that are newer than the current target,
# but limit the list to at most three entries.
list_deps = $(if $(wordlist 4,5,$(?F)),$(firstword $(?F)) and others,$(wordlist 1,3,$(?F)))

ALL_TARGETS := \
    $(DEPEND) \
    $(EXAMPLES_OBJ_NAMES) \
    $(EXAMPLES_CMD_NAMES)
#

.SUFFIXES: .o .cpp

.PHONY: all clean depend install

all: depend $(EXAMPLES_CMD_NAMES)

$(EXAMPLES_OBJ_NAMES): %.o: %.cpp
	@echo "-----------------------------------------------------------------"
	@echo "Compiling $@ because of $(call list_deps)"
	$(CXX) -c $(CXXFLAGS) -fPIC -o $@ $<

$(EXAMPLES_CMD_NAMES): %: %.o $(EXAMPLES_SHARED_SRC_NAMES:.cpp=.o)
	@echo "  Linking $@ because of $(call list_deps)"
	$(CXX) $(LDFLAGS) $(CXXFLAGS) -fPIC -o $@ $^ $(LIBS)

install: $(EXAMPLES_CMD_NAMES)
	@echo "Installing $<"
	if [ ! -d "$(EXAMPLES_DESTDIR)" ]; then \
	    mkdir -p $(EXAMPLES_DESTDIR); \
	    echo "Created $(EXAMPLES_DESTDIR)"; \
	fi
	cp -f $(EXAMPLES_CMD_NAMES) $(EXAMPLES_DESTDIR)
	@echo "Copied $(EXAMPLES_CMD_NAMES) to $(EXAMPLES_DESTDIR)"


# TODO: This accumulates all source file dependencies into a single file
# containing a rule for each *.o file.  Consider generating a separate
# dependency file for each *.o file instead.
$(DEPEND): $(DEPEND_FILES)
	@echo "Generating dependencies because of $(call list_deps)"
	$(RM) $(DEPEND)
	for f in $(DEPEND_FILES); \
		do $(CXX) $(CXXFLAGS) -O0 -MM $$f -MT `echo $$f | sed 's%\.[^.]*%.o%'` >> $(DEPEND); \
	done

depend: $(DEPEND)

clean:
	$(RM) $(ALL_TARGETS) $(DEPEND) $(EXAMPLES_OBJ_NAMES)

ifneq (,$(strip $(wildcard $(DEPEND))))
    include $(DEPEND)
endif

