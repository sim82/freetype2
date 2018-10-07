# CMakeLists.txt
#
# Copyright 2013-2018 by
# David Turner, Robert Wilhelm, and Werner Lemberg.
#
# Written originally by John Cary <cary@txcorp.com>
#
# This file is part of the FreeType project, and may only be used, modified,
# and distributed under the terms of the FreeType project license,
# LICENSE.TXT.  By continuing to use, modify, or distribute this file you
# indicate that you have read the license and understand and accept it
# fully.
#
#
# The following will 1. create a build directory and 2. change into it and
# call cmake to configure the build with default parameters as a static
# library.
#
#   cmake -E make_directory build
#   cmake -E chdir build cmake ..
#
# For a dynamic library, use
#
#   cmake -E chdir build cmake -D BUILD_SHARED_LIBS:BOOL=true ..
#
# For a framework on OS X, use
#
#   cmake -E chdir build cmake -G Xcode -D BUILD_FRAMEWORK:BOOL=true ..
#
# For an iOS static library, use
#
#   cmake -E chdir build cmake -G Xcode -D IOS_PLATFORM=OS ..
#
# or
#
#   cmake -E chdir build cmake -G Xcode -D IOS_PLATFORM=SIMULATOR ..
#
# or
#
#   cmake -E chdir build cmake -G Xcode -D IOS_PLATFORM=SIMULATOR64 ..
#
# Finally, build the project with:
#
#   cmake --build build
#
# Install it with
#
#   (sudo) cmake --build build --target install
#
# A binary distribution can be made with
#
#   cmake --build build --config Release --target package
#
# Please refer to the cmake manual for further options, in particular, how
# to modify compilation and linking parameters.
#
# Some notes.
#
# . `cmake' creates configuration files in
#
#     <build-directory>/include/freetype/config
#
#   which should be further modified if necessary.
#
# . You can use `cmake' directly on a freshly cloned FreeType git
#   repository.
#
# . `CMakeLists.txt' is provided as-is since it is normally not used by the
#   developer team.
#
# . Set the `FT_WITH_ZLIB', `FT_WITH_BZIP2', `FT_WITH_PNG', and
#   `FT_WITH_HARFBUZZ' CMake variables to `ON' to force using a dependency.
#   Leave a variable undefined (which is the default) to use the dependency
#   only if it is available.  Set `CMAKE_DISABLE_FIND_PACKAGE_HarfBuzz=TRUE' to
#   disable a dependency completely (CMake package name, so `BZip2' instead of
#   `BZIP2'). Example:
#
#     cmake -DFT_WITH_ZLIB=ON -DCMAKE_DISABLE_FIND_PACKAGE_HarfBuzz=TRUE [...]
#
# . Installation of FreeType can be controlled with the CMake variables
#   `SKIP_INSTALL_HEADERS', `SKIP_INSTALL_LIBRARIES', and `SKIP_INSTALL_ALL'
#   (this is compatible with the same CMake variables in zlib's CMake
#   support).

# FreeType explicitly marks the API to be exported and relies on the compiler
# to hide all other symbols. CMake supports a C_VISBILITY_PRESET property
# starting with 2.8.12.
cmake_minimum_required(VERSION 3.11)

include(CheckIncludeFile)


project(freetype C)

set(VERSION_MAJOR "2")
set(VERSION_MINOR "9")
set(VERSION_PATCH "1")

# SOVERSION scheme: CURRENT.AGE.REVISION
#   If there was an incompatible interface change:
#     Increment CURRENT. Set AGE and REVISION to 0
#   If there was a compatible interface change:
#     Increment AGE. Set REVISION to 0
#   If the source code was changed, but there were no interface changes:
#     Increment REVISION.
set(LIBRARY_VERSION "6.16.0")
set(LIBRARY_SOVERSION "6")

# These options mean "require x and complain if not found". They'll get
# optionally found anyway. Use `-DCMAKE_DISABLE_FIND_PACKAGE_x=TRUE` to disable
# searching for a packge entirely (x is the CMake package name, so "BZip2"
# instead of "BZIP2").
option(FT_WITH_ZLIB "Use system zlib instead of internal library." OFF)
option(FT_WITH_BZIP2 "Support bzip2 compressed fonts." OFF)
option(FT_WITH_PNG "Support PNG compressed OpenType embedded bitmaps." OFF)
option(FT_WITH_HARFBUZZ "Improve auto-hinting of OpenType fonts." OFF)



# Add local cmake modules
list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/builds/cmake)



file(GLOB PUBLIC_HEADERS "include/ft2build.h" "include/freetype/*.h")
file(GLOB PUBLIC_CONFIG_HEADERS "include/freetype/config/*.h")
file(GLOB PRIVATE_HEADERS "include/freetype/internal/*.h")


