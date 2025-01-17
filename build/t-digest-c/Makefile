
ifeq (n,$(findstring n,$(firstword -$(MAKEFLAGS))))
DRY_RUN:=1
else
DRY_RUN:=
endif

#----------------------------------------------------------------------------------------------

ROOT=../..

ifeq ($(wildcard $(ROOT)/deps/readies/*),)
___:=$(shell git submodule update --init --recursive &> /dev/null)
endif

MK.pyver:=3
include $(ROOT)/deps/readies/mk/main

#----------------------------------------------------------------------------------------------

define HELPTEXT
make build          # compile and link
  DEBUG=1             # build for debugging
  WHY=1               # explain CMake decisions (in /tmp/cmake-why)
  FORCE=1             # Force CMake rerun (default)
  CMAKE_ARGS=...      # extra arguments to CMake
  SLOW=1              # do not parallelize build (for diagnostics)
make clean         # remove build artifacts
  ALL=1              # remove entire artifacts directory

endef

#----------------------------------------------------------------------------------------------

BINDIR=$(BINROOT)/t-digest-c

TARGET=$(BINDIR)/libtdigest_static.a

#----------------------------------------------------------------------------------------------

CMAKE_DIR=$(ROOT)/deps/t-digest-c

ifeq ($(DEBUG),1)
CMAKE_BUILD_TYPE=DEBUG
else
CMAKE_BUILD_TYPE=RelWithDebInfo
endif
CMAKE_DEBUG=-DCMAKE_BUILD_TYPE=$(CMAKE_BUILD_TYPE)

ifeq ($(WHY),1)
CMAKE_WHY=--trace-expand > /tmp/cmake-why 2>&1
endif

CMAKE_FLAGS=-Wno-dev

#ifneq ($(COV),1)
CMAKE_FLAGS+=\
	-DBUILD_SHARED=OFF \
	-DBUILD_STATIC=ON \
	-DENABLE_CODECOVERAGE=OFF \
	-DBUILD_TESTS=OFF \
	-DBUILD_BENCHMARK=OFF \
	-DBUILD_EXAMPLES=OFF
#else
#CMAKE_FLAGS+=\
#	-DBUILD_SHARED=OFF \
#	-DBUILD_STATIC=ON \
#	-DBUILD_TESTS=ON \
#	-DENABLE_CODECOVERAGE=ON \
#	-DBUILD_BENCHMARK=OFF \
#	-DBUILD_EXAMPLES=OFF
#endif

CMAKE_FLAGS += $(CMAKE_ARGS) $(CMAKE_DEBUG) $(CMAKE_STATIC) $(CMAKE_WHY)

#----------------------------------------------------------------------------------------------

include $(MK)/defs

MK_CUSTOM_CLEAN=1

#----------------------------------------------------------------------------------------------

all: bindirs $(TARGET)

include $(MK)/rules

ifeq ($(SLOW),1)
MAKE_J=
else
MAKE_J:=-j$(shell nproc)
endif

$(BINDIR)/Makefile : $(CMAKE_DIR)/CMakeLists.txt
ifeq ($(WHY),1)
	@echo CMake log is in /tmp/cmake-why
endif
	$(SHOW)mkdir -p $(BINDIR)
	$(SHOW)cd $(BINDIR) && cmake $(CMAKE_DIR) $(CMAKE_FLAGS)

$(TARGET): $(BINDIR)/Makefile
	@echo Building $(TARGET) ...
ifneq ($(DRY_RUN),1)
	$(SHOW)$(MAKE) -C $(BINDIR) $(MAKE_J)
else
	@make -C $(BINDIR) $(MAKE_J)
endif

.PHONY: build clean

clean:
ifeq ($(ALL),1)
	$(SHOW)rm -rf $(BINDIR)
else
	$(SHOW)$(MAKE) -C $(BINDIR) clean
endif

#----------------------------------------------------------------------------------------------
