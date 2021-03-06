project(TelepathyLoggerQt4)

cmake_minimum_required(VERSION 2.6)

# CMake policies are used for backwards compatibilty. Setting a policy to a behavior lets newer
# CMake versions where some behaviors changed behave in a way or another. In our specific case,
# From CMake's documentation:
#
# In CMake 2.6.2 and below, CMake Policy settings in scripts loaded by
# the include() and find_package() commands would affect the includer.
# Explicit invocations of cmake_policy(PUSH) and cmake_policy(POP) were
# required to isolate policy changes and protect the includer.  While
# some scripts intend to affect the policies of their includer, most do
# not.  In CMake 2.6.3 and above, include() and find_package() by
# default PUSH and POP an entry on the policy stack around an included
# script, but provide a NO_POLICY_SCOPE option to disable it.  This
# policy determines whether or not to imply NO_POLICY_SCOPE for
# compatibility.  The OLD behavior for this policy is to imply
# NO_POLICY_SCOPE for include() and find_package() commands.  The NEW
# behavior for this policy is to allow the commands to do their default
# cmake_policy PUSH and POP.
#
# This policy was introduced in CMake version 2.6.3.  CMake version
# 2.8.2 warns when the policy is not set and uses OLD behavior.  Use the
# cmake_policy command to set it to OLD or NEW explicitly.
#
# Whenever our cmake_minimum_required version bumps up to 2.7 or 2.6.3, this policy setting can
# hence be removed.
if(POLICY CMP0011)
    cmake_policy(SET CMP0011 NEW)
endif(POLICY CMP0011)

# Making releases:
#   set the new version number:
#     odd minor -> development series
#     even minor -> stable series
#     increment micro for each release within a series
#   set nano_version to 0
#   make the release, tag it
#   set nano_version to 1
set(TP_LOGGER_QT4_MAJOR_VERSION 0)
set(TP_LOGGER_QT4_MINOR_VERSION 5)
set(TP_LOGGER_QT4_MICRO_VERSION 60)
set(TP_LOGGER_QT4_NANO_VERSION  0)

# This value contains the library's SOVERSION. This value is to be increased everytime an API/ABI break
# occurs, and will be used for the SOVERSION of the generated shared libraries.
set(TP_LOGGER_QT4_ABI_VERSION 1)
# This variable is used for the library's long version. It is generated dynamically, so don't change its
# value! Change TP_LOGGER_QT4_ABI_VERSION and TP_LOGGER_QT4_*_VERSION instead.
if (${TP_LOGGER_QT4_NANO_VERSION} EQUAL 0)
    set(TP_LOGGER_QT4_LIBRARY_VERSION ${TP_LOGGER_QT4_ABI_VERSION}.${TP_LOGGER_QT4_MAJOR_VERSION}.${TP_LOGGER_QT4_MINOR_VERSION}.${TP_LOGGER_QT4_MICRO_VERSION})
else (${TP_LOGGER_QT4_NANO_VERSION} EQUAL 0)
    set(TP_LOGGER_QT4_LIBRARY_VERSION ${TP_LOGGER_QT4_ABI_VERSION}.${TP_LOGGER_QT4_MAJOR_VERSION}.${TP_LOGGER_QT4_MINOR_VERSION}.${TP_LOGGER_QT4_MICRO_VERSION}.${TP_QT4_NANO_VERSION})
endif (${TP_LOGGER_QT4_NANO_VERSION} EQUAL 0)

set(PACKAGE_NAME telepathy-logger-qt4)

if (${TP_LOGGER_QT4_NANO_VERSION} EQUAL 0)
    set(PACKAGE_VERSION ${TP_LOGGER_QT4_MAJOR_VERSION}.${TP_LOGGER_QT4_MINOR_VERSION}.${TP_LOGGER_QT4_MICRO_VERSION})
else (${TP_LOGGER_QT4_NANO_VERSION} EQUAL 0)
    set(PACKAGE_VERSION ${TP_LOGGER_QT4_MAJOR_VERSION}.${TP_LOGGER_QT4_MINOR_VERSION}.${TP_LOGGER_QT4_MICRO_VERSION}.${TP_LOGGER_QT4_NANO_VERSION})
endif (${TP_LOGGER_QT4_NANO_VERSION} EQUAL 0)

# where to look first for cmake modules, before ${CMAKE_ROOT}/Modules/ is
# checked
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules")

# Default build type is RelWithDebInfo for release versions and Debug for developement
# versions
if(NOT CMAKE_BUILD_TYPE)
    if(TP_LOGGER_QT4_NANO_VERSION EQUAL 0)
        set(CMAKE_BUILD_TYPE RelWithDebInfo)
    else(TP_LOGGER_QT4_NANO_VERSION EQUAL 0)
        set(CMAKE_BUILD_TYPE Debug)
    endif(TP_LOGGER_QT4_NANO_VERSION EQUAL 0)
endif(NOT CMAKE_BUILD_TYPE)

# This file contains all the needed initialization macros
include(TelepathyDefaults)

# TelepathyLoggerQt4 specific defines needed to trigger deprecation warnings
if (CXX_DEPRECATED_DECLARATIONS)
    set(DEPRECATED_DECLARATIONS_FLAGS "${DEPRECATED_DECLARATIONS_FLAGS} -DTELEPATHY_LOGGER_QT4_DEPRECATED_WARNINGS")
endif (CXX_DEPRECATED_DECLARATIONS)

