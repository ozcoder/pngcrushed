dnl Process with autoconf
AC_INIT(pngcrushed, 0.2)
AC_PREREQ([2.69])
AM_INIT_AUTOMAKE([1.15])

dnl Check for toolchain
AC_PROG_CC
AC_PROG_LN_S
AC_PROG_RANLIB

dnl Check for standard headers:
AC_HEADER_ASSERT
AC_HEADER_STAT
AC_HEADER_STDC
AC_HEADER_TIME

AC_CHECK_HEADERS([alloc.h altivec.h config.h float.h fp.h kernel.h libc/dosio.h malloc.h math.h mem.h msa.h sys/utime.h time.h unix.h unixio.h utime.h])

AC_CHECK_FUNCS([clock_gettime memset mkdir strerror strrchr strtol utime])
AC_FUNC_MALLOC
AC_CHECK_TYPES([ptrdiff_t])
# arm_neon.h

# immintrin.h

# m68881.h

LIBS=-lm

AC_ARG_WITH([zlib],
  [AS_HELP_STRING([--with-zlib], [Use the system zlib instead of the included one])],
  [LOCAL_ZLIB=no],
  [LOCAL_ZLIB=yes])

AS_IF([test "x$LOCAL_ZLIB" = "xno"], [
  LIBS="${LIBS} -lz"
])

AM_CONDITIONAL(LOCAL_ZLIB, test x$LOCAL_ZLIB = xyes)

AS_IF([test "x$LOCAL_ZLIB" = "xyes"], [
  AC_DEFINE(LOCAL_ZLIB, [], [Need zlib source files])
])

AC_ARG_WITH([png],
  [AS_HELP_STRING([--with-png], [Use the system libpng instead of the included one])],
  [LOCAL_PNG=no],
  [LOCAL_PNG=yes])

AS_IF([test "x$LOCAL_PNG" = "xno"], [
  LIBS="${LIBS} -lpng"
])

AM_CONDITIONAL(LOCAL_PNG, test x$LOCAL_PNG = xyes)

AS_IF([test "x$LOCAL_PNG" = "xyes"], [
  AC_DEFINE(LOCAL_PNG, [], [Need libpng source files])
])

# Last line
AC_OUTPUT(Makefile)
