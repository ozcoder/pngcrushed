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

AC_CHECK_HEADERS([alloc.h altivec.h config.h float.h fp.h kernel.h libc/dosio.h malloc.h math.h mem.h msa.h stdarg.h sys/utime.h time.h unix.h unixio.h utime.h])

AC_CHECK_FUNCS([clock_gettime memset mkdir strerror strrchr strtol utime])
AC_FUNC_MALLOC
AC_CHECK_TYPES([ptrdiff_t])
AC_CHECK_HEADERS([arm_neon.h immintrin.h m68881.h])

dnl AS_IF([test "$ac_cv_header_immintrin_h" = "yes"], [
dnl   CXXFLAGS_save=$CXXFLAGS
dnl   CXXFLAGS="$CXXFLAGS"
dnl 
dnl   XXX_PROGRAM="
dnl #ifdef HAVE_IMMINTRIN_H
dnl #include <immintrin.h>
dnl #endif
dnl int main(void) {
dnl     const __m128i x = _mm_setzero_si128();
dnl     return 0;
dnl }
dnl "
dnl   AC_MSG_CHECKING([whether $CXX supports immintrin])
dnl   AC_COMPILE_IFELSE(
dnl     [AC_LANG_SOURCE([$XXX_PROGRAM])],
dnl     [AC_MSG_RESULT([yes])],
dnl     [AC_MSG_RESULT([no])]
dnl   )
dnl 
dnl   CXXFLAGS=$CXXFLAGS_save
dnl ])


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
AC_CONFIG_FILES([Makefile])
AC_OUTPUT
