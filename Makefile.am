bin_PROGRAMS = pngcrushed
pngcrushed_SOURCES = pngcrushed.c
pngcrushed_CFLAGS = -std=c11 -O3 -funroll-loops -fomit-frame-pointer -Wall
# -DPNG_SET_OPTION_SUPPORTED
pngcrushed_CPPFLAGS = -DNO_GZ

DEFAULT_INCLUDES =

if LOCAL_ZLIB
pngcrushed_SOURCES += adler32.c crc32.c compress.c deflate.c infback.c inffast.c inflate.c inftrees.c trees.c uncompr.c zutil.c
else
# Usually the default /usr/include
pngcrushed_CPPFLAGS += `pkg-config --cflags-only-I zlib`
endif
if LOCAL_PNG
pngcrushed_CPPFLAGS += -I. -DLIBPNG_UNIFIED
else
pngcrushed_CPPFLAGS += `pkg-config --cflags-only-I libpng`
endif
