#  Copyright 2005 Adam Jackson.
#
#  Permission is hereby granted, free of charge, to any person obtaining a
#  copy of this software and associated documentation files (the "Software"),
#  to deal in the Software without restriction, including without limitation
#  on the rights to use, copy, modify, merge, publish, distribute, sub
#  license, and/or sell copies of the Software, and to permit persons to whom
#  the Software is furnished to do so, subject to the following conditions:
#
#  The above copyright notice and this permission notice (including the next
#  paragraph) shall be included in all copies or substantial portions of the
#  Software.
#
#  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#  FITNESS FOR A PARTICULAR PURPOSE AND NON-INFRINGEMENT.  IN NO EVENT SHALL
#  ADAM JACKSON BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
#  IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
#  CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#
# Process this file with autoconf to produce a configure script

# Initialize Autoconf
AC_PREREQ([2.60])
AC_INIT([xf86-input-synaptics],
        [1.10.0],
        [https://gitlab.freedesktop.org/xorg/driver/xf86-input-synaptics/-/issues],
        [xf86-input-synaptics])
AC_CONFIG_SRCDIR([Makefile.am])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIRS([m4])
AC_CONFIG_AUX_DIR(.)

# Initialize Automake
AM_INIT_AUTOMAKE([foreign dist-xz])

# Initialize libtool
LT_INIT([disable-static])

# Initialize X.Org macros 1.13 or later for XORG_ENABLE_UNIT_TESTS
m4_ifndef([XORG_MACROS_VERSION],
          [m4_fatal([must install xorg-macros 1.13 or later before running autoconf/autogen])])
XORG_MACROS_VERSION(1.13)
XORG_DEFAULT_OPTIONS
XORG_ENABLE_UNIT_TESTS

# Checks for libraries.
AC_CHECK_LIB([m], [rint])

# Checks for library functions
# We don't use timingsafe_memcmp, but check to quiet -Wredundant-decls warning
# from xorg/os.h which will otherwise redefine it.
AC_CHECK_FUNCS([timingsafe_memcmp])

# Store the list of server defined optional extensions in REQUIRED_MODULES
m4_ifndef([XORG_DRIVER_CHECK_EXT],
          [m4_fatal([must install xorg-server development files before running autoconf/autogen])])
XORG_DRIVER_CHECK_EXT(RANDR, randrproto)

# Obtain compiler/linker options for the Synaptics driver dependencies
PKG_CHECK_MODULES(XORG, [inputproto >= 2.1.99.3] [xorg-server >= 1.18] xproto inputproto $REQUIRED_MODULES)

# X Server SDK location is required to install Synaptics header files
# This location is also relayed in the xorg-synaptics.pc file
sdkdir=`$PKG_CONFIG --variable=sdkdir xorg-server`
AC_SUBST([sdkdir])

# -----------------------------------------------------------------------------
#			Configuration options
# -----------------------------------------------------------------------------
# Define a configure option for an alternate input module directory
AC_ARG_WITH(xorg-module-dir,
            AS_HELP_STRING([--with-xorg-module-dir=DIR],
                           [Default xorg module directory [[default=$libdir/xorg/modules]]]),
            [moduledir="$withval"],
            [moduledir="$libdir/xorg/modules"])
inputdir=${moduledir}/input
AC_SUBST(inputdir)

# Define a configure option for an alternate X Server configuration directory
sysconfigdir=`$PKG_CONFIG --variable=sysconfigdir xorg-server`
AC_ARG_WITH(xorg-conf-dir,
            AS_HELP_STRING([--with-xorg-conf-dir=DIR],
                           [Default xorg.conf.d directory [[default=from $PKG_CONFIG xorg-server]]]),
            [configdir="$withval"],
            [configdir="$sysconfigdir"])
AC_SUBST(configdir)
AM_CONDITIONAL(HAS_XORG_CONF_DIR, [test "x$sysconfigdir" != "x"])

# Define a configure option to enable code debugging
AC_ARG_ENABLE(debug, AS_HELP_STRING([--enable-debug],
                                    [Enable debugging (default: disabled)]),
                                    [DEBUGGING=$enableval], [DEBUGGING=no])
if test "x$DEBUGGING" = xyes; then
   AC_DEFINE(DEBUG, 1, [Enable debugging code])
fi
AM_CONDITIONAL(DEBUG, [test "x$DEBUGGING" = xyes])

# -----------------------------------------------------------------------------
#		Determine which backend, if any, to build
# -----------------------------------------------------------------------------
AC_MSG_CHECKING([which optional backends will be build])
case "${host}" in
*linux*)
	AC_MSG_RESULT([ps2comm alpscomm eventcomm])
	BUILD_PS2COMM="yes"
	BUILD_EVENTCOMM="yes"
	;;
