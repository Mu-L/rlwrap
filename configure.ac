dnl  configuration script for rlwrap
dnl  Process this file with autoconf to produce configure.
dnl
dnl  Copyright (C) 2000-2017 Hans Lub hanslub42@gmail.com
dnl
dnl  This file is part of rlwrap
dnl
dnl  rlwrap is free software; you can redistribute it and/or modify it
dnl  under the terms of the GNU General Public License as published by
dnl  the Free Software Foundation; either version 2, or (at your
dnl  option) any later version.
dnl
dnl  rlwrap is distributed in the hope that it will be useful, but
dnl  WITHOUT ANY WARRANTY; without even the implied warranty of
dnl  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
dnl  General Public License for more details.
dnl
dnl  You should have received a copy of the GNU General Public License
dnl  along with GUILE; see the file COPYING.  If not, write to the
dnl  Free Software Foundation, Inc., 59 Temple Place - Suite 330,
dnl  Boston, MA 02111-1307, USA.




AC_INIT(rlwrap,0.46.2)

AC_PREREQ(2.69)


AC_CONFIG_AUX_DIR(tools)
AC_CANONICAL_HOST
AC_CONFIG_SRCDIR(src/main.c)
AM_INIT_AUTOMAKE([foreign])

AC_PROG_INSTALL
AC_CONFIG_HEADERS([config.h])
AC_PROG_MAKE_SET
AM_SANITY_CHECK


# option parsing for optional features
opt_debug=no
opt_homegrown_redisplay=no
opt_spy_on_readline=yes
opt_multibyte_aware=yes
opt_proc_mountpoint=/proc

AC_ARG_ENABLE(debug,
    AS_HELP_STRING([--enable-debug], [enable debugging (default=NO)]),
    opt_debug=$enableval)

AC_ARG_ENABLE(pedantic-ansi,
    AS_HELP_STRING([--enable-pedantic-ansi], [compile with -ansi and -pedantic (default=NO)]),
    opt_pedantic_ansi=$enableval)


AC_ARG_ENABLE(homegrown-redisplay,
    AS_HELP_STRING([--enable-homegrown-redisplay], [try this only if rlwrap messes up the prompt (default=NO)]),
    opt_homegrown_redisplay=$enableval)



AC_ARG_ENABLE(multibyte-aware,
    AS_HELP_STRING([--enable-multibyte-aware], [make rlwrap aware of multibyte characters (default=yes)]),
    opt_multibyte_aware=$enableval)


AC_ARG_ENABLE(spy-on-readline,
    AS_HELP_STRING([--enable-spy-on-readline], [use private readline variable to keep screen tidy when resizing terminal (default=yes)]),
    opt_spy_on_readline=$enableval)