# This file contains all macros used in the buildsystem
include(TpQtMacros)

include(Doxygen)
include(MacroLogFeature)
include(MacroWriteBasicCMakeVersionFile)

# external dependencies

# Required dependencies
# Find qt4 version >= 4.5
set (QT_MIN_VERSION "4.5.0")
find_package(Qt REQUIRED)
set (TELEPATHYQT4_MIN_VERSION "0.9.1")
find_package(TelepathyQt4 ${TELEPATHYQT4_MIN_VERSION} REQUIRED)

#add_definitions(-DQT_NO_CAST_FROM_ASCII)

set(ENABLE_DEBUG_OUTPUT ON CACHE BOOL "If activated, compiles support for printing debug output to stderr")
if (ENABLE_DEBUG_OUTPUT)
    add_definitions(-DENABLE_DEBUG)
endif (ENABLE_DEBUG_OUTPUT)

# Find python version >= 2.5
find_package(PythonLibrary REQUIRED)
set(REQUIRED_PY 2.5)
if(${PYTHON_SHORT_VERSION} VERSION_GREATER ${REQUIRED_PY} OR ${PYTHON_SHORT_VERSION} VERSION_EQUAL ${REQUIRED_PY})
    message(STATUS "Python ${PYTHON_SHORT_VERSION} found")
else(${PYTHON_SHORT_VERSION} VERSION_GREATER ${REQUIRED_PY} OR ${PYTHON_SHORT_VERSION} VERSION_EQUAL ${REQUIRED_PY})
    message(SEND_ERROR "Python >= ${REQUIRED_PY} is required")
endif(${PYTHON_SHORT_VERSION} VERSION_GREATER ${REQUIRED_PY} OR ${PYTHON_SHORT_VERSION} VERSION_EQUAL ${REQUIRED_PY})

# Check for Qt4 Glib support
include(CheckCXXSourceCompiles)
set(CMAKE_REQUIRED_INCLUDES ${QT_INCLUDES})
set(CMAKE_REQUIRED_DEFINITIONS "")
set(CMAKE_REQUIRED_FLAGS "")

CHECK_CXX_SOURCE_COMPILES("
#include <QtCore/QtGlobal>
int main()
{
#if defined(QT_NO_GLIB)
#error \"Qt was compiled with Glib disabled\"
#endif
return 0;
}"
QT4_GLIB_SUPPORT)
macro_log_feature(QT4_GLIB_SUPPORT "Qt4 Glib Support"
                  "QtCore library using Glib's main event loop"
                  "http://qt.nokia.com/" FALSE ""
                  "Needed, together with Telepathy-Glib, to build most of the unit tests")

find_package(GLIB2 REQUIRED)
find_package(GObject REQUIRED)
find_package(DBus REQUIRED)
find_package(DBusGLib REQUIRED)
find_package(LibXml2 REQUIRED)
find_package(QtGLib REQUIRED)
find_package(TelepathyLogger REQUIRED)

#find_program(GLIB_GENMARSHAL glib-genmarshal)

# Find telepathy-glib
set(TELEPATHY_GLIB_MIN_VERSION 0.16.0)
find_package(TelepathyGlib REQUIRED)

# Activate debug symbols generation
#set(CMAKE_CXX_FLAGS "-g -Wall")

include_directories(${CMAKE_SOURCE_DIR}
                    ${CMAKE_BINARY_DIR}
                    ${QT_INCLUDES}
                    ${TELEPATHY_QT4_INCLUDE_DIR}
                    ${TELEPATHY_LOGGER_INCLUDE_DIR}
                    ${TELEPATHY_GLIB_INCLUDE_DIR}
                    ${GLIB2_INCLUDE_DIR}
                    ${DBUS_GLIB_INCLUDE_DIR}
                    ${QTGLIB_INCLUDE_DIR})

# Search codegen to see if it's installed somewhere
#find_program(CODEGEN_BIN NAMES codegen PATHS /usr/bin /usr/local/bin ~/bin ./codegen ../codegen)

# Macro to run codegen from the subdirs
macro(run_codegen _dir_name _includes _headers)
    set(_prefixed_headers "")
    foreach(_header ${_headers})
        list(APPEND _prefixed_headers ${_dir_name}/${_header})
    endforeach()

    add_custom_command(OUTPUT gen.cpp
                       COMMAND codegen
                       ARGS ${_includes} ${_prefixed_headers}
                              > ${CMAKE_CURRENT_BINARY_DIR}/gen.cpp
                       DEPENDS codegen ${_headers}
                       WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/..)
endmacro()


# Add the source subdirectories
add_subdirectory(codegen)
add_subdirectory(TelepathyLoggerQt4)
#add_subdirectory(tests)
#add_subdirectory(tools)

# Generate config.h
configure_file(${CMAKE_SOURCE_DIR}/config.h.in ${CMAKE_BINARY_DIR}/config.h)

# Create the uninstall target
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake")

# Display the feature log
macro_display_feature_log()

add_custom_command(OUTPUT ${CMAKE_BINARY_DIR}/FIXME.out
                   COMMAND egrep
                   ARGS -A 5 '[F]IXME|[T]ODO|[X]XX' ${CMAKE_SOURCE_DIR}/TelepathyLoggerQt4/*.[ch]*
                        > FIXME.out || true)
add_custom_target(check-local DEPENDS ${CMAKE_BINARY_DIR}/FIXME.out)

include(TelepathyDist)
