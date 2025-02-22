
###########################################################################
# Note: This Makefile is intended for development.  For production
# builds, use build_dynamic/Makefile.
###########################################################################

# Does not currently support -DCPU_CHECK_...

# BASEFLAGS=-g -DZSTD_MULTITHREAD -DSTATIC_ZSTD
BASEFLAGS=-g -mavx2 -mbmi -mbmi2 -mfma -mlzcnt -DZSTD_MULTITHREAD
# BASEFLAGS=-g -msse4.2 -DZSTD_MULTITHREAD

include Makefile.src

# Respect the environment:
# Use defaults below only if not set in env or make arguments
CC		?= gcc
CXX		?= g++
CFLAGS		?= -O3
CXXFLAGS	?= -O2
ZLIB		?= -lz
# ZLIB		?= -L. ../zlib-1.2.12/libz-64.a

# dynamic zstd: remove -DSTATIC_ZSTD above, change this to -lzstd, replace
# OBJ with OBJ_NO_ZSTD below
ZSTD		?=

BLASFLAGS64	?= -llapack -lf77blas -latlas
ARCH32		?=

# Mandatory flags added to defaults or env settings
CFLAGS		+= -std=gnu99 $(BASEFLAGS) $(CWARN) $(CINCLUDE)
CXXFLAGS	+= -std=c++14 $(BASEFLAGS) $(CXXWARN) $(CXXINCLUDE)
LDFLAGS		+= -lm -lpthread $(ZLIB) $(ZSTD)

# Installation defaults
MKDIR		?= mkdir
INSTALL		?= install
STRIP_CMD	?= strip
PREFIX		?= /usr/local
DESTDIR		?= .

UNAME := $(shell uname)
ifeq ($(UNAME), Darwin)
  # argh
  CFLAGS += -I/usr/local/include
  CXXFLAGS += -I/usr/local/include
  BLASFLAGS=-framework Accelerate
  BLASFLAGS64=-framework Accelerate
  LDFLAGS=-ldl -lpthread -lz -lzstd -L/usr/local/lib
endif

%.o: %.c
	$(CC) -c $(CFLAGS) $(ARCH32) -o $@ $<

%.o: %.cc
	$(CXX) -c $(CXXFLAGS) $(ARCH32) -o $@ $<

all: plink2 pgen_compress

# for clean build, "make clean" first
# Run mkdir for both plink2 and pgen_compress as we don't know which
# target will run first
plink2: $(OBJ)
	$(MKDIR) -p bin
	$(CXX) $(ARCH32) $(OBJ) -o bin/plink2 $(BLASFLAGS64) $(LDFLAGS)

# basic pgenlib usage example; also needed for tests
pgen_compress: $(PGCOBJ)
	$(MKDIR) -p bin
	$(CXX) $(PGCOBJ) \
		-o bin/pgen_compress

.PHONY: install-strip install clean

install-strip: install
	$(STRIP_CMD) $(DESTDIR)$(PREFIX)/bin/*

install: all
	$(MKDIR) -p $(DESTDIR)$(PREFIX)/bin
	$(INSTALL) -c bin/* $(DESTDIR)$(PREFIX)/bin

clean:
	rm -f $(CLEAN)
