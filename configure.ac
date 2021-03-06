#
# configure.in
#
# See the LICENSE file for the license associated with this file.

AC_INIT([libtpms], [0.6.0])
AC_PREREQ(2.12)
AC_CONFIG_SRCDIR(Makefile.am)
AC_CONFIG_AUX_DIR([.])
AM_CONFIG_HEADER(config.h)

AC_CONFIG_MACRO_DIR([m4])
AC_CANONICAL_TARGET
AM_INIT_AUTOMAKE([foreign 1.6 subdir-objects])

LIBTPMS_VER_MAJOR=`echo $PACKAGE_VERSION | awk -F. '{print $1}'`
LIBTPMS_VER_MINOR=`echo $PACKAGE_VERSION | awk -F. '{print $2}'`
LIBTPMS_VER_MICRO=`echo $PACKAGE_VERSION | awk -F. '{print $3}'`
LIBTPMS_VERSION=$PACKAGE_VERSION
LIBTPMS_VERSION_INFO=`expr $LIBTPMS_VER_MAJOR + $LIBTPMS_VER_MINOR`:$LIBTPMS_VER_MICRO:$LIBTPMS_VER_MINOR

AC_SUBST([LIBTPMS_VER_MAJOR])
AC_SUBST([LIBTPMS_VER_MINOR])
AC_SUBST([LIBTPMS_VER_MICRO])
AC_SUBST([LIBTPMS_VERSION])
AC_SUBST([LIBTPMS_VERSION_INFO])

DEBUG=""
AC_MSG_CHECKING([for debug-enabled build])
AC_ARG_ENABLE(debug, AC_HELP_STRING([--enable-debug], [create a debug build]),
  [if test "$enableval" = "yes"; then
     DEBUG="yes"
     AC_MSG_RESULT([yes])
   else
     DEBUG="no"
     AC_MSG_RESULT([no])
   fi],
  [DEBUG="no",
   AC_MSG_RESULT([no])])

if test "$DEBUG" = "yes"; then
	CFLAGS="$CFLAGS -O0 -g -DDEBUG"
fi

debug_defines=
if test "$DEBUG" = "yes"; then
	debug_defines="-DTPM_DEBUG"
	# Enable the following only if ABSOLUTELY necessary
	# volatile state will be written and behavior changes
	#"-DTPM_VOLATILE_STORE"
fi
AC_SUBST(DEBUG_DEFINES, $debug_defines)

# AX_CHECK_LINK_FLAG needs autoconf 2.64 or later
have_version_script="no"
m4_if(
      m4_version_compare(
                         m4_defn([AC_AUTOCONF_VERSION]),
                         [2.64]),
      -1,
      [],
      [AX_CHECK_LINK_FLAG([-Wl,--version-script=$srcdir/src/test.syms],
                          [have_version_script="yes"],
                          [])]
)

AM_CONDITIONAL([HAVE_VERSION_SCRIPT], [test "x$have_version_script" = "xyes"])

cryptolib=freebl
AC_SUBST(cryptolib, $cryptolib)

AC_ARG_WITH([openssl],
            AC_HELP_STRING([--with-openssl],
                           [build libtpms with openssl library]),
              [AC_CHECK_LIB(crypto,
                            [AES_set_encrypt_key],
                            [],
                            AC_MSG_ERROR(Faulty openssl crypto library))
               AC_CHECK_HEADERS([openssl/aes.h],[],
                            AC_MSG_ERROR(Is openssl-devel/libssl-dev installed?))
               AC_MSG_RESULT([Building with openssl crypto library])
               cryptolib=openssl
              ]
)

case "$cryptolib" in
freebl)
	AM_CONDITIONAL(LIBTPMS_USE_FREEBL, true)
	AM_CONDITIONAL(LIBTPMS_USE_OPENSSL, false)
        AC_DEFINE([USE_FREEBL_CRYPTO_LIBRARY],
                  [1],
                  [use freebl crypto library])

	CFLAGS_save=$CFLAGS

	AC_CHECK_HEADERS([gmp.h],[],
			 AC_MSG_ERROR(gmp-devel/libgmp-dev is bad))

	CFLAGS="$(nspr-config --cflags)"
	if test $? -ne 0; then
		AC_MSG_ERROR(Could not find nspr-config. Is nspr-devel/libnspr4-dev installed?)
	fi
	CPPFLAGS=$CFLAGS
	AC_CHECK_HEADERS([plbase64.h],[],
			 AC_MSG_ERROR(You must install nspr-devel/libnspr4-dev))

	CFLAGS="$(nss-config --cflags) $CFLAGS"
	if test $? -ne 0; then
		AC_MSG_ERROR(Could not find nss-config. Is nss-devel/libnss3-dev installed?)
	fi
	CPPFLAGS="$CPPFLAGS $CFLAGS"
	AC_CHECK_HEADERS([sslerr.h],[],
			 AC_MSG_ERROR(nss-devel/libnss3-dev is bad))

	# Check for missing headers
	AC_CHECK_HEADERS([blapi.h],[],
			 AC_MSG_ERROR(nss-softokn-freebl-devel/libnss3-dev is missing blapi.h))
	# Check for missing freebl library or missing library functions
	LIBS_save="$LIBS"
	LIBS="$(nss-config --libs) $(nspr-config --libs)"
	AC_SEARCH_LIBS([AES_CreateContext], [freebl],[],
             AC_MSG_ERROR("Could not find AES_CreateContext(). Is nss-softokn-freebl-devel/libnss3-dev installed?"),
             [])
	LIBS="$LIBS_save"
	CFLAGS="$CFLAGS_save $CFLAGS"
	;;
