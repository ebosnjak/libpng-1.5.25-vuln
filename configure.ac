# configure.ac

dnl Process this file with autoconf to produce a configure script.
dnl
dnl Minor upgrades (compatible ABI): increment the package version
dnl (third field in two places below) and set the PNGLIB_RELEASE
dnl variable.
dnl
dnl Major upgrades (incompatible ABI): increment the package major
dnl version (second field, or first if desired), set the minor
dnl to 0, set PNGLIB_MAJOR below *and* follow the instructions in
dnl Makefile.am to upgrade the package name.

dnl This is here to prevent earlier autoconf from being used, it
dnl should not be necessary to regenerate configure if the time
dnl stamps are correct
AC_PREREQ(2.59)

dnl Version number stuff here:

AC_INIT([libpng], [1.5.25], [png-mng-implement@lists.sourceforge.net])
AM_INIT_AUTOMAKE([1.13 serial-tests subdir-objects])
dnl stop configure from automagically running automake
AM_MAINTAINER_MODE

PNGLIB_VERSION=1.5.25
PNGLIB_MAJOR=1
PNGLIB_MINOR=5
PNGLIB_RELEASE=25

dnl End of version number stuff

AC_CONFIG_SRCDIR([pngget.c])
AC_CONFIG_HEADERS([config.h])

# Checks for programs.
AC_LANG([C])
AC_PROG_CC
AM_PROG_AS
AC_PROG_LD
AC_PROG_CPP
AC_PROG_SED
AC_PROG_AWK
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET
LT_INIT([win32-dll])

# Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADERS([malloc.h stdlib.h string.h strings.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_TYPE_SIZE_T
AC_STRUCT_TM
AC_C_RESTRICT

# This is a remnant of the old cc -E validation, where it may have been
# necessary to use a different preprocessor for .dfn files
DFNCPP="$CPP"
AC_SUBST(DFNCPP)

# Checks for library functions.
AC_FUNC_STRTOD
AC_CHECK_FUNCS([memset], , AC_ERROR([memset not found in libc]))
AC_CHECK_FUNCS([pow], , AC_CHECK_LIB(m, pow, , AC_ERROR([cannot find pow])) )
AC_ARG_WITH(zlib-prefix,
   AC_HELP_STRING([--with-zlib-prefix],
                  [prefix that may have been used in installed zlib]),
                  [ZPREFIX=${withval}],
                  [ZPREFIX='z_'])
AC_CHECK_LIB(z, zlibVersion, ,
    AC_CHECK_LIB(z, ${ZPREFIX}zlibVersion, ,
                 AC_ERROR([zlib not installed])))

# The following is for pngvalid, to ensure it catches FP errors even on
# platforms that don't enable FP exceptions, the function appears in the math
# library (typically), it's not an error if it is not found.
AC_CHECK_LIB([m], [feenableexcept])
AC_CHECK_FUNCS([feenableexcept])

LIBPNG_DEFINES=-DPNG_CONFIGURE_LIBPNG
LIBPNG_DEFINES=$LIBPNG_DEFINES
AC_SUBST(LIBPNG_DEFINES)

AC_MSG_CHECKING([if using Solaris linker])
SLD=`$LD --version 2>&1 | grep Solaris`
if test "$SLD"; then
    have_solaris_ld=yes
    AC_MSG_RESULT(yes)
else
    have_solaris_ld=no
    AC_MSG_RESULT(no)
fi
AM_CONDITIONAL(HAVE_SOLARIS_LD, test "$have_solaris_ld" = "yes")

AC_MSG_CHECKING([if libraries can be versioned])
# Special case for PE/COFF platforms: ld reports
# support for version-script, but doesn't actually
# DO anything with it.
case $host in
*cygwin* | *mingw32* | *interix* )
    have_ld_version_script=no
    AC_MSG_RESULT(no)
;;
* )

if test "$have_solaris_ld" = "yes"; then
    GLD=`$LD --help < /dev/null 2>&1 | grep 'M mapfile'`
else
    GLD=`$LD --help < /dev/null 2>/dev/null | grep version-script`
fi

if test "$GLD"; then
    have_ld_version_script=yes
    AC_MSG_RESULT(yes)
else
    have_ld_version_script=no
    AC_MSG_RESULT(no)
    AC_MSG_WARN(*** You have not enabled versioned symbols.)
fi
;;
esac

AM_CONDITIONAL(HAVE_LD_VERSION_SCRIPT, test "$have_ld_version_script" = "yes")

if test "$have_ld_version_script" = "yes"; then
    AC_MSG_CHECKING([for symbol prefix])
    SYMBOL_PREFIX=`echo "PREFIX=__USER_LABEL_PREFIX__" \
                  | ${DFNCPP-${CC-gcc} -E} - 2>&1 \
                  | ${EGREP-grep} "^PREFIX=" \
                  | ${SED-sed} -e "s:^PREFIX=::" -e "s:__USER_LABEL_PREFIX__::"`
    AC_SUBST(SYMBOL_PREFIX)
    AC_MSG_RESULT($SYMBOL_PREFIX)
fi

# Substitutions for .in files
AC_SUBST(PNGLIB_VERSION)
AC_SUBST(PNGLIB_MAJOR)
AC_SUBST(PNGLIB_MINOR)
AC_SUBST(PNGLIB_RELEASE)

# Additional arguments (and substitutions)
# Allow the pkg-config directory to be set
AC_ARG_WITH(pkgconfigdir,
	AC_HELP_STRING([--with-pkgconfigdir],
	[Use the specified pkgconfig dir (default is libdir/pkgconfig)]),
	[pkgconfigdir=${withval}],
	[pkgconfigdir='${libdir}/pkgconfig'])

AC_SUBST([pkgconfigdir])
AC_MSG_NOTICE([pkgconfig directory is ${pkgconfigdir}])

# Make the *-config binary config scripts optional
AC_ARG_WITH(binconfigs,
	AC_HELP_STRING([--with-binconfigs],
		[Generate shell libpng-config scripts as well as pkg-config data]
		[@<:@default=yes@:>@]),
	[if test "${withval}" = no; then
		binconfigs=
		AC_MSG_NOTICE([libpng-config scripts will not be built])
	else
		binconfigs='${binconfigs}'
	fi],
	[binconfigs='${binconfigs}'])
AC_SUBST([binconfigs])

# Control over what links are made for installed files.  Versioned files are
# always installed, when the following options are turned on corresponding
# unversioned links are also created (normally as symbolic links):
AC_ARG_ENABLE([unversioned-links],
   AS_HELP_STRING([[[--enable-unversioned-links]]],
      [Installed libpng header files are placed in a versioned subdirectory]
      [and installed libpng library (including DLL) files are versioned.]
      [If this option is enabled unversioned links will be created pointing to]
      [the corresponding installed files.  If you use libpng.pc or]
      [libpng-config for all builds you do not need these links, but if you]
      [compile programs directly they will typically #include <png.h> and]
      [link with -lpng; in that case you need the links.]
      [The links can be installed manually using 'make install-header-links']
      [and 'make install-library-links' and can be removed using the]
      [corresponding uninstall- targets.  If you do enable this option every]
      [libpng 'make install' will recreate the links to point to the just]
      [installed version of libpng.  The default is to create the links;]
      [use --disable-unversioned-links to change this]))

# The AM_CONDITIONAL test is written so that the default is enabled;
# --disable-unversioned-links must be given to turn the option off.
AM_CONDITIONAL([DO_INSTALL_LINKS],[test "$enable_unversioned_links" != "no"])

AC_ARG_ENABLE([unversioned-libpng-pc],
   AS_HELP_STRING([[[--enable-unversioned-libpng-pc]]],
      [Install the configuration file 'libpng.pc' as a link to the versioned]
      [version.  This is done by default - use --disable-unversioned-libpng-pc]
      [to change this.]))
AM_CONDITIONAL([DO_INSTALL_LIBPNG_PC],
   [test "$enable_unversioned_libpng_pc" != "no"])

AC_ARG_ENABLE([unversioned-libpng-config],
   AS_HELP_STRING([[[--enable-unversioned-libpng-config]]],
      [Install the configuration file 'libpng-config' as a link to the]
      [versioned version.  This is done by default - use]
      [--disable-unversioned-libpng-config to change this.]))
AM_CONDITIONAL([DO_INSTALL_LIBPNG_CONFIG],
   [test "$enable_unversioned_libpng_config" != "no"])

# HOST SPECIFIC OPTIONS
# =====================
#
# ARM
# ===
#
# ARM NEON (SIMD) support.

AC_ARG_ENABLE([arm-neon],
   AS_HELP_STRING([[[--enable-arm-neon]]],
      [Enable ARM NEON optimizations: =no/off, check, api, yes/on:]
      [no/off: disable the optimizations; check: use internal checking code]
      [(deprecated and poorly supported); api: disable by default, enable by]
      [a call to png_set_option; yes/on: turn on unconditionally.]
      [If not specified: determined by the compiler.]),
   [case "$enableval" in
      no|off)
         # disable the default enabling on __ARM_NEON__ systems:
         AC_DEFINE([PNG_ARM_NEON_OPT], [0],
                   [Disable ARM Neon optimizations])
         # Prevent inclusion of the assembler files below:
         enable_arm_neon=no;;
      check)
         AC_DEFINE([PNG_ARM_NEON_CHECK_SUPPORTED], [],
                   [Check for ARM Neon support at run-time]);;
      api)
         AC_DEFINE([PNG_ARM_NEON_API_SUPPORTED], [],
                   [Turn on ARM Neon optimizations at run-time]);;
      yes|on)
         AC_DEFINE([PNG_ARM_NEON_OPT], [2],
                   [Enable ARM Neon optimizations])
         AC_MSG_WARN([--enable-arm-neon: please specify 'check' or 'api', if]
            [you want the optimizations unconditionally pass -mfpu=neon]
            [to the compiler.]);;
      *)
         AC_MSG_ERROR([--enable-arm-neon=${enable_arm_neon}: invalid value])
   esac])

# Add ARM specific files to all builds where the host_cpu is arm ('arm*') or
# where ARM optimizations were explicitly requested (this allows a fallback if a
# future host CPU does not match 'arm*')

AM_CONDITIONAL([PNG_ARM_NEON],
   [test "$enable_arm_neon" != 'no' &&
    case "$host_cpu" in
      arm*) :;;
      *)    test "$enable_arm_neon" != '';;
    esac])
 
AC_MSG_NOTICE([[Extra options for compiler: $PNG_COPTS]])

# Config files, substituting as above
AC_CONFIG_FILES([Makefile libpng.pc:libpng.pc.in])
AC_CONFIG_FILES([libpng-config:libpng-config.in],
		[chmod +x libpng-config])

AC_OUTPUT