*freebsd* | *netbsd* | *dragonfly*)
	AC_MSG_RESULT([ps2comm alpscomm psmcomm])
	BUILD_PS2COMM="yes"
	BUILD_PSMCOMM="yes"
	;;
*solaris* | *gnu*)
	AC_MSG_RESULT([ps2comm alpscomm])
	BUILD_PS2COMM="yes"
	;;
*)
	AC_MSG_ERROR([Cannot find suitable backends for this platform.])
	;;
esac
if test "x$BUILD_EVENTCOMM" = xyes; then
    AC_DEFINE(BUILD_EVENTCOMM, 1, [Optional backend eventcomm enabled])
    PKG_CHECK_MODULES(LIBEVDEV, [libevdev >= 0.4])
    SAVE_LIBS="$LIBS"
    LIBS="$LIBEVDEV_LIBS"
    AC_CHECK_LIB(evdev, libevdev_set_device_log_function,
                      [AC_DEFINE(HAVE_LIBEVDEV_DEVICE_LOG_FUNCS, 1,
                       [libevdev supports per-device log functions])],
                       [])
    LIBS="$SAVE_LIBS"
fi
if test "x$BUILD_PSMCOMM" = xyes; then
    AC_DEFINE(BUILD_PSMCOMM, 1, [Optional backend psmcomm enabled])
fi
if test "x$BUILD_PS2COMM" = xyes; then
    AC_DEFINE(BUILD_PS2COMM, 1, [Optional backend ps2comm and alpscomm enabled])
fi
AM_CONDITIONAL([BUILD_EVENTCOMM], [test "x${BUILD_EVENTCOMM}" = "xyes"])
AM_CONDITIONAL([BUILD_PSMCOMM], [test "x${BUILD_PSMCOMM}" = "xyes"])
AM_CONDITIONAL([BUILD_PS2COMM], [test "x${BUILD_PS2COMM}" = "xyes"])

# -----------------------------------------------------------------------------
#			Dependencies for synclient and syndaemon
# -----------------------------------------------------------------------------
# Obtain compiler/linker options for the Synaptics apps dependencies
PKG_CHECK_MODULES(XI, x11 inputproto [xi >= 1.2])

# The syndaemon program uses an optional XRecord extension implementation
# If libxtst >= 1.0.99 is installed, Cflags contains the path to record.h
# If recordproto < 1.13.99.1 is installed, Cflags contains the path to record.h
PKG_CHECK_MODULES(XTST, xtst recordproto, have_libxtst="yes", have_libxtst="no")
if test "x$have_libxtst" = "xyes" ; then
   # Header record.h may come from the xtst or recordproto package, or may be missing
   SAVE_CPPFLAGS="$CPPFLAGS"
   CPPFLAGS="$CPPFLAGS $XTST_CFLAGS"
   AC_CHECK_HEADERS([X11/extensions/record.h],,,[#include <X11/Xlib.h>])
   CPPFLAGS="$SAVE_CPPFLAGS"
fi
# -----------------------------------------------------------------------------

# Workaround overriding sdkdir to be able to create a tarball when user has no
# write permission in sdkdir. See DISTCHECK_CONFIGURE_FLAGS in Makefile.am
AC_ARG_WITH([sdkdir], [], [sdkdir="$withval"])

AC_CONFIG_FILES([Makefile
                src/Makefile
                man/Makefile
                tools/Makefile
                conf/Makefile
                include/Makefile
                xorg-synaptics.pc])
AC_OUTPUT

