# Author: Eli Bendersky (eliben@gmail.com)
# Revised by Ruoxin(Amy) Jiang
#-------------------------------------------------------------------------------

ifneq ($(wildcard config.mak),)
    include config.mak
else
    $(info )
    $(info Error: config.mak not found.  Create one with your LLVM build info.)
    $(info Example:)
    $(info )
    $(info LLVM_SRC_PATH := /home/llvm/llvm-3.6-svn/llvm/)
    $(info LLVM_BUILD_PATH := /home/llvm/llvm-3.6-svn/llvm/)
    $(info LLVM_BIN_PATH := $$(LLVM_BUILD_PATH)/Release+Asserts/bin/)
    $(info )
    $(error Exiting...)
endif

# Run make BUILD_NINJA=1 to enable these paths
ifdef BUILD_NINJA
	LLVM_BUILD_PATH := $$HOME/llvm/build/svn-ninja-release
	LLVM_BIN_PATH 	:= $(LLVM_BUILD_PATH)/bin
endif

$(info -----------------------------------------------)
$(info Using LLVM_SRC_PATH = $(LLVM_SRC_PATH))
$(info Using LLVM_BUILD_PATH = $(LLVM_BUILD_PATH))
$(info Using LLVM_BIN_PATH = $(LLVM_BIN_PATH))
$(info -----------------------------------------------)

VPATH=.:../common:$(PANDA_SRC_PATH)/panda/src:$(PANDA_BUILD_DIR)/i386-softmmu

CXX := $(LLVM_BIN_PATH)/clang++
CXXFLAGS := -fexceptions -O2 -g -gdwarf-2 -std=c++14 -I/usr/include/jsoncpp
CXXFLAGS += -fno-omit-frame-pointer -I../include -frtti

PLUGIN_CXXFLAGS := -fpic -fexceptions

LLVM_CXXFLAGS := $(filter-out -fno-exceptions,$(shell $(LLVM_BIN_PATH)/llvm-config --cxxflags))
LLVM_LDFLAGS := $(shell $(LLVM_BIN_PATH)/llvm-config --ldflags --libs --system-libs)

LLVM_LDFLAGS_NOLIBS := $(shell $(LLVM_BIN_PATH)/llvm-config --ldflags)

OTHER_LIBS := -lpq -ljsoncpp -lodb -lodb-pgsql -L/usr/local/lib

CLANG_INCLUDES := \
	-I$(LLVM_SRC_PATH)/tools/clang/include \
	-I$(LLVM_BUILD_PATH)/tools/clang/include

CLANG_LIBS := -Wl,--start-group \
	-lclangAST \
	-lclangAnalysis \
	-lclangApplyReplacements \
	-lclangBasic \
	-lclangDriver \
	-lclangEdit \
	-lclangFrontend \
	-lclangFrontendTool \
	-lclangFormat \
	-lclangLex \
	-lclangParse \
	-lclangSema \
	-lclangEdit \
	-lclangASTMatchers \
	-lclangRewrite \
	-lclangRewriteFrontend \
	-lclangStaticAnalyzerFrontend \
	-lclangStaticAnalyzerCheckers \
	-lclangStaticAnalyzerCore \
	-lclangSerialization \
	-lclangToolingCore \
	-lclangTooling \
	-Wl,--end-group

BUILDDIR := build

.PHONY: all lavatool
all: lavatool build/clang-apply-replacements

lavatool: make_builddir \
	emit_build_config \
	$(BUILDDIR)/lavaTool

.PHONY: emit_build_config
emit_build_config: make_builddir
	@echo $(LLVM_BIN_PATH) > $(BUILDDIR)/_build_config

.PHONY: make_builddir
make_builddir:
	@test -d $(BUILDDIR) || mkdir $(BUILDDIR)

$(BUILDDIR)/%.o: %.cpp
	$(CXX) -c -o $@ $< $(LLVM_CXXFLAGS) $(CLANG_INCLUDES) $(CXXFLAGS)

$(BUILDDIR)/%.o: %.cxx
	$(CXX) -c -o $@ $< $(LLVM_CXXFLAGS) $(CLANG_INCLUDES) $(CXXFLAGS)

$(BUILDDIR)/%.o: %.c
	$(CC) -c -o $@ $< $(CFLAGS)

$(BUILDDIR)/lavaTool: $(BUILDDIR)/lavaTool.o $(BUILDDIR)/lavaDB.o $(BUILDDIR)/lava-odb.o
	$(CXX) -o $@ $^ $(CLANG_LIBS) $(LLVM_LDFLAGS) $(OTHER_LIBS)

$(BUILDDIR)/clang-apply-replacements: $(BUILDDIR)/ClangApplyReplacementsMain.o $(BUILDDIR)/ApplyReplacements.o
	$(CXX) -o $@ $^ $(CLANG_LIBS) $(LLVM_LDFLAGS) $(OTHER_LIBS)

$(BUILDDIR)/lavaTool.o: lava-odb.hxx lexpr.hxx Makefile

$(BUILDDIR)/%.o: Makefile

$(BUILDDIR)/attackPoints: $(BUILDDIR)/attackPoints.o
	$(CXX) -o $@ $< $(CLANG_LIBS) $(LLVM_LDFLAGS)

%-odb.cxx %-odb.hxx: ../include/%.hxx Makefile
	LD_LIBRARY_PATH=/usr/local/lib odb -d pgsql --std c++11 -o . \
	--generate-query --generate-schema --generate-prepared \
	--cxx-prologue '#include "../include/pgarray.hxx"' \
	--sql-name-case lower $<

.PHONY: clean
clean:
	rm -rf $(BUILDDIR)/* *~ *.o lava-odb.*xx lava.sql

