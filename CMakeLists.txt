project(klusters)
cmake_minimum_required(VERSION 2.8.6)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules")

set (QT_MIN_VERSION "4.8.0")

option(
  ENFORCE_QT4_BUILD
  "Enable if you want to enfore a build with Qt4"
  ON
)

option(REGENERATE_DOC "Enable if you want to regenerate doc need kde4" OFF)

# try Qt5 first, and prefer that (if found), but only if not disabled via option
#if(NOT ENFORCE_QT4_BUILD)
  find_package(Qt5Core QUIET)
#endif()

if(Qt5Core_FOUND)
  find_package(Qt5Gui REQUIRED)
  find_package(Qt5Widgets REQUIRED)
  find_package(Qt5PrintSupport REQUIRED)
  find_package(Qt5WebKitWidgets 5.0.2 QUIET)
  find_package(Qt5Xml REQUIRED)


  include(ECMQt4To5Porting)
  include_directories(${QT_INCLUDES}) # TODO: Port away from this.

  set(QT_QTWEBKIT_FOUND ${Qt5WebKitWidgets_FOUND}) # backward compat with Qt4 webkit code

  if(Qt5_POSITION_INDEPENDENT_CODE)
    if(CMAKE_VERSION VERSION_LESS 2.8.9) # TODO remove once we increase the cmake requirement
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
    else()
      set(CMAKE_POSITION_INDEPENDENT_CODE ON)
    endif()
  endif()
  MESSAGE(STATUS "use qt5 version")
# Qt4
else()
  MESSAGE(STATUS "use qt4 version")
  find_package(KDE4)
  find_package(Qt4 ${QT_MIN_VERSION} REQUIRED QtCore QtGui)

  include(${QT_USE_FILE})
  include_directories(${QT_INCLUDES})
endif()

add_definitions(${QT_DEFINITIONS} )

include(MacroEnsureVersion)
find_package(LibKlustersShared 2.0.0 REQUIRED)
include(CheckCXXCompilerFlag)



# This macro is for internal use only
# Return the directories present in gcc's include path.
macro(_DETERMINE_GCC_SYSTEM_INCLUDE_DIRS _lang _result)
  set(${_result})
  set(_gccOutput)
  file(WRITE "${CMAKE_BINARY_DIR}/CMakeFiles/dummy" "\n" )
  execute_process(COMMAND ${CMAKE_C_COMPILER} -v -E -x ${_lang} -dD dummy
                  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/CMakeFiles
                  ERROR_VARIABLE _gccOutput
                  OUTPUT_VARIABLE _gccStdout )
  file(REMOVE "${CMAKE_BINARY_DIR}/CMakeFiles/dummy")

  if( "${_gccOutput}" MATCHES "> search starts here[^\n]+\n *(.+) *\n *End of (search) list" )
    SET(${_result} ${CMAKE_MATCH_1})
    STRING(REPLACE "\n" " " ${_result} "${${_result}}")
    SEPARATE_ARGUMENTS(${_result})
  ENDIF( "${_gccOutput}" MATCHES "> search starts here[^\n]+\n *(.+) *\n *End of (search) list" )
ENDMACRO(_DETERMINE_GCC_SYSTEM_INCLUDE_DIRS _lang)



add_definitions(${QT_DEFINITIONS} )
set( QT_USE_XML TRUE)

# tell cmake where to search for Qt headers:
include_directories(${QT_INCLUDE_DIR} ${CMAKE_CURRENT_BINARY_DIR} ${QT_QTXML_INCLUDE_DIR} ${LIBKLUSTERSSHARED_INCLUDE_DIR})

