# Copyright (C) 2013-2015 Yubico AB
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, eithe version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.

AC_INIT([libu2f-host], [1.1.7], [yubico-devel@googlegroups.com])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_AUX_DIR([build-aux])

# http://www.gnu.org/s/libtool/manual/html_node/Updating-version-info.html
AC_SUBST(LT_CURRENT, 1)  # Interfaces removed:    CURRENT++, AGE=0, REVISION=0
AC_SUBST(LT_AGE, 1)      # Interfaces added:      CURRENT++, AGE++, REVISION=0
AC_SUBST(LT_REVISION, 7) # No interfaces changed:                   REVISION++

AM_INIT_AUTOMAKE([gnits dist-xz no-dist-gzip std-options -Wall])
AM_SILENT_RULES([yes])

AC_PROG_CC
gl_EARLY
AM_PROG_AR
LT_INIT([win32-dll])

GTK_DOC_CHECK(1.1)
AM_MISSING_PROG(HELP2MAN, help2man, $missing_dir)
AM_MISSING_PROG(HELP2ADOC, help2adoc, $missing_dir)

PKG_CHECK_MODULES([LIBJSON], [json-c], [], [
  PKG_CHECK_MODULES([LIBJSON], [json])])

# Check for json_object_to_json_string_ext, needed for pretty printing.
am_save_CFLAGS="$CFLAGS"
am_save_LIBS="$LIBS"
CFLAGS="$CFLAGS $LIBJESON_CFLAGS"
LIBS="$LIBS $LIBJSON_LIBS"
AC_CHECK_FUNCS([json_object_to_json_string_ext])
AC_CHECK_FUNCS([json_object_object_get_ex])
CFLAGS=$am_save_CFLAGS
LIBS=$am_save_LIBS

PKG_CHECK_MODULES([HIDAPI], [hidapi], [], [
  PKG_CHECK_MODULES([HIDAPI], [hidapi-hidraw])])

gl_INIT

AC_ARG_ENABLE([gcc-warnings],
  [AS_HELP_STRING([--enable-gcc-warnings],
		  [turn on lots of GCC warnings (for developers)])],
  [case $enableval in
     yes|no) ;;
     *)      AC_MSG_ERROR([bad value $enableval for gcc-warnings option]) ;;
   esac
   gl_gcc_warnings=$enableval],
  [gl_gcc_warnings=no]
)

if test "$gl_gcc_warnings" = yes; then
  nw="$nw -Wsystem-headers"         # Don't let system headers trigger warnings
  nw="$nw -Wpadded"                 # Struct's arenot padded
  nw="$nw -Wc++-compat"             # We don't care strongly about C++ compilers
  nw="$nw -Wtraditional"            # Warns on #elif which we use often
  nw="$nw -Wtraditional-conversion" # Too many warnings for now
  nw="$nw -Wconversion"             # Too many warnings for now
  nw="$nw -Wsuggest-attribute=pure" # Is it worth using attributes?
  nw="$nw -Wsuggest-attribute=const" # Is it worth using attributes?

  gl_MANYWARN_ALL_GCC([ws])
  gl_MANYWARN_COMPLEMENT(ws, [$ws], [$nw])
  for w in $ws; do
    gl_WARN_ADD([$w])
  done

  gl_WARN_ADD([-fdiagnostics-show-option])
fi

AC_SUBST([U2FH_VERSION_MAJOR],
	 `echo $PACKAGE_VERSION | sed 's/\(.*\)\..*\..*/\1/g'`)
AC_SUBST([U2FH_VERSION_MINOR],
	 `echo $PACKAGE_VERSION | sed 's/.*\.\(.*\)\..*/\1/g'`)
AC_SUBST([U2FH_VERSION_PATCH],
	 `echo $PACKAGE_VERSION | sed 's/.*\..*\.\(.*\)/\1/g'`)
AC_SUBST([U2FH_VERSION_NUMBER],
	 `printf "0x%02x%02x%02x" $U2FH_VERSION_MAJOR \
		 $U2FH_VERSION_MINOR $U2FH_VERSION_PATCH`)

AC_ARG_WITH([udevrulesdir],
  AS_HELP_STRING([--with-udevrulesdir=DIR], [Install udev rules into this directory]),
  [], [])
AC_SUBST([udevrulesdir], [$with_udevrulesdir])

PKG_CHECK_MODULES([UDEV], [udev >= 188],
                  udevrulesfile=70-u2f.rules,
                  udevrulesfile=70-old-u2f.rules,
                  )
AC_SUBST([udevrulesfile], [$udevrulesfile])

AC_CONFIG_FILES([
  Makefile
  gl/Makefile
  gtk-doc/Makefile
  src/Makefile
  tests/Makefile
  u2f-host/Makefile
  u2f-host/u2f-host-version.h
  u2f-host/u2f-host.pc
])
AC_OUTPUT

AC_MSG_NOTICE([summary of build options:

  version:          ${VERSION} shared $LT_CURRENT:$LT_REVISION:$LT_AGE major $U2FH_VERSION_MAJOR minor $U2FH_VERSION_MINOR patch $U2FH_VERSION_PATCH number $U2FH_VERSION_NUMBER
  Host type:        ${host}
  Install prefix:   ${prefix}
  udev rules:       ${with_udevrulesdir:-N/A}
  udev rulefile:    ${udevrulesfile}
  Compiler:         ${CC}
  Shared library:   ${enable_shared}
  Static library:   ${enable_static}
  JSON CFLAGS:      $LIBJSON_CFLAGS
  JSON LIBS:        $LIBJSON_LIBS
  HIDAPI CFLAGS:    $HIDAPI_CFLAGS
  HIDAPI LIBS:      $HIDAPI_LIBS
])
