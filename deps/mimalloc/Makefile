# Mimalloc Makefile for Redis dependency build
# Produces lib/libmimalloc.a as a static library

STD= -std=gnu11
WARN= -Wall -Wextra -Wno-unknown-pragmas -Wstrict-prototypes
OPT= -O3
DEBUG= -g

MI_CFLAGS= $(STD) $(WARN) $(OPT) $(DEBUG) $(CFLAGS) \
           -Iinclude \
           -fPIC \
           -fvisibility=hidden \
           -ftls-model=initial-exec \
           -fno-builtin-malloc \
           -DMI_STATIC_LIB \
           -DMI_MALLOC_OVERRIDE \
           -DMI_BUILD_RELEASE

MI_CC=$(CC) $(MI_CFLAGS)

AR= ar
ARFLAGS= rcs

MI_SOURCES= \
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

MI_OBJECTS= $(MI_SOURCES:.c=.o)

all: lib/libmimalloc.a

lib/libmimalloc.a: $(MI_OBJECTS)
	@mkdir -p lib
	$(AR) $(ARFLAGS) $@ $(MI_OBJECTS)

%.o: %.c
	$(MI_CC) -c $< -o $@

clean:
	rm -f $(MI_OBJECTS)
	rm -rf lib

.PHONY: all clean
