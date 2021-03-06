#
# Kmscon - build configuration script
# Copyright (c) 2012 David Herrmann <dh.herrmann@googlemail.com>
#

AC_PREREQ(2.68)

AC_INIT([kmscon],
        [8],
        [http://bugs.freedesktop.org/enter_bug.cgi?product=kmscon],
        [kmscon],
        [http://www.freedesktop.org/wiki/Software/kmscon])
AC_CONFIG_SRCDIR([src/kmscon_main.c])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_HEADER(config.h)
AC_USE_SYSTEM_EXTENSIONS
AC_SYS_LARGEFILE
AC_CANONICAL_SYSTEM

AM_INIT_AUTOMAKE([foreign 1.11 subdir-objects dist-xz no-dist-gzip tar-pax -Wall -Werror -Wno-portability])
AM_SILENT_RULES([yes])

#
# Don't add a default "-g -O2" if CFLAGS wasn't specified. For debugging it is
# often more convenient to have "-g -O0". You can still override it by
# explicitly setting it on the command line.
#

: ${CFLAGS=""}

AC_PROG_CC
AX_PROG_CC_FOR_BUILD
AC_PROG_CC_C99
AM_PROG_CC_C_O
m4_ifdef([AM_PROG_AR], [AM_PROG_AR])
AC_PROG_SED
AC_PROG_MKDIR_P
AC_PROG_LN_S
AC_PROG_GREP
AC_PROG_AWK

LT_PREREQ(2.2)
LT_INIT

#
# Required pkg-config dependencies
#

PKG_CHECK_MODULES([XKBCOMMON], [xkbcommon >= 0.5.0])
AC_SUBST(XKBCOMMON_CFLAGS)
AC_SUBST(XKBCOMMON_LIBS)

PKG_CHECK_MODULES([TSM], [libtsm >= 4.0.0])
AC_SUBST(TSM_CFLAGS)
AC_SUBST(TSM_LIBS)

PKG_CHECK_MODULES([UDEV], [libudev >= 172])
AC_SUBST(UDEV_CFLAGS)
AC_SUBST(UDEV_LIBS)

PKG_CHECK_MODULES([DRM], [libdrm])
AC_SUBST(DRM_CFLAGS)
AC_SUBST(DRM_LIBS)

#
# pkg-config dependencies
# This unconditionally checks for all dependencies even if they are disabled. We
# later look whether all required depedencies are met and finish the
# configuration. We group similar packages into one logical group here to avoid
# having variables for each single library.
# This, however, makes ./configure output very unintuitive error messages if a
# package is not found so we must make sure we print more verbose messages
# ourself.
#

PKG_CHECK_MODULES([SYSTEMD], [libsystemd],
                  [have_systemd=yes], [have_systemd=no])
AC_SUBST(SYSTEMD_CFLAGS)
AC_SUBST(SYSTEMD_LIBS)

PKG_CHECK_MODULES([GBM], [gbm],
                  [have_gbm=yes], [have_gbm=no])
AC_SUBST(GBM_CFLAGS)
AC_SUBST(GBM_LIBS)

PKG_CHECK_MODULES([EGL], [egl],
                  [have_egl=yes], [have_egl=no])
AC_SUBST(EGL_CFLAGS)
AC_SUBST(EGL_LIBS)

PKG_CHECK_MODULES([GLES2], [glesv2],
                  [have_gles2=yes], [have_gles2=no])
AC_SUBST(GLES2_CFLAGS)
AC_SUBST(GLES2_LIBS)

PKG_CHECK_MODULES([PANGO], [pango pangoft2],
                  [have_pango=yes], [have_pango=no])
AC_SUBST(PANGO_CFLAGS)
AC_SUBST(PANGO_LIBS)

PKG_CHECK_MODULES([PIXMAN], [pixman-1],
                  [have_pixman=yes], [have_pixman=no])
AC_SUBST(PIXMAN_CFLAGS)
AC_SUBST(PIXMAN_LIBS)

#
# Parse arguments
# This parses all arguments that are given via "--enable-XY" or "--with-XY" and
# saves the results in local variables. This does not check dependencies or
# similar but does only parse the arguments.
#

# all
AC_MSG_CHECKING([whether user wants all])
AC_ARG_ENABLE([all],
              [AS_HELP_STRING([--enable-all],
                              [enable all options (used for debugging)])])
if test "x$enable_all" = "x" ; then
        enable_all="no"
fi
AC_MSG_RESULT([$enable_all])

# debug
AC_MSG_CHECKING([whether to build with debugging on])
AC_ARG_ENABLE([debug],
              [AS_HELP_STRING([--enable-debug],
                              [whether to build with debugging on])])
if test "x$enable_all" = "xyes" ; then
        enable_debug="yes"
elif test "x$enable_debug" = "x" ; then
        enable_debug="yes (default)"
fi
AC_MSG_RESULT([$enable_debug])

# optimizations
AC_MSG_CHECKING([whether to disable code optimizations])
AC_ARG_ENABLE([optimizations],
              [AS_HELP_STRING([--disable-optimizations],
                              [whether to disable code optimizations])])
if test "x$enable_all" = "xyes" ; then
        enable_optimizations="yes"
elif test "x$enable_optimizations" = "x" ; then
        enable_optimizations="yes (default)"
fi
AC_MSG_RESULT([$enable_optimizations])

# multi-seat
AC_MSG_CHECKING([whether user wants systemd for multi-seat support])
AC_ARG_ENABLE([multi-seat],
              [AS_HELP_STRING([--enable-multi-seat],
                              [enable multi-seat support with systemd])])
if test "x$enable_all" = "xyes" ; then
        enable_multi_seat="yes"
elif test "x$enable_multi_seat" = "x" ; then
        enable_multi_seat="yes (default)"
fi
AC_MSG_RESULT([$enable_multi_seat])

# video backends
AC_MSG_CHECKING([which video backends the user wants])
AC_ARG_WITH([video],
            [AS_HELP_STRING([--with-video],
              [specify list of optional video backends])],
            [],
            [with_video="default"])
enable_video_fbdev="no"
enable_video_drm2d="no"
enable_video_drm3d="no"
if test "x$enable_all" = "xyes" ; then
        enable_video_fbdev="yes"
        enable_video_drm2d="yes"
        enable_video_drm3d="yes"
        with_video="fbdev,drm2d,drm3d (all)"
elif test "x$with_video" = "xdefault" ; then
        enable_video_fbdev="yes (default)"
        enable_video_drm2d="yes (default)"
        enable_video_drm3d="yes (default)"
        with_video="fbdev,drm2d,drm3d (default)"
elif test ! "x$with_video" = "x" ; then
        SAVEIFS="$IFS"
        IFS=","
        for i in $with_video ; do
                if test "x$i" = "xfbdev" ; then
                        enable_video_fbdev="yes"
                elif test "x$i" = "xdrm2d" ; then
                        enable_video_drm2d="yes"
                elif test "x$i" = "xdrm3d" ; then
                        enable_video_drm3d="yes"
                else
                        IFS="$SAVEIFS"
                        AC_ERROR([Invalid video backend $i])
                fi
        done
        IFS="$SAVEIFS"
fi
AC_MSG_RESULT([$with_video])

# renderers
AC_MSG_CHECKING([which render backends the user wants])
AC_ARG_WITH([renderers],
            [AS_HELP_STRING([--with-renderers],
              [specify list of optional render backends])],
            [],
            [with_renderers="default"])
enable_renderer_bbulk="no"
enable_renderer_gltex="no"
enable_renderer_pixman="no"
if test "x$enable_all" = "xyes" ; then
        enable_renderer_bbulk="yes"
        enable_renderer_gltex="yes"
        enable_renderer_pixman="yes"
        with_renderers="bbulk,gltex,pixman (all)"
elif test "x$with_renderers" = "xdefault" ; then
        enable_renderer_bbulk="yes (default)"
        enable_renderer_gltex="yes (default)"
        enable_renderer_pixman="no (default)"
        with_renderers="bbulk,gltex (default)"
elif test ! "x$with_renderers" = "x" ; then
        SAVEIFS="$IFS"
        IFS=","
        for i in $with_renderers ; do
                if test "x$i" = "xbbulk" ; then
                        enable_renderer_bbulk="yes"
                elif test "x$i" = "xgltex" ; then
                        enable_renderer_gltex="yes"
                elif test "x$i" = "xpixman" ; then
                        enable_renderer_pixman="yes"
                else
                        IFS="$SAVEIFS"
                        AC_ERROR([Unknown renderer $i])
                fi
        done
        IFS="$SAVEIFS"
fi
AC_MSG_RESULT([$with_renderers])

# font backends
AC_MSG_CHECKING([which font backends the user wants])
AC_ARG_WITH([fonts],
            [AS_HELP_STRING([--with-fonts],
              [specify list of optional font backends])],
            [],
            [with_fonts="default"])
enable_font_unifont="no"
enable_font_pango="no"
if test "x$enable_all" = "xyes" ; then
        enable_font_unifont="yes"
        enable_font_pango="yes"
        with_fonts="unifont,pango (all)"
elif test "x$with_fonts" = "xdefault" ; then
        enable_font_unifont="yes (default)"
        enable_font_pango="yes (default)"
        with_fonts="unifont,pango (default)"
elif test ! "x$with_fonts" = "x" ; then
        SAVEIFS="$IFS"
        IFS=","
        for i in $with_fonts ; do
                if test "x$i" = "xunifont" ; then
                        enable_font_unifont="yes"
                elif test "x$i" = "xpango" ; then
                        enable_font_pango="yes"
                else
                        IFS="$SAVEIFS"
                        AC_ERROR([Unknown font backend $i])
                fi
        done
        IFS="$SAVEIFS"
fi
AC_MSG_RESULT([$with_fonts])

# kmscon sessions
AC_MSG_CHECKING([which sessions the user wants])
AC_ARG_WITH([sessions],
            [AS_HELP_STRING([--with-sessions],
              [specify list of optional sessions])],
            [],
            [with_sessions="default"])
enable_session_dummy="no"
enable_session_terminal="no"
if test "x$enable_all" = "xyes" ; then
        enable_session_dummy="yes"
        enable_session_terminal="yes"
        with_sessions="dummy,terminal (all)"
elif test "x$with_sessions" = "xdefault" ; then
        enable_session_dummy="yes (default)"
        enable_session_terminal="yes (default)"
        with_sessions="dummy,terminal (default)"
elif test ! "x$with_sessions" = "x" ; then
        SAVEIFS="$IFS"
        IFS=","
        for i in $with_sessions ; do
                if test "x$i" = "xdummy" ; then
                        enable_session_dummy="yes"
                elif test "x$i" = "xterminal" ; then
                        enable_session_terminal="yes"
                else
                        IFS="$SAVEIFS"
                        AC_ERROR([Unknown session type $i])
                fi
        done
        IFS="$SAVEIFS"
fi
AC_MSG_RESULT([$with_sessions])

#
# Check what can be built
# This checks each configuration option and tests whether all dependencies are
# met. This is done from bottom up to ensure the right order.
#

# debug
debug_avail=no
debug_missing=""
if test ! "x$enable_debug" = "xno" ; then
        debug_avail=yes
else
        debug_missing="enable-debug"
fi

# optimizations
optimizations_avail=no
optimizations_missing=""
if test ! "x$enable_optimizations" = "xno" ; then
        optimizations_avail=yes
else
        optimizations_missing="enable-optimizations"
fi

# video fbdev
video_fbdev_avail=no
video_fbdev_missing=""
if test ! "x$enable_video_fbdev" = "xno" ; then
        # TODO: check for kernel headers
        video_fbdev_avail=yes
else
        video_fbdev_missing="enable-video-fbdev"
fi

# video drm2d
video_drm2d_avail=no
video_drm2d_missing=""
if test ! "x$enable_video_drm2d" = "xno" ; then
        video_drm2d_avail=yes
else
        video_drm2d_missing="enable-video-drm2d"
fi

# video drm3d
video_drm3d_avail=no
video_drm3d_missing=""
if test ! "x$enable_video_drm3d" = "xno" ; then
        video_drm3d_avail=yes
        if test "x$have_gbm" = "xno" ; then
                video_drm3d_avail=no
                video_drm3d_missing="libgbm,$video_drm3d_missing"
        fi
        if test "x$have_egl" = "xno" ; then
                video_drm3d_avail=no
                video_drm3d_missing="libegl,$video_drm3d_missing"
        fi
        if test "x$have_gles2" = "xno" ; then
                video_drm3d_avail=no
                video_drm3d_missing="libgles2,$video_drm3d_missing"
        fi

        if test "x$video_drm3d_avail" = "xno" ; then
                if test "x$enable_video_drm3d" = "xyes" ; then
                        AC_ERROR([missing for drm3d video backend: $video_drm3d_missing])
                fi
        fi
else
        video_drm3d_missing="enable-video-drm3d"
fi

# multi-seat
multi_seat_avail=no
multi_seat_missing=""
if test ! "x$enable_multi_seat" = "xno" ; then
        multi_seat_avail=yes
        if test "x$have_systemd" = "xno" ; then
                multi_seat_avail=no
                multi_seat_missing="libsystemd"
        fi

        if test "x$multi_seat_avail" = "xno" ; then
                if test "x$enable_multi_seat" = "xyes" ; then
                        AC_ERROR([missing for multi-seat support: $multi_seat_missing])
                fi
        fi
else
        multi_seat_missing="enable-multi-seat"
fi

# renderer bbulk
renderer_bbulk_avail=no
renderer_bbulk_missing=""
if test ! "x$enable_renderer_bbulk" = "xno" ; then
        renderer_bbulk_avail=yes
else
        renderer_bbulk_missing="enable-renderer-bbulk"
fi

# renderer gltex
renderer_gltex_avail=no
renderer_gltex_missing=""
if test ! "x$enable_renderer_gltex" = "xno" ; then
        renderer_gltex_avail=yes
        if test "x$have_gles2" = "xno" ; then
                renderer_gltex_avail=no
                renderer_gltex_missing="libgles2"
        fi

        if test "x$renderer_gltex_avail" = "xno" ; then
                if test "x$enable_renderer_gltex" = "xyes" ; then
                        AC_ERROR([missing for renderer-gltex: $renderer_gltex_missing])
                fi
        fi
else
        renderer_gltex_missing="enable-renderer-gltex"
fi

# renderer pixman
renderer_pixman_avail=no
renderer_pixman_missing=""
if test ! "x$enable_renderer_pixman" = "xno" ; then
        renderer_pixman_avail=yes
        if test "x$have_pixman" = "xno" ; then
                renderer_pixman_avail=no
                renderer_pixman_missing="pixman"
        fi

        if test "x$renderer_pixman_avail" = "xno" ; then
                if test "x$enable_renderer_pixman" = "xyes" ; then
                        AC_ERROR([missing for renderer-pixman: $renderer_pixman_missing])
                fi
        fi
else
        renderer_pixman_missing="enable-renderer-pixman"
fi

# font unifont
font_unifont_avail=no
font_unifont_missing=""
if test ! "x$enable_font_unifont" = "xno" ; then
        font_unifont_avail=yes
else
        font_unifont_missing="enable-font-unifont"
fi

# font pango
font_pango_avail=no
font_pango_missing=""
if test ! "x$enable_font_pango" = "xno" ; then
        font_pango_avail=yes
        if test "x$have_pango" = "xno" ; then
                font_pango_avail=no
                font_pango_missing="libpango"
        fi

        if test "x$font_pango_avail" = "xno" ; then
                if test "x$enable_font_pango" = "xyes" ; then
                        AC_ERROR([missing for font-pango: $font_pango_missing])
                fi
        fi
else
        font_pango_missing="enable-font-pango"
fi

# session dummy
session_dummy_avail=no
session_dummy_missing=""
if test ! "x$enable_session_dummy" = "xno" ; then
        session_dummy_avail=yes
else
        session_dummy_missing="enable-session-dummy"
fi

# session terminal
session_terminal_avail=no
session_terminal_missing=""
if test ! "x$enable_session_terminal" = "xno" ; then
        session_terminal_avail=yes
else
        session_terminal_missing="enable-session-terminal"
fi

#
# Enable all required modules
# We now know which modules can be built by checking the *_avail variables set
# above. We now only have to disable all modules that are disabled by default
# and the user didn't force-enable them and no other module that is enabled
# needs them. This is done top-down of course.
#

# session terminal
session_terminal_enabled=no
if test "x$session_terminal_avail" = "xyes" ; then
        if test "x${enable_session_terminal% *}" = "xyes" ; then
                session_terminal_enabled=yes
        fi
fi

# session dummy
session_dummy_enabled=no
if test "x$session_dummy_avail" = "xyes" ; then
        if test "x${enable_session_dummy% *}" = "xyes" ; then
                session_dummy_enabled=yes
        fi
fi

# font pango
font_pango_enabled=no
if test "x$font_pango_avail" = "xyes" ; then
        if test "x${enable_font_pango% *}" = "xyes" ; then
                font_pango_enabled=yes
        fi
fi

# font unifont
font_unifont_enabled=no
if test "x$font_unifont_avail" = "xyes" ; then
        if test "x${enable_font_unifont% *}" = "xyes" ; then
                font_unifont_enabled=yes
        fi
fi

# renderer gltex
renderer_gltex_enabled=no
if test "x$renderer_gltex_avail" = "xyes" ; then
        if test "x${enable_renderer_gltex% *}" = "xyes" ; then
                renderer_gltex_enabled=yes
        fi
fi

# renderer pixman
renderer_pixman_enabled=no
if test "x$renderer_pixman_avail" = "xyes" ; then
        if test "x${enable_renderer_pixman% *}" = "xyes" ; then
                renderer_pixman_enabled=yes
        fi
fi

# renderer bbulk
renderer_bbulk_enabled=no
if test "x$renderer_bbulk_avail" = "xyes" ; then
        if test "x${enable_renderer_bbulk% *}" = "xyes" ; then
                renderer_bbulk_enabled=yes
        fi
fi

# multi-seat
multi_seat_enabled=no
if test "x$multi_seat_avail" = "xyes" ; then
        if test "x${enable_multi_seat% *}" = "xyes" ; then
                multi_seat_enabled=yes
        fi
fi

# video drm3d
video_drm3d_enabled=no
if test "x$video_drm3d_avail" = "xyes" ; then
        if test "x${enable_video_drm3d% *}" = "xyes" ; then
                video_drm3d_enabled=yes
        fi
fi

# video drm2d
video_drm2d_enabled=no
if test "x$video_drm2d_avail" = "xyes" ; then
        if test "x${enable_video_drm2d% *}" = "xyes" ; then
                video_drm2d_enabled=yes
        fi
fi

# video fbdev
video_fbdev_enabled=no
if test "x$video_fbdev_avail" = "xyes" ; then
        if test "x${enable_video_fbdev% *}" = "xyes" ; then
                video_fbdev_enabled=yes
        fi
fi

# optimizations
optimizations_enabled=no
if test "x$optimizations_avail" = "xyes" ; then
        if test "x${enable_optimizations% *}" = "xyes" ; then
                optimizations_enabled=yes
        fi
fi

# debug
debug_enabled=no
if test "x$debug_avail" = "xyes" ; then
        if test "x${enable_debug% *}" = "xyes" ; then
                debug_enabled=yes
        fi
fi

#
# Module Configuration
# We have now done all dependency checking and default-value validation and we
# now know which modules are enabled via the *_enabled variables.
# Everything below is related to the configuration of each module and setting
# the correct flags for the build process.
#

# debug
if test "x$debug_enabled" = "xyes" ; then
        AC_DEFINE([BUILD_ENABLE_DEBUG], [1], [Enable debug mode])
else
        AC_DEFINE([NDEBUG], [1], [No Debug])
fi

AM_CONDITIONAL([BUILD_ENABLE_DEBUG],
               [test "x$debug_enabled" = "xyes"])

# optimizations
AM_CONDITIONAL([BUILD_ENABLE_OPTIMIZATIONS],
               [test "x$optimizations_enabled" = "xyes"])

# video fbdev
if test "x$video_fbdev_enabled" = "xyes" ; then
        AC_DEFINE([BUILD_ENABLE_VIDEO_FBDEV], [1],
                  [Build uterm fbdev video backend])
fi

AM_CONDITIONAL([BUILD_ENABLE_VIDEO_FBDEV],
               [test "x$video_fbdev_enabled" = "xyes"])

# video drm2d
if test "x$video_drm2d_enabled" = "xyes" ; then
        AC_DEFINE([BUILD_ENABLE_VIDEO_DRM2D], [1],
                  [Build uterm drm2d drm3d video backend])
fi

AM_CONDITIONAL([BUILD_ENABLE_VIDEO_DRM2D],
               [test "x$video_drm2d_enabled" = "xyes"])

# video drm3d
if test "x$video_drm3d_enabled" = "xyes" ; then
        AC_DEFINE([BUILD_ENABLE_VIDEO_DRM3D], [1],
                  [Build uterm drm3d video backend])
fi

AM_CONDITIONAL([BUILD_ENABLE_VIDEO_DRM3D],
               [test "x$video_drm3d_enabled" = "xyes"])

# multi-seat
if test "x$multi_seat_enabled" = "xyes" ; then
        AC_DEFINE([BUILD_ENABLE_MULTI_SEAT], [1],
                  [Use systemd for multi-seat support])
fi

AM_CONDITIONAL([BUILD_ENABLE_MULTI_SEAT],
               [test "x$multi_seat_enabled" = "xyes"])

# renderer bbulk
if test "x$renderer_bbulk_enabled" = "xyes" ; then
        AC_DEFINE([BUILD_ENABLE_RENDERER_BBULK], [1],
                  [Build bbulk rendering backend])
fi

AM_CONDITIONAL([BUILD_ENABLE_RENDERER_BBULK],
               [test "x$renderer_bbulk_enabled" = "xyes"])

# renderer gltex
if test "x$renderer_gltex_enabled" = "xyes" ; then
        AC_DEFINE([BUILD_ENABLE_RENDERER_GLTEX], [1],
                  [Build gltex rendering backend])
fi

AM_CONDITIONAL([BUILD_ENABLE_RENDERER_GLTEX],
               [test "x$renderer_gltex_enabled" = "xyes"])

# renderer pixman
if test "x$renderer_pixman_enabled" = "xyes" ; then
        AC_DEFINE([BUILD_ENABLE_RENDERER_PIXMAN], [1],
                  [Build pixman rendering backend])
fi

AM_CONDITIONAL([BUILD_ENABLE_RENDERER_PIXMAN],
               [test "x$renderer_pixman_enabled" = "xyes"])

# font unifont
if test "x$font_unifont_enabled" = "xyes" ; then
        AC_DEFINE([BUILD_ENABLE_FONT_UNIFONT], [1],
                  [Build static unifont backend])
fi

AM_CONDITIONAL([BUILD_ENABLE_FONT_UNIFONT],
               [test "x$font_unifont_enabled" = "xyes"])

# font pango
if test "x$font_pango_enabled" = "xyes" ; then
        AC_DEFINE([BUILD_ENABLE_FONT_PANGO], [1],
                  [Build pango font backend])
fi

AM_CONDITIONAL([BUILD_ENABLE_FONT_PANGO],
               [test "x$font_pango_enabled" = "xyes"])

# session dummy
if test "x$session_dummy_enabled" = "xyes" ; then
        AC_DEFINE([BUILD_ENABLE_SESSION_DUMMY], [1],
                  [Build dummy session])
fi

AM_CONDITIONAL([BUILD_ENABLE_SESSION_DUMMY],
               [test "x$session_dummy_enabled" = "xyes"])

# session terminal
if test "x$session_terminal_enabled" = "xyes" ; then
        AC_DEFINE([BUILD_ENABLE_SESSION_TERMINAL], [1],
                  [Build terminal session])
fi

AM_CONDITIONAL([BUILD_ENABLE_SESSION_TERMINAL],
               [test "x$session_terminal_enabled" = "xyes"])

#
# Miscellaneous Checks
# All checks below are independent of module checking or depend on the results
# of it. They do not have any dependencies themselves so they are not part of the
# module infrastructure.
#

# check for _Static_assert
AC_MSG_CHECKING([whether _Static_assert() is supported])
AC_LANG([C])
have_static_assert=yes
AC_COMPILE_IFELSE([AC_LANG_SOURCE([[_Static_assert(1, "What?");]])],
                  [AC_DEFINE([BUILD_HAVE_STATIC_ASSERT],
                             [1],
                             [Define to 1 if _Static_assert() is supported])],
                  [have_static_assert=no])
AC_MSG_RESULT([$have_static_assert])

# check for gbm_bo_get_pitch() function, otherwise gbm_bo_get_stride() is used
if test x$have_gbm = xyes ; then
        save_CFLAGS="$CFLAGS"
        save_LIBS="$LIBS"
        save_LDFLAGS="$LDFLAGS"
        CFLAGS="$DRM_CFLAGS $GBM_CFLAGS"
        LIBS="$DRM_LIBS $GBM_LIBS"
        LDFLAGS=""
        AC_CHECK_LIB([gbm],
                     [gbm_bo_get_pitch],
                     [AC_DEFINE([BUILD_HAVE_GBM_BO_GET_PITCH],
                                [1],
                                [Define to 1 if your libgbm provides gbm_bo_get_pitch])])
        CFLAGS="$save_CFLAGS"
        LIBS="$save_LIBS"
        LDFLAGS="$save_LDFLAGS"
fi

# check for xsltproc
AC_ARG_VAR([XSLTPROC], [xsltproc program])
AC_PATH_PROG(XSLTPROC, xsltproc)
AM_CONDITIONAL([BUILD_HAVE_XSLTPROC], [test "x$XSLTPROC" != "x"])

# check for offline man-pages stylesheet
AC_MSG_CHECKING([for docbook manpages stylesheet])
BUILD_MANPAGES_STYLESHEET="http://docbook.sourceforge.net/release/xsl/current/manpages/docbook.xsl"
AC_PATH_PROGS_FEATURE_CHECK([XSLTPROC_TMP], [xsltproc],
                            AS_IF([`"$ac_path_XSLTPROC_TMP" --nonet "$BUILD_MANPAGES_STYLESHEET" > /dev/null 2>&1`],
                                  [BUILD_HAVE_MANPAGES_STYLESHEET=yes]))
if test "x$BUILD_HAVE_MANPAGES_STYLESHEET" = "xyes"; then
        AM_CONDITIONAL([BUILD_HAVE_MANPAGES_STYLESHEET], true)
        AC_SUBST(BUILD_MANPAGES_STYLESHEET)
        AC_MSG_RESULT([yes])
else
        AM_CONDITIONAL([BUILD_HAVE_MANPAGES_STYLESHEET], false)
        AC_MSG_RESULT([no])
fi

# gles2 helpers
AM_CONDITIONAL([BUILD_HAVE_GLES2], [test "x$have_gles2" = "xyes"])

# check for mandatory objcopy program
AC_ARG_VAR([OBJCOPY], [objcopy program])
AC_CHECK_TOOL([OBJCOPY], [objcopy], "")

if test "x$OBJCOPY" = "x" ; then
        AC_ERROR([Cannot find "objcopy" program])
fi

#
# Makefile vars
# After everything is configured, we correctly substitute the values for the
# makefiles.
#

AC_CONFIG_FILES([Makefile scripts/kmscon])
AC_OUTPUT

#
# Configuration output
# Show configuration to the user so they can check whether everything was
# configured as expected.
#

AC_MSG_NOTICE([Build configuration:

               prefix: $prefix
          exec-prefix: $exec_prefix
               libdir: $libdir
           includedir: $includedir

  Miscellaneous Options:
                debug: $debug_enabled ($debug_avail: $debug_missing)
        optimizations: $optimizations_enabled ($optimizations_avail: $optimizations_missing)
           multi-seat: $multi_seat_enabled ($multi_seat_avail: $multi_seat_missing)

  Video Backends:
                fbdev: $video_fbdev_enabled ($video_fbdev_avail: $video_fbdev_missing)
                drm2d: $video_drm2d_enabled ($video_drm2d_avail: $video_drm2d_missing)
                drm3d: $video_drm3d_enabled ($video_drm3d_avail: $video_drm3d_missing)

  Font Backends:
              unifont: $font_unifont_enabled ($font_unifont_avail: $font_unifont_missing)
                pango: $font_pango_enabled ($font_pango_avail: $font_pango_missing)

  Renderers:
                bbulk: $renderer_bbulk_enabled ($renderer_bbulk_avail: $renderer_bbulk_missing)
                gltex: $renderer_gltex_enabled ($renderer_gltex_avail: $renderer_gltex_missing)
               pixman: $renderer_pixman_enabled ($renderer_pixman_avail: $renderer_pixman_missing)

  Session Types:
                dummy: $session_dummy_enabled ($session_dummy_avail: $session_dummy_missing)
             terminal: $session_terminal_enabled ($session_terminal_avail: $session_terminal_missing)

        Run "${MAKE-make}" to start compilation process])
