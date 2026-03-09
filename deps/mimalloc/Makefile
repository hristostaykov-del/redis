# Mimalloc Makefile for Redis dependency build
# Produces lib/libmimalloc.a as a static library
#
# Flags are split into CONFIGURE_CFLAGS (auto-detected by this Makefile)
# and SPECIFIED_CFLAGS (caller-provided via CFLAGS=...), following the same
# convention used by jemalloc's autoconf-generated Makefile.

CC ?= cc

# User-provided flags (passed in by deps/Makefile via CFLAGS="...")
SPECIFIED_CFLAGS := $(CFLAGS)
SPECIFIED_LDFLAGS := $(LDFLAGS)

# Auto-detected flags, built up below.
CONFIGURE_CFLAGS :=

# --- Feature / compiler detection ---

# Check for -std=gnu11 support, fall back to -std=c99.
TEST_GNU11 := $(shell echo 'int main(void){return 0;}' | $(CC) -std=gnu11 -x c -o /dev/null - 2>/dev/null && echo yes)
ifeq ($(TEST_GNU11),yes)
  CONFIGURE_CFLAGS += -std=gnu11
else
  CONFIGURE_CFLAGS += -std=c99
endif

ENABLE_DEBUG ?= 0

ifneq ($(ENABLE_DEBUG),1)
  # Release mode: check for -O3 support, fall back to -O2.
  TEST_O3 := $(shell echo 'int main(void){return 0;}' | $(CC) -O3 -x c -o /dev/null - 2>/dev/null && echo yes)
  ifeq ($(TEST_O3),yes)
    CONFIGURE_CFLAGS += -O3
  else
    CONFIGURE_CFLAGS += -O2
  endif
  CONFIGURE_CFLAGS += -DMI_BUILD_RELEASE
else
  # Debug mode: minimal optimization for debuggability.
  CONFIGURE_CFLAGS += -O0
endif

CONFIGURE_CFLAGS += -g

# Warnings
CONFIGURE_CFLAGS += -Wall -Wextra -Wno-unknown-pragmas -Wstrict-prototypes

# --- Compose final flags ---

MI_CFLAGS = $(CONFIGURE_CFLAGS) $(SPECIFIED_CFLAGS) \
            -Iinclude \
            -fPIC \
            -fvisibility=hidden \
            -ftls-model=initial-exec \
            -fno-builtin-malloc \
            -DMI_STATIC_LIB \
            -DMI_MALLOC_OVERRIDE

MI_CC_CMD = $(CC) $(MI_CFLAGS)
MI_LDFLAGS = $(SPECIFIED_LDFLAGS)

AR ?= ar
ARFLAGS ?= rcs

MI_SOURCES = \
    src/alloc.c \
    src/alloc-aligned.c \
    src/alloc-posix.c \
    src/arena.c \
    src/arena-meta.c \
    src/bitmap.c \
    src/heap.c \
    src/init.c \
    src/libc.c \
    src/options.c \
    src/os.c \
    src/page.c \
    src/page-map.c \
    src/random.c \
    src/stats.c \
    src/prim/prim.c \
    src/theap.c \
    src/threadlocal.c

MI_OBJECTS = $(MI_SOURCES:.c=.o)

all: lib/libmimalloc.a

lib/libmimalloc.a: $(MI_OBJECTS)
	@mkdir -p lib
	@echo "==== mimalloc build parameters ===="
	@echo "CC:              $(CC)"
	@echo "CONFIGURE_CFLAGS:$(CONFIGURE_CFLAGS)"
	@echo "SPECIFIED_CFLAGS:$(SPECIFIED_CFLAGS)"
	@echo "MI_CFLAGS:       $(MI_CFLAGS)"
	@echo "MI_LDFLAGS:      $(MI_LDFLAGS)"
	@echo "ENABLE_DEBUG:    $(ENABLE_DEBUG)"
	@echo "AR:              $(AR)"
	@echo "ARFLAGS:         $(ARFLAGS)"
	@echo "==================================="
	$(AR) $(ARFLAGS) $@ $(MI_OBJECTS)

%.o: %.c
	$(MI_CC_CMD) -c $< -o $@

clean:
	rm -f $(MI_OBJECTS)
	rm -rf lib

.PHONY: all clean