# This macro is for internal use only
# Return the directories present in gcc's include path.
macro(_DETERMINE_GCC_SYSTEM_INCLUDE_DIRS _lang _result)
  set(${_result})
  set(_gccOutput)
  file(WRITE "${CMAKE_BINARY_DIR}/CMakeFiles/dummy" "\n" )
  execute_process(COMMAND ${CMAKE_C_COMPILER} -v -E -x ${_lang} -dD dummy
                  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/CMakeFiles
                  ERROR_VARIABLE _gccOutput
                  OUTPUT_VARIABLE _gccStdout )
  file(REMOVE "${CMAKE_BINARY_DIR}/CMakeFiles/dummy")

  if( "${_gccOutput}" MATCHES "> search starts here[^\n]+\n *(.+) *\n *End of (search) list" )
    SET(${_result} ${CMAKE_MATCH_1})
    STRING(REPLACE "\n" " " ${_result} "${${_result}}")
    SEPARATE_ARGUMENTS(${_result})
  ENDIF( "${_gccOutput}" MATCHES "> search starts here[^\n]+\n *(.+) *\n *End of (search) list" )
ENDMACRO(_DETERMINE_GCC_SYSTEM_INCLUDE_DIRS _lang)

if (CMAKE_SYSTEM_NAME MATCHES Linux OR CMAKE_SYSTEM_NAME STREQUAL GNU)
   if (CMAKE_COMPILER_IS_GNUCXX)
      set ( _KDE4_PLATFORM_DEFINITIONS -D_XOPEN_SOURCE=500 -D_BSD_SOURCE -D_GNU_SOURCE)
      set ( CMAKE_SHARED_LINKER_FLAGS "-Wl,--fatal-warnings -Wl,--no-undefined -lc ${CMAKE_SHARED_LINKER_FLAGS}")
      set ( CMAKE_MODULE_LINKER_FLAGS "-Wl,--fatal-warnings -Wl,--no-undefined -lc ${CMAKE_MODULE_LINKER_FLAGS}")

      set ( CMAKE_SHARED_LINKER_FLAGS "-Wl,--enable-new-dtags ${CMAKE_SHARED_LINKER_FLAGS}")
      set ( CMAKE_MODULE_LINKER_FLAGS "-Wl,--enable-new-dtags ${CMAKE_MODULE_LINKER_FLAGS}")
      set ( CMAKE_EXE_LINKER_FLAGS "-Wl,--enable-new-dtags ${CMAKE_EXE_LINKER_FLAGS}")

      # we profile...
      if(CMAKE_BUILD_TYPE_TOLOWER MATCHES profile)
        set (CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -fprofile-arcs -ftest-coverage")
        set (CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} -fprofile-arcs -ftest-coverage")
      endif(CMAKE_BUILD_TYPE_TOLOWER MATCHES profile)
   endif (CMAKE_COMPILER_IS_GNUCXX)
   if (CMAKE_C_COMPILER MATCHES "icc")
      set ( _KDE4_PLATFORM_DEFINITIONS -D_XOPEN_SOURCE=500 -D_BSD_SOURCE -D_GNU_SOURCE)
      set ( CMAKE_SHARED_LINKER_FLAGS "-Wl,--fatal-warnings -Wl,--no-undefined -lc ${CMAKE_SHARED_LINKER_FLAGS}")
      set ( CMAKE_MODULE_LINKER_FLAGS "-Wl,--fatal-warnings -Wl,--no-undefined -lc ${CMAKE_MODULE_LINKER_FLAGS}")
   endif (CMAKE_C_COMPILER MATCHES "icc")
endif (CMAKE_SYSTEM_NAME MATCHES Linux OR CMAKE_SYSTEM_NAME STREQUAL GNU)

if (CMAKE_COMPILER_IS_GNUCXX)
   _DETERMINE_GCC_SYSTEM_INCLUDE_DIRS(c++ _dirs)
   set(CMAKE_CXX_IMPLICIT_INCLUDE_DIRECTORIES
       ${CMAKE_CXX_IMPLICIT_INCLUDE_DIRECTORIES} ${_dirs})

   set (KDE4_ENABLE_EXCEPTIONS "-fexceptions -UQT_NO_EXCEPTIONS")
   # Select flags.
   set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O2 -g -DNDEBUG -DQT_NO_DEBUG")
   set(CMAKE_CXX_FLAGS_RELEASE        "-O2 -DNDEBUG -DQT_NO_DEBUG")
   set(CMAKE_CXX_FLAGS_DEBUG          "-g -O2 -fno-reorder-blocks -fno-schedule-insns -fno-inline")
   set(CMAKE_CXX_FLAGS_DEBUGFULL      "-g3 -fno-inline")
   set(CMAKE_CXX_FLAGS_PROFILE        "-g3 -fno-inline -ftest-coverage -fprofile-arcs")
   set(CMAKE_C_FLAGS_RELWITHDEBINFO   "-O2 -g -DNDEBUG -DQT_NO_DEBUG")
   set(CMAKE_C_FLAGS_RELEASE          "-O2 -DNDEBUG -DQT_NO_DEBUG")
   set(CMAKE_C_FLAGS_DEBUG            "-g -O2 -fno-reorder-blocks -fno-schedule-insns -fno-inline")
   set(CMAKE_C_FLAGS_DEBUGFULL        "-g3 -fno-inline")
   set(CMAKE_C_FLAGS_PROFILE          "-g3 -fno-inline -ftest-coverage -fprofile-arcs")

   set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS} -Wno-long-long -std=iso9899:1990 -Wundef -Wcast-align -Werror-implicit-function-declaration -Wchar-subscripts -Wall -W -Wpointer-arith -Wwrite-strings -Wformat-security -Wmissing-format-attribute -fno-common")
   # As of Qt 4.6.x we need to override the new exception macros if we want compile with -fno-exceptions
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wnon-virtual-dtor -Wno-long-long -Wundef -Wcast-align -Wchar-subscripts -Wall -W -Wpointer-arith -Wformat-security -fno-exceptions -DQT_NO_EXCEPTIONS -fno-check-new -fno-common")

   if (CMAKE_SYSTEM_NAME MATCHES Linux OR CMAKE_SYSTEM_NAME STREQUAL GNU)
     # This should not be needed, as it is also part of _KDE4_PLATFORM_DEFINITIONS below.
     # It is kept here nonetheless both for backwards compatibility in case one does not use add_definitions(${KDE4_DEFINITIONS})
     # and also because it is/was needed by glibc for snprintf to be available when building C files.
     # See commit 4a44862b2d178c1d2e1eb4da90010d19a1e4a42c.
     add_definitions (-D_BSD_SOURCE)
   endif (CMAKE_SYSTEM_NAME MATCHES Linux OR CMAKE_SYSTEM_NAME STREQUAL GNU)

   if (CMAKE_SYSTEM_NAME STREQUAL GNU)
      set (CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -pthread")
      set (CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} -pthread")
   endif (CMAKE_SYSTEM_NAME STREQUAL GNU)

   # gcc under Windows
   if (MINGW)
      set (CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--export-all-symbols -Wl,--disable-auto-import")
      set (CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} -Wl,--export-all-symbols -Wl,--disable-auto-import")
   endif (MINGW)

   check_cxx_compiler_flag(-fPIE HAVE_FPIE_SUPPORT)
   if(KDE4_ENABLE_FPIE)
       if(HAVE_FPIE_SUPPORT)
        set (KDE4_CXX_FPIE_FLAGS "-fPIE")
        set (KDE4_PIE_LDFLAGS "-pie")
       else(HAVE_FPIE_SUPPORT)
        message(STATUS "Your compiler doesn't support the PIE flag")
       endif(HAVE_FPIE_SUPPORT)
   endif(KDE4_ENABLE_FPIE)

   check_cxx_compiler_flag(-Woverloaded-virtual __KDE_HAVE_W_OVERLOADED_VIRTUAL)
   if(__KDE_HAVE_W_OVERLOADED_VIRTUAL)
       set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Woverloaded-virtual")
   endif(__KDE_HAVE_W_OVERLOADED_VIRTUAL)

   # visibility support
   check_cxx_compiler_flag(-fvisibility=hidden __KDE_HAVE_GCC_VISIBILITY)
   set( __KDE_HAVE_GCC_VISIBILITY ${__KDE_HAVE_GCC_VISIBILITY} CACHE BOOL "GCC support for hidden visibility")

   # get the gcc version
   exec_program(${CMAKE_C_COMPILER} ARGS ${CMAKE_C_COMPILER_ARG1} --version OUTPUT_VARIABLE _gcc_version_info)

   string (REGEX MATCH "[345]\\.[0-9]\\.[0-9]" _gcc_version "${_gcc_version_info}")
   # gcc on mac just reports: "gcc (GCC) 3.3 20030304 ..." without the patch level, handle this here:
   if (NOT _gcc_version)
      string (REGEX MATCH ".*\\(GCC\\).* ([34]\\.[0-9]) .*" "\\1.0" _gcc_version "${gcc_on_macos}")
      if (gcc_on_macos)
        string (REGEX REPLACE ".*\\(GCC\\).* ([34]\\.[0-9]) .*" "\\1.0" _gcc_version "${_gcc_version_info}")
      endif (gcc_on_macos)
   endif (NOT _gcc_version)

   if (_gcc_version)
      macro_ensure_version("4.1.0" "${_gcc_version}" GCC_IS_NEWER_THAN_4_1)
      macro_ensure_version("4.2.0" "${_gcc_version}" GCC_IS_NEWER_THAN_4_2)
      macro_ensure_version("4.3.0" "${_gcc_version}" GCC_IS_NEWER_THAN_4_3)
   endif (_gcc_version)

   # save a little by making local statics not threadsafe
   # ### do not enable it for older compilers, see
   # ### http://gcc.gnu.org/bugzilla/show_bug.cgi?id=31806
   if (GCC_IS_NEWER_THAN_4_3)
       set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-threadsafe-statics")
   endif (GCC_IS_NEWER_THAN_4_3)

   set(_GCC_COMPILED_WITH_BAD_ALLOCATOR FALSE)
   if (GCC_IS_NEWER_THAN_4_1)
      exec_program(${CMAKE_C_COMPILER} ARGS ${CMAKE_C_COMPILER_ARG1} -v OUTPUT_VARIABLE _gcc_alloc_info)
      string(REGEX MATCH "(--enable-libstdcxx-allocator=mt)" _GCC_COMPILED_WITH_BAD_ALLOCATOR "${_gcc_alloc_info}")
   endif (GCC_IS_NEWER_THAN_4_1)

   if (__KDE_HAVE_GCC_VISIBILITY AND GCC_IS_NEWER_THAN_4_1 AND NOT _GCC_COMPILED_WITH_BAD_ALLOCATOR AND NOT WIN32)
      set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")
      set (KDE4_C_FLAGS "-fvisibility=hidden")
      # check that Qt defines Q_DECL_EXPORT as __attribute__ ((visibility("default")))
      # if it doesn't and KDE compiles with hidden default visibiltiy plugins will break
      set(_source "#include <QtCore/QtGlobal>\n int main()\n {\n #ifndef QT_VISIBILITY_AVAILABLE \n #error QT_VISIBILITY_AVAILABLE is not available\n #endif \n }\n")
      set(_source_file ${CMAKE_BINARY_DIR}/CMakeTmp/check_qt_visibility.cpp)
      file(WRITE "${_source_file}" "${_source}")
      set(_include_dirs "-DINCLUDE_DIRECTORIES:STRING=${QT_INCLUDES}")

      try_compile(_compile_result ${CMAKE_BINARY_DIR} ${_source_file} CMAKE_FLAGS "${_include_dirs}" OUTPUT_VARIABLE _compile_output_var)

      if(NOT _compile_result)
         message("${_compile_output_var}")
         message(FATAL_ERROR "Qt compiled without support for -fvisibility=hidden. This will break plugins and linking of some applications. Please fix your Qt installation (try passing --reduce-exports to configure).")
      endif(NOT _compile_result)

      if (GCC_IS_NEWER_THAN_4_2)
         set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror=return-type -fvisibility-inlines-hidden")
      endif (GCC_IS_NEWER_THAN_4_2)
   else (__KDE_HAVE_GCC_VISIBILITY AND GCC_IS_NEWER_THAN_4_1 AND NOT _GCC_COMPILED_WITH_BAD_ALLOCATOR AND NOT WIN32)
      set (__KDE_HAVE_GCC_VISIBILITY 0)
   endif (__KDE_HAVE_GCC_VISIBILITY AND GCC_IS_NEWER_THAN_4_1 AND NOT _GCC_COMPILED_WITH_BAD_ALLOCATOR AND NOT WIN32)

endif (CMAKE_COMPILER_IS_GNUCXX)


if (CMAKE_C_COMPILER MATCHES "icc")

   set (KDE4_ENABLE_EXCEPTIONS -fexceptions)
   # Select flags.
   set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O2 -g")
   set(CMAKE_CXX_FLAGS_RELEASE        "-O2 -DNDEBUG -DQT_NO_DEBUG")
   set(CMAKE_CXX_FLAGS_DEBUG          "-O2 -g -fno-inline -noalign")
   set(CMAKE_CXX_FLAGS_DEBUGFULL      "-g -fno-inline -noalign")
   set(CMAKE_C_FLAGS_RELWITHDEBINFO   "-O2 -g")
   set(CMAKE_C_FLAGS_RELEASE          "-O2 -DNDEBUG -DQT_NO_DEBUG")
   set(CMAKE_C_FLAGS_DEBUG            "-O2 -g -fno-inline -noalign")
   set(CMAKE_C_FLAGS_DEBUGFULL        "-g -fno-inline -noalign")

   set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -ansi -Wall -w1 -Wpointer-arith -fno-common")
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ansi -Wall -w1 -Wpointer-arith -fno-exceptions -fno-common")

   # visibility support
   set(__KDE_HAVE_ICC_VISIBILITY)
#   check_cxx_compiler_flag(-fvisibility=hidden __KDE_HAVE_ICC_VISIBILITY)
#   if (__KDE_HAVE_ICC_VISIBILITY)
#      set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")
#   endif (__KDE_HAVE_ICC_VISIBILITY)

endif (CMAKE_C_COMPILER MATCHES "icc")

# Adjust code on OS X since file operation are always 64bit there.
if(APPLE)
  set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Dfseeko64=fseeko")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Dfseeko64=fseeko")

  set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Dftello64=ftello")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Dftello64=ftello")
