

set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/install)
project(RedlandLibraries)

cmake_minimum_required(VERSION 3.15)
include(ExternalProject)

###################################################################
# Build options
#
option(BUILD_SHARED_LIBS OFF)

# curl paths
set(CURL_SOURCE_DIR ${CMAKE_SOURCE_DIR}/curl-7.69.1)
set(CURL_BUILD_DIR ${CURL_SOURCE_DIR}/build)
set(CURL_INSTALL_DIR ${CURL_SOURCE_DIR}/install)

#################################################################
# Get or build dependencies
#

if (WIN32)
    # build curl from source on windows
    ExternalProject_Add(
            curl
            SOURCE_DIR ${CURL_SOURCE_DIR}
            BINARY_DIR ${CURL_BUILD_DIR}
            CMAKE_ARGS
            -DCMAKE_INSTALL_PREFIX=${CURL_INSTALL_DIR}
    )


    #    # libxml2 paths for windows builds (prebuilt)
    #    set(LIBXML2_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/libxml2-2.7.8.win32)
    #    set(LIBXML2_BIN_DIR ${LIBXML2_INSTALL_PREFIX}/bin)
    #    set(LIBXML2_INCLUDE_DIRS ${LIBXML2_INSTALL_PREFIX}/include)
    #    set(LIBXML2_LIB_DIR ${LIBXML2_INSTALL_PREFIX}/lib)
    #    find_library(LIBXML2_STATIC_LIBRARY
    #            NAMES libxml2
    #            PATHS ${LIBXML2_LIB_DIR}
    #            REQUIRED
    #            )
else ()

endif ()

############################################################################
# Commands for all systems
#

# find libxml2 static library
find_library(LIBXML2_STATIC_LIBRARY
        NAMES libxml2.a libxml2.lib
        PATHS
        /usr/lib/x86_64-linux-gnu # default for WSL
        ${CMAKE_SOURCE_DIR}/libxml2-2.7.8.win32/lib # for windows x64
        REQUIRED
        )
# find libxml2 include dirs
find_path(LIBXML2_INCLUDE_DIRS
        NAMES
        libxml/parser.h
        PATHS
        /usr/include/ # linux
        /usr/include/libxml2/ # linux
        ${CMAKE_SOURCE_DIR}/libxml2-2.7.8.win32/include # windows x64
        )

# find libxslt static library
find_library(LIBXSLT_STATIC_LIBRARY
        NAMES libxslt.a libxslt.lib
        PATHS
        /usr/lib/x86_64-linux-gnu # default for WSL
        ${CMAKE_SOURCE_DIR}/libxslt-1.1.26.win32/lib # for windows x64
        REQUIRED
        )
# find libxslt include dirs
find_path(LIBXSLT_INCLUDE_DIRS
        NAMES
        libxslt/xslt.h
        PATHS
        /usr/include/ # linux (WSL)
        ${CMAKE_SOURCE_DIR}/libxslt-1.1.26.win32/include # for windows x64
        )

# find libcurl static library
find_library(LIBCURL_STATIC_LIBRARY
        NAMES libcurl.a libcurl.lib libcurl-d_imp.lib
        PATHS
        /usr/lib/x86_64-linux-gnu # default for WSL
        ${CURL_INSTALL_DIR}/lib # for windows x64
        REQUIRED
        )
# find libcurl libraries
find_path(LIBCURL_INCLUDE_DIRS
        NAMES
        curl/curl.h
        PATHS
        /usr/include/x86_64-linux-gnu/
        ${CMAKE_SOURCE_DIR}/libxml2-2.7.8.win32/include # windows x64
        )

############################################################
# Compile list of CMAKE_ARGS for raptor
#
message(STATUS "${CMAKE_SOURCE_DIR}/raptor2-2.0.15")
ExternalProject_Add(raptor2
        SOURCE_DIR ${CMAKE_SOURCE_DIR}/raptor2-2.0.15
        BINARY_DIR ${CMAKE_SOURCE_DIR}/raptor2-2.0.15/build

        CMAKE_ARGS
            -DLIBCURL_STATIC_LIBRARY=${LIBCURL_STATIC_LIBRARY}
            -DLIBCURL_INCLUDE_DIRS=${LIBCURL_INCLUDE_DIRS}
            -DLIBXML2_INCLUDE_DIRS=${LIBXML2_INCLUDE_DIRS}
            -DLIBXML2_STATIC_LIBRARY=${LIBXML2_STATIC_LIBRARY}
            -DLIBXSLT_STATIC_LIBRARY=${LIBXSLT_STATIC_LIBRARY}
            -DLIBXSLT_INCLUDE_DIRS=${LIBXSLT_INCLUDE_DIRS}

)

#add_subdirectory(raptor2-2.0.15)


message(STATUS "
    CMake variables:
        CMAKE_SOURCE_DIR                ${CMAKE_SOURCE_DIR}
        CMAKE_SYSTEM                    ${CMAKE_SYSTEM}
        BUILD_SHARED_LIBS               ${BUILD_SHARED_LIBS}

    Curl Directories:
        (only used on windows)
        CURL_SOURCE_DIR                 ${CURL_SOURCE_DIR}
        CURL_BUILD_DIR                  ${CURL_BUILD_DIR}
        CURL_INSTALL_DIR                ${CURL_INSTALL_DIR}
        (Used on all systems)
        LIBCURL_STATIC_LIBRARY          ${LIBCURL_STATIC_LIBRARY}
        LIBCURL_INCLUDE_DIRS            ${LIBCURL_INCLUDE_DIRS}

    LIBXML2 Directories
        LIBXML2_INCLUDE_DIRS            ${LIBXML2_INCLUDE_DIRS}
        LIBXML2_STATIC_LIBRARY          ${LIBXML2_STATIC_LIBRARY}

    LIBXSLT Directories
        LIBXSLT_STATIC_LIBRARY          ${LIBXSLT_STATIC_LIBRARY}
        LIBXSLT_INCLUDE_DIRS            ${LIBXSLT_INCLUDE_DIRS}
")

































