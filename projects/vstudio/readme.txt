
VisualStudio instructions

libpng version 1.5.25 - December 3, 2015

Copyright (c) 1998-2010 Glenn Randers-Pehrson

This code is released under the libpng license.
For conditions of distribution and use, see the disclaimer
and license in png.h

This directory  contains support for building libpng under MicroSoft
VisualStudio 2010.  It may also work under later versions of VisualStudio.
You should be familiar with VisualStudio before using this directory.

WARNING
=======
Libpng 1.5 erroneously uses /MD when building debug DLL versions of libpng.
It should use /MDd - you can change this under properties\C/C++\Code
Generation\Runtime Library if you need to use the debug runtime for debug
builds.  This will be changed in libpng 1.6 but is currently retained for
compatibility with older libpng 1.5 releases.

Initial preparations
====================
You must enter some information in zlib.props before attempting to build
with this 'solution'.  Please read and edit zlib.props first.  You will
probably not be familiar with the contents of zlib.props - do not worry,
it is mostly harmless.

This is all you need to do to build the 'release' and 'release library'
configurations.

Debugging
=========
The release configurations default to /Ox optimization.  Full debugging
information is produced (in the .pdb), but if you encounter a problem the
optimization may make it difficult to debug.  Simply rebuild with a lower
optimization level (e.g. /Od.)

Linking your application
========================
Normally you should link against the 'release' configuration.  This builds a
DLL for libpng 1.5 with the default runtime options used by Visual Studio
2010.  In particular the runtime library is the "MultiThreaded DLL" version.
If you use Visual Studio defaults to build your application you will have no
problems.

If you don't use the Visual Studio defaults your application must still be built
with the default runtime option (/MD).  If, for some reason, it is not then your
application will crash inside libpng15.dll as soon as libpng tries to read
from a file handle you pass in.

If you do not want to use the DLL, for example for a very small application,
the 'release library' configuration may be more appropriate.  This is built
with a non-standard runtime library - the "MultiThreaded" version.  When you
build your application it must be compiled with this option (/MT), otherwise
it will not build (if you are lucky) or crash (if you are not.)

Stop reading here
=================
You have enough information to build a working application.

Debug versions have limited support
===================================
This solution includes limited support for debug versions of libpng.  You
do not need these unless your own solution itself uses debug builds (it is
far more effective to debug on the release builds, there is no point building
a special debug build.)

The debug build of libpng is minimally supported.  Support for debug builds of
zlib is also minimal.  You really don't want to do this.
