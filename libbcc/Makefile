ifeq ($(V),1)
	Q =
	msg =
else
	Q = @
	msg = @printf '  %-8s %s%s\n' "$(1)" "$(notdir $(2))" "$(if $(3), $(3))";
endif

CXX := /work/x86_64-linux-musl-cross/bin/x86_64-linux-musl-c++
CC :=  /work/x86_64-linux-musl-cross/bin/x86_64-linux-musl-gcc
AR := /work/x86_64-linux-musl-cross/bin/x86_64-linux-musl-ar

BCCDIR_SRC ?= /work/musl-bcc/bcc/src
OBJDIR ?= .
STATIC_OBJDIR := $(OBJDIR)/bcc-static.dir

# table_storage.cc.o 
# bcc_debug.cc.o
# bpf_module_rw_engine.cc.o
# bpf_module.cc.o
# json_map_decl_visitor.cc.o
OBJS := bcc_common.cc.o \
	bcc_btf.cc.o \
	exported_files.cc.o \
	shared_table.cc.o \
	bpffs_table.cc.o \
	common.cc.o \
	bcc_syms.cc.o \
	usdt/usdt.cc.o \
	usdt/usdt_args.cc.o

OBJS_C := bcc_elf.c.o \
	  bcc_perf_map.c.o \
	  bcc_proc.c.o

STATIC_OBJS := $(addprefix $(STATIC_OBJDIR)/,$(OBJS))
STATIC_OBJS_C := $(addprefix $(STATIC_OBJDIR)/,$(OBJS_C))

build: $(STATIC_OBJS) $(STATIC_OBJS_C)
	$(AR) qc libbcc.a $(STATIC_OBJS) $(STATIC_OBJS_C)

$(STATIC_OBJDIR) $(STATIC_OBJDIR_C):
	$(call msg,MKDIR,$@)
	$(Q)mkdir -p $@/usdt

CFLAGS_CXX ?= -I$(BCCDIR_SRC) \
	-I$(BCCDIR_SRC)/cc \
	-I$(BCCDIR_SRC)/cc/frontends/b \
	-I$(BCCDIR_SRC)/cc/frontends/clang \
	-I$(BCCDIR_SRC)/cc/libbpf/include \
	-I$(BCCDIR_SRC)/cc/libbpf/include/uapi \
	-I/work/x86_64-linux-musl-cross/x86_64-linux-musl/include \
	-Wall -fno-rtti -fPIC -DBCC_PROG_TAG_DIR='"/var/tmp/bcc"' -Wno-unused-result -DLLVM_MAJOR_VERSION=11 \
	-g -O2 -DNDEBUG   -D_GNU_SOURCE -D__STDC_CONSTANT_MACROS -D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS \
	-std=gnu++1y 

CFLAGS ?= -I. \
	-I$(BCCDIR_SRC) \
	-I$(BCCDIR_SRC)/cc \
	-I$(BCCDIR_SRC)/cc/frontends/b \
	-I$(BCCDIR_SRC)/cc/frontends/clang \
	-I$(BCCDIR_SRC)/cc/libbpf/include \
	-I$(BCCDIR_SRC)/cc/libbpf/include/uapi \
	-I/work/x86_64-linux-musl-cross/x86_64-linux-musl/include \
	-g -Wall -Werror=pointer-to-int-cast -fPIC -Wno-unused-result -DCOMPAT_NEED_REALLOCARRAY -O2 -DNDEBUG \
	-D_GNU_SOURCE -D__STDC_CONSTANT_MACROS -D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS
	

$(STATIC_OBJDIR)/%.c.o: %.c | $(STATIC_OBJDIR)
	$(call msg,CC,$@)
	$(CC) $(CFLAGS) -c $< -o $@

$(STATIC_OBJDIR)/%.o: % | $(STATIC_OBJDIR)
	$(call msg,CXX,$@)
	$(CXX) $(CFLAGS_CXX) -c $< -o $@

clean:
	@rm -rf $(STATIC_OBJS) $(STATIC_OBJS_C) $(STATIC_OBJDIR) ./libbcc.a

.PHONY: clean
