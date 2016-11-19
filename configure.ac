AC_INIT([irssi-otr],[1.0.1],[dgoulet@ev0ke.net],[],[https://github.com/cryptodotis/irssi-otr])
AC_CONFIG_AUX_DIR([config])
AC_CANONICAL_TARGET
AC_CANONICAL_HOST
AC_CONFIG_MACRO_DIR([config])
AM_INIT_AUTOMAKE([foreign dist-bzip2 no-dist-gzip])
AC_CONFIG_HEADERS([config.h])
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

AC_PROG_GREP
AC_PROG_CC

# We do not want to create a .a for the module, so disable by default.
AM_DISABLE_STATIC
AM_PROG_LIBTOOL

# Check for pthread
AC_CHECK_LIB([pthread], [pthread_create], [],
	[AC_MSG_ERROR([Cannot find libpthread. Use [LDFLAGS]=-Ldir to specify its location.])]
)

AC_ARG_WITH([irssi-headers],
	AS_HELP_STRING([--with-irssi-headers],
	[The include directory prefix for the irssi header files]),
	[IRSSI_HEADER_DIR="$withval"],
	[IRSSI_HEADER_DIR="\"\""])

# Check for Glib. It needs to be installed anyway or this macro will not be defined.
AM_PATH_GLIB_2_0([2.22.0], [],
			[AC_MSG_ERROR([Glib 2.22 is required in order to compile.
			Please install the Glib development files.])], [gmodule])

AM_PATH_LIBGCRYPT(1:1.2.0,,AC_MSG_ERROR(libgcrypt 1.2.0 or newer is required.))

AM_PATH_LIBOTR(4.1.0, [], [AC_MSG_ERROR([libotr 4.1.0 or newer is required.])])

pkg_modules="gmodule-2.0 >= 2.0.0"
PKG_CHECK_MODULES(GMODULE, [$pkg_modules])

LIBS="$LIBS"

PACKAGE_CFLAGS="$GMODULE_CFLAGS"
AC_SUBST(PACKAGE_CFLAGS)
CPPFLAGS="$CPPFLAGS $PACKAGE_CFLAGS"

if test "\"\"" = "$IRSSI_HEADER_DIR"; then
    AC_MSG_WARN([The --with-irssi-headers option was not set. You may need to set extra CPPFLAGS to find all the irssi headers])
else
   # A hack to set some odd include flags here as irssi has an unconventional way of installing it's header files
   CPPFLAGS="$CPPFLAGS -I$IRSSI_HEADER_DIR -I$IRSSI_HEADER_DIR/irssi -I$IRSSI_HEADER_DIR/irssi/src/fe-common/core"
   CPPFLAGS="$CPPFLAGS -I$IRSSI_HEADER_DIR/irssi/src/core -I$IRSSI_HEADER_DIR/irssi/src"
fi
# Check for the two required irssi headers initially 
AC_CHECK_HEADERS([irssi/irssi-config.h irssi/irssi-version.h], [] ,[AC_MSG_ERROR([Irssi header files are required])], [])
# Check for the other irssi headers 
AC_CHECK_HEADER([irssi/src/common.h], [] ,[AC_MSG_ERROR([Irssi Header files are needed])], 
   [ #define HAVE_CONFIG_H
     #include <irssi/irssi-config.h>
     #include <irssi/irssi-version.h>
   ])
# There are more irssi headers that could be checked but checking for them here would be redundant

LT_INIT

# Test what compile flags are appropriate for the given platform/compiler
CC_CHECK_CFLAG_APPEND([-Wall])
CC_CHECK_CFLAG_APPEND([-g])
CC_CHECK_CFLAG_APPEND([-fno-strict-aliasing])
CC_CHECK_CFLAG_APPEND([-fstack-protector-all])
CC_CHECK_CFLAG_APPEND([-fPIE])
CC_CHECK_CFLAG_APPEND([-pie])
CC_CHECK_CFLAG_APPEND([-Werror=format-security])
CC_CHECK_CFLAG_APPEND([-v])
CFLAGS="$CFLAGS -D_FORTIFY_SOURCE=2" 
CC_CHECK_LDFLAGS([-Wl,-z,relro,-z,now])

DEFAULT_INCLUDES="-I\$(top_srcdir) -I\$(top_builddir) -I\$(top_builddir)/src -I\$(top_builddir)/include"
AC_SUBST(DEFAULT_INCLUDES)

# Irssi searches for modules in $HOME/.irssi/modules and  $(libdir)/irssi/modules
# where the value of $(libdir) may depend on your distribution, architecture,
# and whether irssi was installed from package or from source.
# The hardcoded value should work for most systems but you need to override it for
# e.g. x86_64 Fedora.
AC_ARG_WITH([irssi-module-dir],
	AS_HELP_STRING([--with-irssi-module-dir=DIR], [Irssi module directory]),
	[IRSSI_MODULE_DIR="$withval"],
	[IRSSI_MODULE_DIR='${libdir}/irssi/modules'])

AC_SUBST(IRSSI_MODULE_DIR)

AC_CONFIG_FILES([
	Makefile
	help/Makefile
	src/Makefile
   irssi-otr.spec
])

# Check for common functions
AC_CHECK_FUNCS([strndup])

AC_OUTPUT

#
# Mini-report on what will be built
#
AS_ECHO()

# Target architecture we're building for
target_arch=$host_cpu
[
for f in $CFLAGS; do
        if test $f = "-m32"; then
			target_arch="32-bit"
        elif test $f = "-m64"; then
			target_arch="64-bit"
        fi
done
]
AS_ECHO_N("Target architecture: ")
AS_ECHO($target_arch)

# Print the bindir and libdir this `make install' will install into.
AS_ECHO()
AS_ECHO_N("Module will be installed in: ")
AS_ECHO("`eval eval echo $IRSSI_MODULE_DIR`")
AS_ECHO()