endif()

if (MSVC)
   set (KDE4_ENABLE_EXCEPTIONS -EHsc)
   # Qt disables the native wchar_t type, do it too to avoid linking issues
   set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Zc:wchar_t-" )
   add_definitions( -DUSE_MSVC_COMPILER )
endif()
add_definitions(-DQT_USE_FAST_CONCATENATION -DQT_USE_FAST_OPERATOR_PLUS)

add_definitions(${KDE4_ENABLE_EXCEPTIONS})
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config-klusters.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config-klusters.h )

set(CMAKE_INCLUDE_CURRENT_DIR TRUE)
set(CMAKE_AUTOMOC TRUE)
add_subdirectory(src)
add_subdirectory(po)
add_subdirectory(doc)

INCLUDE(InstallRequiredSystemLibraries)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Cluster cutting application")
set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_CURRENT_SOURCE_DIR}/description)
set(CPACK_PACKAGE_NAME "klusters" )
set(CPACK_PACKAGE_VERSION_MAJOR "2")
set(CPACK_PACKAGE_VERSION_MINOR "0")
set(CPACK_PACKAGE_VERSION_PATCH "0")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "klusters")
set(CPACK_PACKAGE_VERSION "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")
if(WIN32)
  install(FILES ${LIBKLUSTERSSHARED_DLL_DIR}/libklustersshared.dll DESTINATION bin)
  set (QT_QTCORE_LIBRARY ${QT_LIBRARY_DIR}/../bin/QtCore4.dll)
  set (QT_QTGUI_LIBRARY ${QT_LIBRARY_DIR}/../bin/QtGui4.dll)
  set (QT_QTNETWORK_LIBRARY ${QT_LIBRARY_DIR}/../bin/QtNetwork4.dll)
  set (QT_QTWEBKIT_LIBRARY ${QT_LIBRARY_DIR}/../bin/QtWebKit4.dll)
  set (QT_QTXML_LIBRARY ${QT_LIBRARY_DIR}/../bin/QtXml4.dll)
  install(FILES ${QT_QTCORE_LIBRARY} ${QT_QTGUI_LIBRARY} ${QT_QTNETWORK_LIBRARY} ${QT_QTWEBKIT_LIBRARY} ${QT_QTXML_LIBRARY} DESTINATION bin)
  find_path(MINGW_DIR libstdc++-6.dll)
  install(FILES ${MINGW_DIR}/libgcc_s_dw2-1.dll ${MINGW_DIR}/libstdc++-6.dll ${MINGW_DIR}/mingwm10.dll DESTINATION bin)
  set(CPACK_GENERATOR "NSIS")
  set(CPACK_PACKAGE_EXECUTABLES "klusters" "Klusters")
  set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/GPL-3.0.txt)
  set(CPACK_PACKAGE_INSTALL_REGISTRY_KEY "klusters")
  set(CPACK_NSIS_INSTALL_ROOT "$PROGRAMFILES")
  set(CPACK_PACKAGE_INSTALL_DIRECTORY "Klusters")