AC_ARG_ENABLE(proc-mountpoint,                                                                                                                    
     AS_HELP_STRING([--enable-proc-mountpoint],                                                                                  
       [specify mountpoint for Linux-style procfs (used for determination of command's PWD) (default=/proc)]), # '                               
     opt_proc_mountpoint=$enableval) 


if test x$opt_debug = xyes ; then
    AC_DEFINE(DEBUG,1 ,Define to 1 to get debugging info)
    test -z "$CFLAGS" || AC_MSG_WARN([Because CFLAGS is set, configure will not add -g and -Wall to them - do it yourself if necessary])
    test -z "$CFLAGS" && AC_SUBST(CFLAGS,"-O0 -g -Wall") # compile for debugging, and  give plenty of warnings
fi

if test x$opt_pedantic_ansi = xyes ; then
    AC_SUBST(CFLAGS,"$CFLAGS -ansi -Wimplicit-function-declaration -Wimplicit -pedantic -Wextra")
fi

# wait with defining SPY_ON_READLINE until we know we can use _rl_horizontal_scroll_mode (cf below)


AC_USE_SYSTEM_EXTENSIONS

test x$opt_homegrown_redisplay = xyes && AC_DEFINE(HOMEGROWN_REDISPLAY,1 ,[Define to 1 to use homegrown_redisplay()])

test x$opt_multibyte_aware  = xyes && AC_DEFINE(MULTIBYTE_AWARE,1 ,Define to 1 to be aware of  wide chars in prompts)

AC_CONFIG_FILES([Makefile filters/Makefile doc/Makefile src/Makefile doc/rlwrap.man]) 

# Checks for programs.


AC_PROG_CC
AC_PROG_CPP

AC_PATH_PROG(PERL,perl)

AC_CHECK_PROG(STRIP,strip,strip,true)

# Checks for header files.
AC_HEADER_SYS_WAIT
AC_CHECK_HEADERS([errno.h fcntl.h libgen.h libutil.h stdlib.h string.h sched.h sys/file.h sys/ioctl.h sys/wait.h sys/resource.h stddef.h ])
AC_CHECK_HEADERS([termios.h unistd.h stdint.h time.h sys/time.h getopt.h regex.h curses.h stropts.h termcap.h util.h stdarg.h])

AC_CHECK_HEADERS([ term.h  ncurses/term.h], , ,
    [#ifdef HAVE_CURSES_H
     #include <curses.h>
     #endif])


AC_CHECK_DECLS(PROC_PIDVNODEPATHINFO, , , [#include <libproc.h>])

have_freebsd_libprocstat=yes
AC_CHECK_DECLS([procstat_open_sysctl, procstat_getprocs, procstat_getfiles, STAILQ_FOREACH] ,[] ,[have_freebsd_libprocstat=no],
      [#include <sys/param.h>
       #include <sys/queue.h>
       #include <sys/socket.h>
       #include <libprocstat.h>
       #include <sys/sysctl.h>])

if test x$have_freebsd_libprocstat = xyes ; then
   AC_DEFINE(HAVE_FREEBSD_LIBPROCSTAT,1 ,[Define to 1 to use FREEBSD libprocstat])
   PROCSTATLIB='-lprocstat'
fi

# Found this in configure.ac for 'top':
# The third argument to tputs is a putc-like function that takes an
# argument.  On most systems that argument is an int, but on some (you
# know who you are, Solaris!) it is a char.  Determine which:

_savedwerror_flag=$ac_c_werror_flag 
ac_c_werror_flag=yes # make $CC fail after warning
AC_MSG_CHECKING([argument type of tputs putc function])
AC_COMPILE_IFELSE( [AC_LANG_PROGRAM(
             [ #ifdef HAVE_CURSES_H
               #  include <curses.h>
               #  ifdef HAVE_TERM_H
               #    include <term.h>
               #  else 
               #    ifdef HAVE_NCURSES_TERM_H /* cygwin? AIX? */
               #      include <ncurses/term.h>
               #    endif
               #  endif
               #else
               #  ifdef HAVE_TERMCAP_H
               #     include <termcap.h>
               #  endif
               #endif
               int f(char i) { }],
             [tputs("a", 1, f);])],
             [ac_cv_type_tputs_putc="char"],
             [ac_cv_type_tputs_putc="int"])
AC_MSG_RESULT($ac_cv_type_tputs_putc)
AC_DEFINE_UNQUOTED(TPUTS_PUTC_ARGTYPE, $ac_cv_type_tputs_putc,
                   [Define as the type for the argument to the putc function of tputs ('int' or 'char')])
ac_c_werror_flag=$_savedwerror_flag       

# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_TYPE_PID_T



# Let groff -Tman know whether we have long options
# Up to 0.45, rlwrap would test if getopt() understands option strings like "a:b::c" with double colons
# I now tend to think that that only happens if (POSIX) getopt() is implemented using (GNU) getopt_long(), so this test
# is actually superfluous. I leave GETOPT_GROKS_OPTIONAL_ARGS alone for the moment, in case some getopt_long() 
# implementation that doesnt grok double colons turns up:


AC_CHECK_FUNC([getopt_long],
  [AC_SUBST(HAVE_LONG_OPTS,[yes])
   AC_DEFINE(GETOPT_GROKS_OPTIONAL_ARGS, 1, Define if your getopt() correctly understands double colons in the option string)
   AC_SUBST(HAVE_OPTIONAL_ARGS,yes)],
  [AC_SUBST(HAVE_LONG_OPTS,[no])]
   AC_SUBST(HAVE_OPTIONAL_ARGS,no))

AC_CHECK_FUNCS(basename dirname flock getopt_long isastream  mkstemps pselect putenv readlink sched_yield )
AC_CHECK_FUNCS(setenv setitimer setsid setrlimit sigaction snprintf strlcpy strlcat strnlen system)

# Try to determine the kind of pty support on this system
# This is not so easy and may guess wrong; if this happens:
#    - edit config.h by hand (and keep a copy of it somewhere as configure will overwrite it), and 
#    - *please* notify the author! (hanslub42@gmail.com) 
# Most of the code below is taken from rxvt-2.7.10 (Copyright (c) 1999-2001
# Geoff Wing <gcw@pobox.com>)

# In the newest FreeBSD, we need openpty, which exists in libutil
# NB: The rxvt configure script warns against this. Why?


AC_CHECK_LIB(util, openpty)
AC_CHECK_FUNCS(openpty getpty grantpt unlockpt)



# AC_CACHE_CHECK(for getpt, ptyttylib_cv_func_getpt,
# [AC_TRY_LINK([#define _GNU_SOURCE
# #ifdef HAVE_STDLIB_H
# # include <stdlib.h>
# #endif],
#   [(void)getpt();],
#   ptyttylib_cv_func_getpt=yes, ptyttylib_cv_func_getpt=no)])



AC_CACHE_CHECK(for getpt, ptyttylib_cv_func_getpt,
[AC_LINK_IFELSE(
  [AC_LANG_PROGRAM([[
  #define _GNU_SOURCE
  #ifdef HAVE_STDLIB_H
  # include <stdlib.h>
  #endif
  ]],
  [[(void)getpt();]],
  ptyttylib_cv_func_getpt=yes, ptyttylib_cv_func_getpt=no)])])


if test x$ptyttylib_cv_func_getpt = xyes; then
  AC_DEFINE(HAVE_GETPT, 1, Define if you have _GNU_SOURCE getpt() )
fi


AC_CACHE_CHECK(for pty/tty type, ptyttylib_cv_ptys,
[if test x$ac_cv_func_openpty = xyes; then
    ptyttylib_cv_ptys=OPENPTY
    AC_CHECK_HEADERS([pty.h])    
 else if test x$ac_cv_func__getpty = xyes; then
    ptyttylib_cv_ptys=SGI4
 else if test -c /dev/ttyp20 -a ! -c /dev/ptmx; then 
   ptyttylib_cv_ptys=SCO
 else if test -c /dev/ptym/clone; then
    ptyttylib_cv_ptys=HPUX
 else if test x$ptyttylib_cv_func_getpt = xyes; then
    ptyttylib_cv_ptys=GLIBC
 else if test -c /dev/ptc -a -c /dev/pts; then
    ptyttylib_cv_ptys=PTC
 else if test -c /dev/ptc -a -d /dev/pts; then
    ptyttylib_cv_ptys=PTC
 else if test -c /dev/ptmx -a -c /dev/pts/0; then
    ptyttylib_cv_ptys=STREAMS
 else case "$host_os" in
         *cygwin*) ptyttylib_cv_ptys=STREAMS ;;
         *)        ptyttylib_cv_ptys=PREHISTORIC ;;
      esac
fi
fi
fi
fi
fi
fi
fi
fi
])

if test x$ptyttylib_cv_ptys = xOPENPTY; then
  AC_DEFINE(PTYS_ARE_OPENPTY, 1, Define for this pty type)
fi
if test x$ptyttylib_cv_ptys = xSCO; then
  AC_DEFINE(PTYS_ARE_NUMERIC, 1, Define for this pty type)
fi
if test x$ptyttylib_cv_ptys = xSTREAMS; then
  AC_DEFINE(PTYS_ARE_PTMX, 1, Define for this pty type)
fi
if test x$ptyttylib_cv_ptys = xPTC; then
  AC_DEFINE(PTYS_ARE_PTC, 1, Define for this pty type)
fi
if test x$ptyttylib_cv_ptys = xSGI4; then
  AC_DEFINE(PTYS_ARE__GETPTY, 1, Define for this pty type)
fi
if test x$ptyttylib_cv_ptys = xCONVEX; then
  AC_DEFINE(PTYS_ARE_GETPTY, 1, Define for this pty type)
fi
if test x$ptyttylib_cv_ptys = xGLIBC; then
  AC_DEFINE(PTYS_ARE_GETPT, 1, Define for this pty type)
fi
if test x$ptyttylib_cv_ptys = xHPUX; then
  AC_DEFINE(PTYS_ARE_CLONE, 1, Define for this pty type)
fi
if test x$ptyttylib_cv_ptys = xPREHISTORIC -o x$ptyttylib_cv_ptys = xHPUX -o x$ptyttylib_cv_ptys = xGLIBC; then
  AC_DEFINE(PTYS_ARE_SEARCHED, 1, Define for this pty type)
fi




AC_MSG_NOTICE(checking for pty ranges)
ptys=`echo /dev/pty??`
pch1=`echo $ptys | tr ' ' '\012' | sed -e 's/^.*\(.\).$/\1/g' | sort -u | tr -d '\012'`
pch2=`echo $ptys | tr ' ' '\012' | sed -e 's/^.*\(.\)$/\1/g' | sort -u | tr -d '\012'`
if test x$pch1 != x; then
  AC_DEFINE_UNQUOTED(PTYCHAR1, "$pch1", Define for first char in devptyXX)
fi
if test x$pch2 != x; then
  AC_DEFINE_UNQUOTED(PTYCHAR2, "$pch2", Define for second char in devptyXX)
fi



# Checks whether readline needs additional libs.


AC_CHECK_FUNC(tgetent,
   [READLINE_SUPPLIB=],
     [AC_CHECK_LIB(tinfo, tgetent,
     [READLINE_SUPPLIB=-ltinfo],
       [AC_CHECK_LIB(curses, tgetent,
         [READLINE_SUPPLIB=-lcurses],
         [AC_CHECK_LIB(ncurses, tgetent,
            [READLINE_SUPPLIB=-lncurses],
            [AC_CHECK_LIB(termcap, tgetent,
               [READLINE_SUPPLIB=-ltermcap],
               [AC_MSG_ERROR(No termcap nor curses library found)])
            ])
         ])
      ])
   ])




AC_CHECK_LIB(readline,readline,[READLINELIB=-lreadline],
	AC_MSG_ERROR([
You need the GNU readline library(ftp://ftp.gnu.org/gnu/readline/ ) to build
this program!
]), [$READLINE_SUPPLIB])

LIBS="$LIBS $PROCSTATLIB $READLINELIB $READLINE_SUPPLIB"

AC_CHECK_FUNCS(tigetnum)



AC_CHECK_HEADER([readline/readline.h],[],AC_MSG_ERROR([readline.h not found - you may need to install a readline development package]))

# AC_EGREP_RL_HEADER_AND_CHECK_FUNC(function_or_variable, [code using function_or_variable],  HAVE_FUNCTION_OR_VARIABLE)
# This macro looks for <function_or_variable>, both as a declaration in header.h, and as a symbol in $LIBS  
# ------------------------------------------------------------------------------------------

# @@ Haal hier TRY_LINK uit!
# AC_DEFUN([AC_EGREP_RL_HEADER_AND_CHECK_FUNC],
#          [ AC_MSG_CHECKING([whether your readline headers and library know about $1])
#            AC_EGREP_HEADER($1, [readline/readline.h], in_header=1, in_header=0)
#            AC_TRY_LINK([ #include <stdio.h>
#                          #include "readline/readline.h" ],
#                          [$2], in_lib=1, in_lib=0)
#            if test x$in_lib = x1 -a x$in_header = x1 ; then
#               AC_DEFINE($3, 1 ,[Define to 1 if your readline lib has $1])
#               last_check=yes
#            else
#               last_check=no
#            fi
#            remark=
#            if test x$in_lib = x0 -a x$in_header = x1 ; then
#                remark=" (only in headers, not in lib. Hmmmmm....)"
#            fi
#            AC_MSG_RESULT([$last_check$remark])
#            ])



AC_DEFUN([AC_EGREP_RL_HEADER_AND_CHECK_FUNC],
         [ AC_MSG_CHECKING([whether your readline headers and library know about $1])
           AC_EGREP_HEADER($1, [readline/readline.h], in_header=1, in_header=0)

           AC_LINK_IFELSE(
              [AC_LANG_PROGRAM([[ #include <stdio.h>
                                  #include "readline/readline.h"
                               ]], [[$2]])], in_lib=1, in_lib=0)
           if test x$in_lib = x1 -a x$in_header = x1 ; then
              AC_DEFINE($3, 1 ,[Define to 1 if your readline lib has $1])
              last_check=yes
           else
              last_check=no
           fi
           remark=
           if test x$in_lib = x0 -a x$in_header = x1 ; then
               remark=" (only in headers, not in lib. Hmmmmm....)"
           fi
           AC_MSG_RESULT([$last_check$remark])
           ])


AC_DEFUN([AC_MY_ERROR],
[AC_MSG_ERROR([
           
********************************************************************************
$1
********************************************************************************
])])

AC_EGREP_RL_HEADER_AND_CHECK_FUNC([rl_set_screen_size], [rl_set_screen_size(25,80)], [HAVE_RL_SET_SCREEN_SIZE])

test $last_check = no && AC_MY_ERROR([You need a more recent version (at least 4.2) of the readline
library (ftp://ftp.gnu.org/gnu/readline/) to build this program!])


AC_EGREP_RL_HEADER_AND_CHECK_FUNC([rl_basic_quote_characters], [printf("%s", rl_basic_quote_characters)], [HAVE_RL_BASIC_QUOTE_CHARS]) 

test $last_check = no && AC_MY_ERROR([[The readline library that I found is incomplete, probably just a wrapper
around libedit (like on Mac OS X) Please install GNU Readline and re-configure
with the correct CFLAGS and LDFLAGS (cf. the instructions in INSTALL)]])

AC_EGREP_RL_HEADER_AND_CHECK_FUNC([rl_variable_value], [rl_variable_value("blah")], [HAVE_RL_VARIABLE_VALUE]) 

AC_EGREP_RL_HEADER_AND_CHECK_FUNC([rl_readline_version],[printf("%d",rl_readline_version)], [HAVE_RL_READLINE_VERSION]) 

AC_EGREP_RL_HEADER_AND_CHECK_FUNC([rl_executing_keyseq],[printf("%s", rl_executing_keyseq)], [HAVE_RL_EXECUTING_KEYSEQ]) 

# rlwrap tries to read a global (but private) readline variable _rl_horizontal_scroll_mode if the the option spy-on-realine is enabled
# Depending on the linker (or linker options like gcc's -fvisibility=xxx) it may or may not be visible:

AC_MSG_CHECKING([whether the private symbol _rl_horizontal_scroll_mode is visble in your readline libs])
# AC_TRY_LINK([#include <stdio.h>
#              extern int _rl_horizontal_scroll_mode;],
#              [printf("%d", _rl_horizontal_scroll_mode)],
#              found_rl_horiz=yes, found_rl_horiz=no)

AC_LINK_IFELSE(
           [AC_LANG_PROGRAM( [[
              #include <stdio.h>
             extern int _rl_horizontal_scroll_mode;]],
             [[printf("%d", _rl_horizontal_scroll_mode)]])],
             found_rl_horiz=yes, found_rl_horiz=no)


AC_MSG_RESULT($found_rl_horiz)

if test $opt_spy_on_readline = yes  -a $found_rl_horiz = no ; then
  AC_MSG_WARN([I can't find _rl_horizontal_scroll_mode: disabling spy-on-readline now])
  opt_spy_on_readline=no
fi

test x$opt_spy_on_readline = xyes && AC_DEFINE(SPY_ON_READLINE,1 ,Define to 1 to use private _rl_horizontal_scroll_mode)



# check for existence of myself under $opt_proc_mountpoint/$$/cwd If the user specified --disable-proc-mountpoint we'll still do these tests
# but they wil fail unless /no is a procfs mountpoint ;-)

echo "Will rlwrap find command's working directory under ${opt_proc_mountpoint}/<commands pid>/cwd? let's see..."
if test x$cross_compiling = xyes; then
   AC_DEFINE(HAVE_PROC_PID_CWD, 0, [Define to 1 if <opt_proc_mountpoint>/<pid>/cwd is a link to working directory of process <pid>])
   AC_SUBST(HAVE_PROC_PID_CWD,[no])
   echo
   AC_MSG_WARN([This test doesn't work when cross-compiling. Edit HAVE_PROC_PID_CWD in config.h by hand. Guessing: ])   
else 
   AC_CHECK_FILES($opt_proc_mountpoint/$$/cwd/configure.ac,                                                                                      
              [AC_DEFINE(HAVE_PROC_PID_CWD, 1 ,[Define to 1 if <opt_proc_mountpoint>/<pid>/cwd is a link to working directory of process <pid>])]
              ,[])
fi
#AC_MSG_RESULT([$HAVE_PROC_PID_CWD])


AC_DEFINE_UNQUOTED(PROC_MOUNTPOINT, "[$opt_proc_mountpoint]", [Define to location of Linux-style procfs mountpoint if provided and valid])


AC_MSG_CHECKING([whether we can find command line under <opt_proc_mountpoint>/<pid>/cmdline and mirror it by overwriting our own *argv (this may take a few secs)])
# We won't exactly do that, but instead write a test program that writes a random string to *argv (making sure it is longer than argv[0])
# and then verify that we can see the same string under <opt_proc_mountpoint>/<pid>/cmdline
# If so, we enable the --mirror-args option
random_nonsense1="fymaytskjqhjeeimtmjqvqbnrnfutgnkehkmqqealklsoswyepxugcckimqdwbwfqdhdsdxfxiagqbnm"
random_nonsense2="iraliuuklajcorjfunavdsytdiydjaoaeauiapidtoxogfplrxfmfcmajikwenafwcownmoodxjljhsv"

AC_DEFINE_UNQUOTED(RANDOM_NONSENSE2, "$random_nonsense2", nonsense arg to overwrite conftest argv with)

if test $cross_compiling = "yes"; then
   AC_DEFINE(ENABLE_MIRROR_ARGS, 0, [Define if overwriting argv changes args visible in ps (1)])
   AC_MSG_WARN([This test doesn't work when cross-compiling. Edit ENABLE_MIRROR_ARGS  in config.h by hand. Guessing:])
   result="no"
else
   AC_LANG_CONFTEST(
        [AC_LANG_SOURCE([[#include <unistd.h>
                          #include <string.h>
                          int main(int argc, char **argv) {
                                       memcpy(*argv, "a\0" RANDOM_NONSENSE2, strlen(RANDOM_NONSENSE2) + 2);
                                       return(sleep(10));
                                   }
                         ]])])

  result="no"
  $CC -o conftest conftest.c  
  ./conftest $random_nonsense1 >/dev/null 2>&1 &
  CONFTEST_PID=$!
  for N in  1 2 3 ; do # retry a few times because of a possible  race (on 1-processor systems) where ./conftest has been forked by the shell, ...
                       # ... but not yet executed (which will make the following test fail)
    if grep $random_nonsense2 $opt_proc_mountpoint/$CONFTEST_PID/cmdline >/dev/null 2>&1; then
      AS_ECHO(["#define ENABLE_MIRROR_ARGS 1"]) >>confdefs.h
      result="yes"
      break
    fi
    sleep 1 # shorter sleep is not portable
  done
  kill $CONFTEST_PID
fi
AC_MSG_RESULT($result)

if test x$cross_compiling = xyes; then
  cat <<EOF


    **************************************************************************************
    * This configure script runs a few handwritten scripts and ad-hoc programs in order  *
    * to determine capabilities of the host (mostly around the /proc filesystem)         *
    * When cross-compiling, this makes no sense, as you can see above.                   *
    * Hence, after configuration, please set HAVE_PROC_PID_CWD,                          *
    * GETOPT_GROKS_OPTIONAL_ARGS and ENABLE_MIRROR_ARGS by hand (in config.h)            *
    **************************************************************************************

EOF
fi


echo
echo


AC_OUTPUT

cat <<EOF 

Now do:
    make (or gmake)  to build rlwrap 
    make check       for instructions how to test it
    make install     to install it
 
EOF


# Local Variables:
# mode: autoconf
# End:

