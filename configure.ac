# Copyright 2018 Gary Greene
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

AC_INIT([motherd], [0.1], [https://github.com/AltimatOS/motherd/issues])

AC_PREREQ([2.59])
AC_CONFIG_AUX_DIR(configlib)
AC_CANONICAL_HOST
AM_INIT_AUTOMAKE(1.9)

AC_PATH_PROG(PERL, perl, no)
AC_PATH_PROG(SED, sed, no)
AC_PATH_PROG(RM, rm, no)
AC_PATH_PROG(RMDIR, rmdir, no)
AC_PATH_PROG(FIND, find, no)
AC_PATH_PROG(TELNET, telnet, no)
AC_PATH_PROG(SSH, ssh, no)

AC_ARG_ENABLE(pkgonly, [AC_HELP_STRING([--enable-pkgonly], [Skip all checking])])
AC_SUBST(enable_pkgonly)
        
perllibdirs="\'\${perllibdir}\'"

PERLOPTS=
AC_ARG_VAR(PERLINC,  [[] Additional space-separated Perl library paths])
if test ! -z "$PERLINC"; then
    for d in $PERLINC; do
        PERLOPTS="${PERLOPTS} -I${d}"
        perllibdirs=${perllibdirs}"\,\'"${d}"\'"
    done
fi

AC_SUBST(perllibdirs, [${perllibdirs}])

# Check the necessary Perl modules
if test "$enable_pkgonly" != yes; then
    for module in 'Config::Tiny' 'Config::Any' 'Log::Handler' \
        'Expect' 'Date::Format' 'XML::LibXML' 'JSON' 'DBI' \
        'Net::SNMP' 'Digest::MD5'
    do
        AC_MSG_CHECKING([presence of $module])
        if ${PERL} ${PERLOPTS} -e 'use '$module 2>/dev/null; then
            AC_MSG_RESULT([Ok])
        else
            AC_MSG_ERROR([Perl cannot find $module]);
        fi
    done
fi

AC_ARG_VAR(mansec_usercmd, [[1] User commands man section])
if test -z "$mansec_usercmd"; then
    mansec_usercmd='1'
fi

AC_ARG_VAR(mansec_misc, [[7] Miscellaneous man section])
if test -z "$mansec_misc"; then
    mansec_misc='7'
fi

AC_ARG_VAR(perllibdir,  [[LIBDIR/PACKAGE/perl] Gerty Perl libraries])
if test -z "$perllibdir"; then
    perllibdir='${libdir}/${PACKAGE}/perl'
fi

AC_ARG_VAR(pkgdatadir, [[DATADIR/PACKAGE] Gerty data directory])
if test -z "$pkgdatadir"; then
    pkgdatadir='${datadir}/${PACKAGE}'
fi

AC_ARG_VAR(plugconfdir, [[PKGDATADIR/plugconf] Plugin configuration directory])
if test -z "$plugconfdir"; then
    plugconfdir='${pkgdatadir}/plugconf'
fi

AC_ARG_VAR(devclassdir, [[PKGDATADIR/devclasses] Device classes directory])
if test -z "$devclassdir"; then
    devclassdir='${pkgdatadir}/devclasses'
fi

AC_CONFIG_FILES([Makefile sbin/Makefile])
AC_CONFIG_FILES([lib/Makefile share/Makefile])

AC_SUBST(VERSION)
AC_OUTPUT