endif()
if(UNIX)
  if(APPLE)
    set( CPACK_GENERATOR "ZIP" )
    set( CPACK_DMG_VOLUME_NAME "klusters" )
    set( CPACK_SYSTEM_NAME "OSX" )
 else()
    set(CPACK_GENERATOR "TBZ2") # for testing mostly, we probably want STGZ or RPM/DEB
    set(CPACK_SYSTEM_NAME "${CMAKE_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}")

    # Determine distribution and release
    execute_process(COMMAND lsb_release -si OUTPUT_VARIABLE DISTRIBUTION OUTPUT_STRIP_TRAILING_WHITESPACE)
    execute_process(COMMAND lsb_release -sc OUTPUT_VARIABLE RELEASE OUTPUT_STRIP_TRAILING_WHITESPACE)
    execute_process(COMMAND uname -i OUTPUT_VARIABLE ARCHITECTURE)

    # Set package type (deb vs rpm) and info
    if(DISTRIBUTION STREQUAL "Ubuntu")

      # Set Debian-specific information (see http://www.cmake.org/Wiki/CMake:CPackPackageGenerators)
      if(ARCHITECTURE MATCHES ".*x86_64.*")
        set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "amd64")
      else()
        set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "i386")
      endif()
      set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Micha??l Zugaro")
      set(CPACK_DEBIAN_PACKAGE_SECTION "Science")
      set(CPACK_DEBIAN_PACKAGE_RECOMMENDS "nphys-data, neuroscope, ndmanager")
      set(CPACK_DEBIAN_PACKAGE_DEPENDS "libklustersshared, libqt4-network, libqt4-xml, libqtcore4, libqtgui4, libqtwebkit4")
      set(CPACK_DEBIAN_PACKAGE_HOMEPAGE "http://klusters.sourceforge.net")
      set(CPACK_GENERATOR "DEB")
      set(CPACK_PACKAGE_FILE_NAME ${CPACK_PACKAGE_NAME}_${CPACK_PACKAGE_VERSION}-${RELEASE}_${CPACK_DEBIAN_PACKAGE_ARCHITECTURE})
      set(CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA "${CMAKE_CURRENT_SOURCE_DIR}/postinst;${CMAKE_CURRENT_SOURCE_DIR}/postrm;" )

    # >>> TEMPLATE CODE FOR ADDITIONAL DEBIAN-LIKE DISTRIBUTIONS <<<

    #elseif(DISTRIBUTION STREQUAL "Ubuntu")
    #
    #  # Set Debian-specific information (see http://www.cmake.org/Wiki/CMake:CPackPackageGenerators)
    #  if(ARCHITECTURE MATCHES ".*x86_64.*")
    #    set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "amd64")
    #  else()
    #    set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "i386")
    #  endif()
    #  set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Micha??l Zugaro")
    #  set(CPACK_DEBIAN_PACKAGE_SECTION "Science")
    #  set(CPACK_DEBIAN_PACKAGE_RECOMMENDS "nphys-data, neuroscope, ndmanager")
    #  set(CPACK_DEBIAN_PACKAGE_DEPENDS "libklustersshared, libqt4-network, libqt4-xml, libqtcore4, libqtgui4, libqtwebkit4")
    #  set(CPACK_DEBIAN_PACKAGE_HOMEPAGE "http://klusters.sourceforge.net")
    #  set(CPACK_GENERATOR "DEB")
    #  set(CPACK_DEBIAN_PACKAGE_CONTROL_EXTRA "${CMAKE_CURRENT_SOURCE_DIR}/postinst;${CMAKE_CURRENT_SOURCE_DIR}/postrm;" )
    #  set(CPACK_PACKAGE_FILE_NAME ${CPACK_PACKAGE_NAME}_${CPACK_PACKAGE_VERSION}-${RELEASE}_${CPACK_DEBIAN_PACKAGE_ARCHITECTURE})

    elseif(DISTRIBUTION MATCHES "openSUSE.*")

      # Set openSUSE-specific information (see http://www.cmake.org/Wiki/CMake:CPackPackageGenerators)
      if(ARCHITECTURE MATCHES ".*x86_64.*")
        set(CPACK_RPM_PACKAGE_ARCHITECTURE "x86_64")
      else()
        set(CPACK_RPM_PACKAGE_ARCHITECTURE "i586")
      endif()
      set(CPACK_PACKAGE_CONTACT "Micha??l Zugaro")
      set(CPACK_RPM_PACKAGE_GROUP "Science")
      set(CPACK_RPM_PACKAGE_LICENSE "GPL")
      set(CPACK_RPM_PACKAGE_SUGGESTS "nphys-data, neuroscope, ndmanager")
      set(CPACK_RPM_PACKAGE_REQUIRES "libklustersshared, libqt4, libqt4-x11, libQtWebKit4")
      set(CPACK_RPM_PACKAGE_URL "http://klusters.sourceforge.net")
      set(CPACK_GENERATOR "RPM")
      set(CPACK_PACKAGE_FILE_NAME ${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-${RELEASE}.${CPACK_RPM_PACKAGE_ARCHITECTURE})
      set(CPACK_RPM_POST_INSTALL_SCRIPT_FILE "${CMAKE_CURRENT_SOURCE_DIR}/postinst")
      set(CPACK_RPM_POST_UNINSTALL_SCRIPT_FILE "${CMAKE_CURRENT_SOURCE_DIR}/postrm")

    elseif(DISTRIBUTION STREQUAL "Fedora")

      # Set Fedora-specific information (see http://www.cmake.org/Wiki/CMake:CPackPackageGenerators)
      execute_process(COMMAND lsb_release -sr OUTPUT_VARIABLE RELEASE OUTPUT_STRIP_TRAILING_WHITESPACE)
      if(ARCHITECTURE MATCHES ".*x86_64.*")
        set(CPACK_RPM_PACKAGE_ARCHITECTURE "x86_64")
      else()
        set(CPACK_RPM_PACKAGE_ARCHITECTURE "i686")
      endif()
      set(CPACK_PACKAGE_CONTACT "Micha??l Zugaro")
      set(CPACK_RPM_PACKAGE_GROUP "Science")
      set(CPACK_RPM_PACKAGE_LICENSE "GPL")
      set(CPACK_RPM_PACKAGE_SUGGESTS "nphys-data, neuroscope, ndmanager")
      set(CPACK_RPM_PACKAGE_REQUIRES "libklustersshared, qt, qt-mobility, qtwebkit, qt-x11")
      set(CPACK_RPM_PACKAGE_URL "http://klusters.sourceforge.net")
      set(CPACK_GENERATOR "RPM")
      set(CPACK_PACKAGE_FILE_NAME ${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}.fc${RELEASE}.${CPACK_RPM_PACKAGE_ARCHITECTURE})

      # Set Scientific-Linux-specific information (see http://www.cmake.org/Wiki/CMake:CPackPackageGenerators)
      if(ARCHITECTURE MATCHES ".*x86_64.*")
        set(CPACK_RPM_PACKAGE_ARCHITECTURE "x86_64")
      else()
        set(CPACK_RPM_PACKAGE_ARCHITECTURE "i586")
      endif()
      set(CPACK_PACKAGE_CONTACT "Rajat Mani Thomas")
      set(CPACK_RPM_PACKAGE_GROUP "Science")
      set(CPACK_RPM_PACKAGE_LICENSE "GPL")
      set(CPACK_RPM_PACKAGE_SUGGESTS "nphys-data, neuroscope, ndmanager")
      set(CPACK_RPM_PACKAGE_REQUIRES "libklustersshared, libQtCore, libQtGui, libQtWebKit, libQtNetwork, libQtXml")
      set(CPACK_RPM_PACKAGE_URL "http://klusters.sourceforge.net")
      set(CPACK_GENERATOR "RPM")
      set(CPACK_RPM_POST_INSTALL_SCRIPT_FILE "${CMAKE_CURRENT_SOURCE_DIR}/postinst")
      set(CPACK_RPM_POST_UNINSTALL_SCRIPT_FILE "${CMAKE_CURRENT_SOURCE_DIR}/postrm")
      set(CPACK_PACKAGE_FILE_NAME ${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-${RELEASE}.${CPACK_RPM_PACKAGE_ARCHITECTURE})

    # >>> TEMPLATE CODE FOR ADDITIONAL REDHAT-LIKE DISTRIBUTIONS <<<

    #elseif(DISTRIBUTION MATCHES "openSUSE.*")
    #
    #  # Set openSUSE-specific information (see http://www.cmake.org/Wiki/CMake:CPackPackageGenerators)
    #  if(ARCHITECTURE MATCHES ".*x86_64.*")
    #    set(CPACK_RPM_PACKAGE_ARCHITECTURE "x86_64")
    #  else()
    #    set(CPACK_RPM_PACKAGE_ARCHITECTURE "i586")
    #  endif()
    #  set(CPACK_PACKAGE_CONTACT "Micha??l Zugaro")
    #  set(CPACK_RPM_PACKAGE_GROUP "Science")
    #  set(CPACK_RPM_PACKAGE_LICENSE "GPL")
    #  set(CPACK_RPM_PACKAGE_SUGGESTS "nphys-data, neuroscope, ndmanager")
    #  set(CPACK_RPM_PACKAGE_REQUIRES "libklustersshared, libqt4, libqt4-x11, libQtWebKit4")
    #  set(CPACK_RPM_PACKAGE_URL "http://klusters.sourceforge.net")
    #  set(CPACK_GENERATOR "RPM")
    #  set(CPACK_RPM_POST_INSTALL_SCRIPT_FILE "${CMAKE_CURRENT_SOURCE_DIR}/postinst")
    #  set(CPACK_RPM_POST_UNINSTALL_SCRIPT_FILE "${CMAKE_CURRENT_SOURCE_DIR}/postrm")
    #  set(CPACK_PACKAGE_FILE_NAME ${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-${RELEASE}.${CPACK_RPM_PACKAGE_ARCHITECTURE})

    endif()
  endif()
endif()
include(CPack)
