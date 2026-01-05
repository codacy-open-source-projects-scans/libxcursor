# SPDX-License-Identifier: MIT
#
# Copyright (c) 2025, Oracle and/or its affiliates.
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice (including the next
# paragraph) shall be included in all copies or substantial portions of the
# Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL
# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.
#

#
# This is the package version number, not the shared library
# version.  This version number will be substituted into Xcursor.h
#
project(
  'libXcursor',
  'c',
  version: '1.2.3',
  license: 'HPND-sell-variant',
  license_files: 'COPYING',
  meson_version: '>= 1.1.0',
)

cc = meson.get_compiler('c')
conf = configuration_data()

# Replaces AC_USE_SYSTEM_EXTENSIONS
if host_machine.system() == 'sunos'
    system_extensions = '__EXTENSIONS__'
elif host_machine.system() == 'netbsd'
    system_extensions = '_OPENBSD_SOURCE'
else
    system_extensions = '_GNU_SOURCE'
endif
conf.set(system_extensions, 1,
         description: 'Enable non-standardized system API extensions')

# Replacement for XORG_DEFAULT_OPTIONS
if cc.has_argument('-fno-strict-aliasing')
  add_project_arguments('-fno-strict-aliasing', language: 'c')
endif

prog_sed = find_program('sed')

# Set library version for Xcursor.h from package version set above
xcursor_version = meson.project_version()
xcursor_vers_components = xcursor_version.split('.')
conf.set('XCURSOR_LIB_MAJOR', xcursor_vers_components[0],
         description: 'Major version of libXcursor')
conf.set('XCURSOR_LIB_MINOR', xcursor_vers_components[1],
         description: 'Minor version of libXcursor')
conf.set('XCURSOR_LIB_REVISION', xcursor_vers_components[2],
         description: 'Micro revision of libXcursor')
# Temporary solution to allow building with either autoconf or meson
# during the transition period - can be replaced by configure_file()
# and use of '#mesondefine' when configure.ac is removed.
xcursor_h = custom_target(
  input: 'include/X11/Xcursor/Xcursor.h.in',
  output: 'Xcursor.h',
  command: [
    prog_sed,
    '-e', f's/#undef XCURSOR_LIB_MAJOR/#define XCURSOR_LIB_MAJOR @0@/'.format(conf.get('XCURSOR_LIB_MAJOR')),
    '-e', f's/#undef XCURSOR_LIB_MINOR/#define XCURSOR_LIB_MINOR @0@/'.format(conf.get('XCURSOR_LIB_MINOR')),
    '-e', f's/#undef XCURSOR_LIB_REVISION/#define XCURSOR_LIB_REVISION @0@/'.format(conf.get('XCURSOR_LIB_REVISION')),
    '@INPUT@',
  ],
  capture: true,
  install: true,
  install_dir: get_option('includedir') / 'X11/Xcursor'
)

icondir = get_option('icondir')
if icondir == 'auto'
  icondir = get_option('prefix') / get_option('datadir') / 'icons'
endif
conf.set_quoted('ICONDIR', icondir, description: 'Default icon directory')

cursorpath = get_option('cursorpath')
if cursorpath == 'auto'
  cursordirs = [
    '~/.local/share/icons',
    '~/.icons',
    get_option('prefix') / get_option('datadir') / 'icons',
    get_option('prefix') / get_option('datadir') / 'pixmaps'
  ]
  if not cursordirs.contains(icondir)
    cursordirs += icondir
  endif
  cursorpath = ':'.join(cursordirs)
endif
conf.set_quoted('XCURSORPATH', cursorpath,
                description: 'Default search path for cursors')

# Obtain compiler/linker options for dependencies
dep_xrender    = dependency('xrender', required: true, version: '>= 0.8.2')
dep_libxfixes  = dependency('xfixes', required: true)
dep_libx11     = dependency('x11', required: true)
dep_fixesproto = dependency('fixesproto', required: true)

conf.set('HAVE_XFIXES', 1, description: 'Define to 1 if you have Xfixes')

config_h = configure_file(
  configuration : conf,
  output : 'config.h',
)
add_project_arguments('-DHAVE_CONFIG_H', language: 'c')

libXcursor_sources = [
  config_h,
  xcursor_h,
  'src/xcursorint.h',
  'src/cursor.c',
  'src/display.c',
  'src/file.c',
  'src/library.c',
  'src/xlib.c'
]

lib = library(
  'Xcursor',
  libXcursor_sources,
  include_directories: 'include',
  dependencies: [dep_xrender, dep_libxfixes, dep_libx11, dep_fixesproto],
  version: '1.0.2',
  install: true,
)

lib_dep = declare_dependency(
  link_with: lib,
  include_directories: 'include',
)

meson.override_dependency('xcursor', lib_dep)

datarootdir = get_option('prefix') / get_option('datadir')
pc_datarootdir = datarootdir.replace(get_option('prefix'), '${prefix}')
pc_icondir = icondir.replace(datarootdir, '${datarootdir}')
pc_icondir = pc_icondir.replace(get_option('prefix'), '${prefix}')

pkg = import('pkgconfig')
pkg.generate(
  name: 'Xcursor',
  description: 'X Cursor Library',
  filebase: 'xcursor',
  libraries: '-L${libdir} -lXcursor',
  requires: ['xproto'],
  requires_private: ['x11', 'xrender', 'xfixes'],
  variables: [
    'datarootdir=' + pc_datarootdir,
    'icondir=' + pc_icondir,
  ],
  url: 'https://gitlab.freedesktop.org/xorg/lib/libxcursor/'
)

prog_sed = find_program('sed')

libXcursor_manpages = [
  'Xcursor',
  'XcursorAnimateCreate',
  'XcursorAnimateDestroy',
  'XcursorAnimateNext',
  'XcursorCommentCreate',
  'XcursorCommentDestroy',
  'XcursorCommentsCreate',
  'XcursorCommentsDestroy',
  'XcursorCursorsCreate',
  'XcursorCursorsDestroy',
  'XcursorFileLoad',
  'XcursorFileLoadAllImages',
  'XcursorFileLoadImage',
  'XcursorFileLoadImages',
  'XcursorFileSave',
  'XcursorFileSaveImages',
  'XcursorFilenameLoad',
  'XcursorFilenameLoadAllImages',
  'XcursorFilenameLoadCursor',
  'XcursorFilenameLoadCursors',
  'XcursorFilenameLoadImage',
  'XcursorFilenameLoadImages',
  'XcursorFilenameSave',
  'XcursorFilenameSaveImages',
  'XcursorGetDefaultSize',
  'XcursorGetTheme',
  'XcursorGetThemeCore',
  'XcursorImageCreate',
  'XcursorImageDestroy',
  'XcursorImageHash',
  'XcursorImageLoadCursor',
  'XcursorImagesCreate',
  'XcursorImagesDestroy',
  'XcursorImagesLoadCursor',
  'XcursorImagesLoadCursors',
  'XcursorImagesSetName',
  'XcursorLibraryLoadCursor',
  'XcursorLibraryLoadCursors',
  'XcursorLibraryLoadImage',
  'XcursorLibraryLoadImages',
  'XcursorLibraryPath',
  'XcursorLibraryShape',
  'XcursorNoticeCreateBitmap',
  'XcursorNoticePutBitmap',
  'XcursorSetDefaultSize',
  'XcursorSetTheme',
  'XcursorSetThemeCore',
  'XcursorShapeLoadCursor',
  'XcursorShapeLoadCursors',
  'XcursorShapeLoadImage',
  'XcursorShapeLoadImages',
  'XcursorSupportsARGB',
  'XcursorSupportsAnim',
  'XcursorTryShapeBitmapCursor',
  'XcursorTryShapeCursor',
  'XcursorXcFileLoad',
  'XcursorXcFileLoadAllImages',
  'XcursorXcFileLoadImage',
  'XcursorXcFileLoadImages',
  'XcursorXcFileSave'
]

# Change ':' to a comma and a space to help path formatting
comma_cursorpath = cursorpath.replace(':', ', ')

lib_man_suffix = get_option('lib_man_suffix')
foreach man: libXcursor_manpages
  custom_target(
    f'@man@.man',
    input: f'man/@man@.man',
    output: f'@man@.@lib_man_suffix@',
    command: [
      prog_sed,
      '-e', 's/__xorgversion__/"libXcursor @0@" "X Version 11"/'.format(meson.project_version()),
      '-e', f's/__libmansuffix__/@lib_man_suffix@/g',
      '-e', 's%__XCURSORPATH__%@0@%g'.format(comma_cursorpath),
      '@INPUT@',
    ],
    capture: true,
    install: true,
    install_dir: get_option('prefix') / get_option('mandir') / f'man@lib_man_suffix@',
  )
endforeach
