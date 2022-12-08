################################################################################
# Project:  Lib freexl
# Purpose:  CMake build scripts
# Author:   Alexander Lisovenko, alexander.lisovenko@nexgis.com
################################################################################
# Copyright (C) 2016, NextGIS <info@nextgis.com>
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included
# in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
# OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.
################################################################################

cmake_minimum_required(VERSION 2.8.12)
project(freexl)

#------------------------------------------------------------------------------
# internal cmake settings
#------------------------------------------------------------------------------
set(CMAKE_COLOR_MAKEFILE ON)

SET(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})
configure_file(config-msvc.h config.h)
include_directories(${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR})
include(GNUInstallDirs)
set(INSTALL_BIN_DIR ${CMAKE_INSTALL_BINDIR} CACHE INTERNAL "Installation directory for executables" FORCE)
set(INSTALL_LIB_DIR ${CMAKE_INSTALL_LIBDIR} CACHE INTERNAL "Installation directory for libraries" FORCE)
set(INSTALL_INC_DIR "${CMAKE_INSTALL_INCLUDEDIR}" CACHE INTERNAL "Installation directory for headers" FORCE)

include(util)
check_version(SPL_MAJOR_VERSION SPL_MINOR_VERSION SPL_REV_VERSION)
set(VERSION ${SPL_MAJOR_VERSION}.${SPL_MINOR_VERSION}.${SPL_REV_VERSION})
#------------------------------------------------------------------------------
# Source files specification
#------------------------------------------------------------------------------
set(SOURCES
    src/freexl.c
)

include_directories("${PROJECT_SOURCE_DIR}")
include_directories("${PROJECT_SOURCE_DIR}/headers")

#------------------------------------------------------------------------------
# Platform and compiler specific settings
#------------------------------------------------------------------------------
if(WIN32)
    if (MSVC)
        add_definitions(-D_CRT_SECURE_NO_WARNINGS)
        add_definitions(-DDLL_EXPORT)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /fp:precise")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /Ox")
        if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
            string(REGEX REPLACE "/W[0-4]" "/W3"
                CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}"
            )
        else()
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W3")
        endif()
        
        if (MSVC13)
            add_definitions(-DLROUND_PRESENT)
        endif()
    endif()
else()
    #TODO
endif(WIN32)

#------------------------------------------------------------------------------
# search for dependencies
#------------------------------------------------------------------------------
include(FindAnyProject)
find_anyproject(ICONV REQUIRED)

set(LIB_NAME ${PROJECT_NAME})
if(BUILD_SHARED_LIBS)
    add_library(${LIB_NAME} SHARED ${SOURCES})
    set_target_properties(${LIB_NAME} PROPERTIES DEFINE_SYMBOL DLL_EXPORTS)
    set_target_properties(${LIB_NAME} PROPERTIES SOVERSION 1)
else()
    add_library(${LIB_NAME} STATIC ${SOURCES})
    set_target_properties(${LIB_NAME} PROPERTIES SOVERSION 1)
endif()

target_link_extlibraries(${LIB_NAME})

if (REGISTER_PACKAGE)
    export(TARGETS ${LIB_NAME} FILE ${LIB_NAME}-exports.cmake)

    set(EXPORT_PACKAGE_NAME ${LIB_NAME})
    string(TOUPPER ${EXPORT_PACKAGE_NAME} EXPORT_PACKAGE_NAME_UPPER)
    set(EXPORT_PACKAGE_LIB_NAME ${LIB_NAME})

    export(PACKAGE ${EXPORT_PACKAGE_NAME})

    include(CMakePackageConfigHelpers)
    configure_package_config_file(
        ${CMAKE_SOURCE_DIR}/cmake/PackageConfig.cmake.in
        ${CMAKE_CURRENT_BINARY_DIR}/${EXPORT_PACKAGE_NAME}Config.cmake
        INSTALL_DESTINATION ${INSTALL_LIB_DIR}/cmake
        PATH_VARS INSTALL_INC_DIR )

    write_basic_package_version_file(
        ${CMAKE_CURRENT_BINARY_DIR}/${EXPORT_PACKAGE_NAME}ConfigVersion.cmake
        VERSION ${VERSION}
        COMPATIBILITY AnyNewerVersion )
else()
    export(TARGETS ${LIB_NAME} FILE ${LIB_NAME}-exports.cmake EXPORT_LINK_INTERFACE_LIBRARIES)
endif()

#------------------------------------------------------------------------------
# targets installation
#------------------------------------------------------------------------------

install(TARGETS ${LIB_NAME}
  RUNTIME DESTINATION ${INSTALL_BIN_DIR}
  LIBRARY DESTINATION ${INSTALL_LIB_DIR}
  ARCHIVE DESTINATION ${INSTALL_LIB_DIR}
)

file(GLOB HEADERS "${PROJECT_SOURCE_DIR}/headers/*.h")
install(FILES ${HEADERS}
  DESTINATION ${INSTALL_INC_DIR}
)