set(BASE_SRCS
  src/autofit/autofit.c
  src/base/ftbase.c
  src/base/ftbbox.c
  src/base/ftbdf.c
  src/base/ftbitmap.c
  src/base/ftcid.c
  src/base/ftfstype.c
  src/base/ftgasp.c
  src/base/ftglyph.c
  src/base/ftgxval.c
  src/base/ftinit.c
  src/base/ftmm.c
  src/base/ftotval.c
  src/base/ftpatent.c
  src/base/ftpfr.c
  src/base/ftstroke.c
  src/base/ftsynth.c
  src/base/ftsystem.c
  src/base/fttype1.c
  src/base/ftwinfnt.c
  src/bdf/bdf.c
  src/bzip2/ftbzip2.c
  src/cache/ftcache.c
  src/cff/cff.c
  src/cid/type1cid.c
  src/gzip/ftgzip.c
  src/lzw/ftlzw.c
  src/pcf/pcf.c
  src/pfr/pfr.c
  src/psaux/psaux.c
  src/pshinter/pshinter.c
  src/psnames/psnames.c
  src/raster/raster.c
  src/sfnt/sfnt.c
  src/smooth/smooth.c
  src/truetype/truetype.c
  src/type1/type1.c
  src/type42/type42.c
  src/winfonts/winfnt.c
)

if (WIN32)
  enable_language(RC)
  list(APPEND BASE_SRCS builds/windows/ftdebug.c
                        src/base/ftver.rc)
elseif (WINCE)
  list(APPEND BASE_SRCS builds/wince/ftdebug.c)
else ()
  list(APPEND BASE_SRCS src/base/ftdebug.c)
endif ()

if (BUILD_FRAMEWORK)
  list(APPEND BASE_SRCS builds/mac/freetype-Info.plist)
endif ()



add_library(freetype
  ${PUBLIC_HEADERS}
  ${PUBLIC_CONFIG_HEADERS}
  ${PRIVATE_HEADERS}
  ${BASE_SRCS}
)

target_compile_definitions(
  freetype PRIVATE FT2_BUILD_LIBRARY)

if (WIN32)
  target_compile_definitions(
    freetype PRIVATE _CRT_SECURE_NO_WARNINGS _CRT_NONSTDC_NO_WARNINGS)
  if (BUILD_SHARED_LIBS)
    target_compile_definitions(
      freetype PRIVATE DLL_EXPORT)
  endif ()
endif ()

if (BUILD_SHARED_LIBS)
  set_target_properties(freetype PROPERTIES
    VERSION ${LIBRARY_VERSION}
    SOVERSION ${LIBRARY_SOVERSION})
endif ()

# Pick up ftconfig.h and ftoption.h generated above, first.
target_include_directories(
  freetype
    PUBLIC
      $<INSTALL_INTERFACE:include/freetype2>
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    PRIVATE
      ${CMAKE_CURRENT_SOURCE_DIR}/include)



set(PKG_CONFIG_REQUIRED_PRIVATE "")


# Installation
include(GNUInstallDirs)

install(TARGETS
      freetype
  EXPORT
      FreetypeD3Targets
      RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
      LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
      ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
      FRAMEWORK DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(FILES ${PUBLIC_HEADERS} DESTINATION include/freetype2/freetype)
install(FILES ${PUBLIC_CONFIG_HEADERS} DESTINATION include/freetype2/freetype/config)

set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/FreetypeD3)


install(EXPORT FreetypeD3Targets
    FILE FreetypeD3Targets.cmake
    NAMESPACE FreetypeD3::
    DESTINATION ${INSTALL_CONFIGDIR}
)


include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/FreetypeD3Version.cmake
    VERSION 0.0.0
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
    ${CMAKE_CURRENT_LIST_DIR}/cmake/FreetypeD3Config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/FreetypeD3Config.cmake
    INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)

## Install all the helper files
install(
    FILES
      ${CMAKE_CURRENT_BINARY_DIR}/FreetypeD3Config.cmake
      ${CMAKE_CURRENT_BINARY_DIR}/FreetypeD3Version.cmake
    DESTINATION ${INSTALL_CONFIGDIR}
)


#  install(
#    # Note the trailing slash in the argument to `DIRECTORY'!
#    DIRECTORY ${PROJECT_SOURCE_DIR}/include/
#      DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/freetype2
#      COMPONENT headers
#      PATTERN "internal" EXCLUDE
#      PATTERN "ftconfig.h" EXCLUDE
#      PATTERN "ftoption.h" EXCLUDE)
#  install(
#    FILES ${PROJECT_BINARY_DIR}/include/freetype/config/ftconfig.h
#          ${PROJECT_BINARY_DIR}/include/freetype/config/ftoption.h
#      DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/freetype2/freetype/config
#      COMPONENT headers)

#  install(
#    TARGETS freetype
#      EXPORT freetype-targets
#      LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
#      ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
#      FRAMEWORK DESTINATION Library/Frameworks
#      COMPONENT libraries)
#  install(
#    EXPORT freetype-targets
#      DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/freetype
#      FILE freetype-config.cmake
#      COMPONENT headers)