openssl)
	AM_CONDITIONAL(LIBTPMS_USE_FREEBL, false)
	AM_CONDITIONAL(LIBTPMS_USE_OPENSSL, true)
        AC_DEFINE([USE_OPENSSL_CRYPTO_LIBRARY],
                  [1],
                  [use openssl crypto library])
	;;
esac

AC_ARG_WITH([tpm2],
	AC_HELP_STRING([--with-tpm2],
		       [build libtpms with TPM2 support (experimental)]),
	AC_MSG_RESULT([Building with TPM2 support])
	if test "x$cryptolib" = "xfreebl"; then
		AC_MSG_ERROR([TPM2 support requires openssl crypto library])
	fi
	AC_DEFINE_UNQUOTED([WITH_TPM2], 1, [whether to support TPM2])
	AM_CONDITIONAL(WITH_TPM2, true),
	AM_CONDITIONAL(WITH_TPM2, false)
)

AC_ARG_ENABLE([sanitizers], AS_HELP_STRING([--enable-sanitizers], [Enable address sanitizing]),
    [SANITIZERS="-fsanitize=address,undefined"], [])
AC_ARG_ENABLE([fuzzer], AS_HELP_STRING([--enable-fuzzer], [Enable fuzzer]),
    [FUZZER="$SANITIZERS -fsanitize=fuzzer"
     AM_CONDITIONAL(WITH_FUZZER, true)],
    [AM_CONDITIONAL(WITH_FUZZER, false)])
AC_SUBST([SANITIZERS])
AC_SUBST([FUZZER])

AC_ARG_ENABLE([test-coverage],
  AS_HELP_STRING([--enable-test-coverage], [Enable test coverage flags]),
  [COVERAGE_CFLAGS="-fprofile-arcs -ftest-coverage" COVERAGE_LDFLAGS="-fprofile-arcs"])

LT_INIT
AC_PROG_CC
AC_PROG_INSTALL
AC_PROG_LIBTOOL

#AM_GNU_GETTEXT_VERSION([0.15])
#AM_GNU_GETTEXT([external])

AC_HEADER_STDC
AC_C_CONST
AC_C_INLINE

AC_TYPE_SIZE_T

AC_CHECK_LIB(c, clock_gettime, LIBRT_LIBS="", LIBRT_LIBS="-lrt")
AC_SUBST([LIBRT_LIBS])

AC_ARG_ENABLE([hardening],
  AS_HELP_STRING([--disable-hardening], [Disable hardening flags]))

if test "x$enable_hardening" != "xno"; then
	# Some versions of gcc fail with -Wstack-protector enabled
	TMP="$($CC -fstack-protector-strong $srcdir/include/libtpms/tpm_error.h 2>&1)"
	if echo $TMP | $GREP 'unrecognized command line option' >/dev/null; then
		HARDENING_CFLAGS="-fstack-protector "
	else
		HARDENING_CFLAGS="-fstack-protector-strong "
	fi

	dnl Must not have -O0 but must have a -O for -D_FORTIFY_SOURCE=2
	TMP1="$(echo $CFLAGS | sed -n 's/.*\(-O0\).*/\1/p')"
	TMP2="$(echo $CFLAGS | sed -n 's/.*\(-O\).*/\1/p')"
	if test -z "$TMP1" && test -n "$TPM2"; then
		HARDENING_CFLAGS="$HARDENING_CFLAGS -D_FORTIFY_SOURCE=2 "
	fi
	dnl Check ld for 'relro' and 'now'
	if $LD --help 2>&1 | $GREP '\-z relro ' > /dev/null; then
		HARDENING_LDFLAGS="$HARDENING_LDFLAGS -Wl,-z,relro "
	fi
	if $LD --help 2>&1 | $GREP '\-z now ' > /dev/null; then
		HARDENING_LDFLAGS="$HARDENING_LDFLAGS -Wl,-z,now "
	fi
	AC_SUBST([HARDENING_CFLAGS])
	AC_SUBST([HARDENING_LDFLAGS])
fi

CFLAGS="$CFLAGS $COVERAGE_CFLAGS -Wall -Werror -Wreturn-type -Wsign-compare -Wno-self-assign"
LDFLAGS="$LDFLAGS $COVERAGE_LDFLAGS"

AC_CONFIG_FILES(Makefile                   \
		dist/libtpms.spec          \
		include/Makefile           \
		include/libtpms/Makefile   \
		include/libtpms/tpm_library.h \
		man/Makefile               \
		man/man3/Makefile          \
		src/Makefile               \
		libtpms.pc                 \
		tests/Makefile)
AC_OUTPUT

if test -z "$enable_debug" ; then
    enable_debug="no"
fi
if test -z "$with_tpm2"; then
    with_tpm2=no
fi

echo
echo "CFLAGS=$CFLAGS"
echo "HARDENING_CFLAGS=$HARDENING_CFLAGS"
echo "HARDENING_LDFLAGS=$HARDENING_LDFLAGS"
echo "LDFLAGS=$LDFLAGS"
echo
echo "Version to build    : $PACKAGE_VERSION"
echo "Crypto library      : $cryptolib"
echo "Debug build         : $enable_debug"
echo "With TPM2 support   : $with_tpm2"
echo "HAVE_VERSION_SCRIPT : $have_version_script"
echo
echo
