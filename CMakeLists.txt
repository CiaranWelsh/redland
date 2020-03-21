

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

# zlib paths
set(ZLIB_INSTALL_DIR ${CMAKE_SOURCE_DIR}/zlib-1.2.5)


#################################################################
# Get or build dependencies
#

if (WIN32 AND MSVC) # AND MSVC so we can add cygwin/mingw specific stuff later
    # libxml paths
    set(LIBXML2_SOURCE_DIR ${CMAKE_SOURCE_DIR}/libxml2-2.9.10/win32)
    set(LIBXML2_BUILD_DIR ${LIBXML2_SOURCE_DIR}/build-windows)
    set(LIBXML2_INSTALL_DIR ${LIBXML2_SOURCE_DIR}/install.windows)

    # get paths to windows builds of iconv
    set(ICONV_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/libiconv_x64-windows)
    set(ICONV_INSTALL_PREFIX D:/redland/libiconv_x64-windows)

    find_path(ICONV_INCLUDE_DIRS
            NAMES iconv.h
            PATHS ${ICONV_INSTALL_PREFIX}/include
            REQUIRED
            )
    find_path(ICONV_LIBRARY_DIRS
            NAMES iconv.lib libiconv.lib
            PATHS ${ICONV_INSTALL_PREFIX}/lib
            REQUIRED
            )

    find_program(CSCRIPT_EXE
            NAMES cscript
            REQUIRED
            )

    SET(LIBXML2_WIN_CONFIGURE_COMMAND "${CSCRIPT_EXE} ${LIBXML2_SOURCE_DIR}/configure.js compiler=msvc prefix=${CMAKE_SOURCE_DIR}/install.windows include=${ICONV_INCLUDE_DIRS} lib=${ICONV_LIBRARY_DIRS}")
    MESSAGE(STATUS "LIBXML2_WIN_CONFIGURE_COMMAND ${LIBXML2_WIN_CONFIGURE_COMMAND}")

    ExternalProject_Add(libxml2
            SOURCE_DIR ${LIBXML2_SOURCE_DIR}
            PREFIX ${LIBXML2_SOURCE_DIR}
            BINARY_DIR ${LIBXML2_SOURCE_DIR}
            CONFIGURE_COMMAND ${CSCRIPT_EXE} ${LIBXML2_SOURCE_DIR}/configure.js compiler=msvc prefix=${CMAKE_SOURCE_DIR}/install.windows include=${ICONV_INCLUDE_DIRS} lib=${ICONV_LIBRARY_DIRS}
            BUILD_COMMAND nmake /f ${LIBXML2_SOURCE_DIR}/Makefile.msvc
            INSTALL_COMMAND nmake install

            )

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
elseif (UNIX)

    set(LIBXML2_SOURCE_DIR ${CMAKE_SOURCE_DIR}/libxml2-2.9.10)
    set(LIBXML2_BUILD_DIR ${LIBXML2_SOURCE_DIR}/build.ubuntu)
    set(LIBXML2_INSTALL_DIR ${LIBXML2_SOURCE_DIR}/install.ubuntu)

    message(STATUS "LIBXML2_SOURCE_DIR ${LIBXML2_SOURCE_DIR}")
    message(STATUS "LIBXML2_BUILD_DIR ${LIBXML2_BUILD_DIR}")
    message(STATUS "LIBXML2_INSTALL_DIR ${LIBXML2_INSTALL_DIR}")

    if (NOT EXISTS ${LIBXML2_SOURCE_DIR}/autogen.sh)
        MESSAGE(FATAL_ERROR "libxml2 autogen.sh script does not exist")

    endif ()


    ExternalProject_Add(
            libxml2
            SOURCE_DIR ${LIBXML2_SOURCE_DIR}
            PREFIX ${LIBXML2_SOURCE_DIR}        # Necessary for getting correct build directory
            BINARY_DIR ${LIBXML2_BUILD_DIR}
            CONFIGURE_COMMAND ${LIBXML2_SOURCE_DIR}/autogen.sh --prefix=${LIBXML2_INSTALL_DIR}
            BUILD_COMMAND make
            INSTALL_COMMAND sudo make install
    )


endif ()
#
#############################################################################
## Commands for all systems
##
#
# find libxml2 static library
#find_library(LIBXML2_STATIC_LIBRARY
#        NAMES libxml2.dll libxml2.so
#        PATHS
#        /usr/lib/x86_64-linux-gnu # default for WSL
#        ${CMAKE_SOURCE_DIR}/libxml2-2.7.8.win32/bin # for windows x64
#        REQUIRED
#        )
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

# find CURL static library
find_library(CURL_STATIC_LIBRARY
        NAMES libcurl.a libcurl.lib libcurl-d_imp.lib
        PATHS
        /usr/lib/x86_64-linux-gnu # default for WSL
        ${CURL_INSTALL_DIR}/lib # for windows x64
        REQUIRED
        )
# find CURL libraries
find_path(CURL_INCLUDE_DIRS
        NAMES
        curl/curl.h
        PATHS
        /usr/include/x86_64-linux-gnu/
        ${CMAKE_SOURCE_DIR}/libxml2-2.7.8.win32/include # windows x64
        )

# find zlib static library
find_library(ZLIB_STATIC_LIBRARY
        NAMES libz.a
        PATHS
        /usr/lib/x86_64-linux-gnu # default for WSL
        ${ZLIB_INSTALL_DIR}/lib # for windows x64
        REQUIRED
        )
# find ZLIB include_dirs
find_path(ZLIB_INCLUDE_DIRS
        NAMES
        zlib/zlib.h zlib.h
        PATHS
        /usr/include
        ${ZLIB_INSTALL_DIR}/include # windows x64
        )


# find libm.h
find_library(LIBM_STATIC_LIBRARY
        NAMES
        libm.a
        PATHS
        /usr/lib/x86_64-linux-gnu/
        /usr/lib
        )
# find math.h header
find_path(MATH_INCLUDE_DIRS
        NAMES
        math.h
        PATHS
        /usr/include
        )

###########################################################
# Compile list of CMAKE_ARGS for raptor

set(RAPTOR_SOURCE_DIR ${CMAKE_SOURCE_DIR}/raptor2-2.0.15)
set(RAPTOR_BUILD_DIR ${CMAKE_SOURCE_DIR}/raptor2-2.0.15/build)
set(RAPTOR_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/raptor2-2.0.15/install)

message(STATUS "${CMAKE_SOURCE_DIR}/raptor2-2.0.15")

ExternalProject_Add(
        raptor2
        SOURCE_DIR ${RAPTOR_SOURCE_DIR}
        BINARY_DIR ${RAPTOR_BUILD_DIR}

        CMAKE_ARGS
        -DCMAKE_INSTALL_PREFIX=${RAPTOR_INSTALL_PREFIX}
        -DCURL_STATIC_LIBRARY=${CURL_STATIC_LIBRARY}
        -DCURL_INCLUDE_DIRS=${CURL_INCLUDE_DIRS}
        -DLIBXML2_INCLUDE_DIRS=${LIBXML2_INCLUDE_DIRS}
        -DLIBXML2_STATIC_LIBRARY=${LIBXML2_STATIC_LIBRARY}
        -DLIBXSLT_STATIC_LIBRARY=${LIBXSLT_STATIC_LIBRARY}
        -DLIBXSLT_INCLUDE_DIRS=${LIBXSLT_INCLUDE_DIRS}
        -DZLIB_INCLUDE_DIRS=${ZLIB_INCLUDE_DIRS}
        -DZLIB_STATIC_LIBRARY=${ZLIB_STATIC_LIBRARY}
        -DMATH_INCLUDE_DIRS=${MATH_INCLUDE_DIRS}
        -DLIBM_STATIC_LIBRARY=${LIBM_STATIC_LIBRARY}
)

#find_library(RAPTOR2_STATIC_LIBRARY)
#set(RAPTOR2_STATIC_LIBRARY ${RAPTOR_INSTALL_PREFIX}/raptor2)
#ExternalProject_Add(
#        rasqal
#        SOURCE_DIR ${RASQAL_SOURCE_DIR}
#        BINARY_DIR ${RASQAL_BUILD_DIR}
#
#        CMAKE_ARGS
#        -DCMAKE_INSTALL_PREFIX=${RASQAL_INSTALL_PREFIX}
#        -DCURL_STATIC_LIBRARY=${CURL_STATIC_LIBRARY}
#        -DCURL_INCLUDE_DIRS=${CURL_INCLUDE_DIRS}
#        -DLIBXML2_INCLUDE_DIRS=${LIBXML2_INCLUDE_DIRS}
#        -DLIBXML2_STATIC_LIBRARY=${LIBXML2_STATIC_LIBRARY}
#        -DLIBXSLT_STATIC_LIBRARY=${LIBXSLT_STATIC_LIBRARY}
#        -DLIBXSLT_INCLUDE_DIRS=${LIBXSLT_INCLUDE_DIRS}
#        -DZLIB_INCLUDE_DIRS=${ZLIB_INCLUDE_DIRS}
#        -DZLIB_STATIC_LIBRARY=${ZLIB_STATIC_LIBRARY}
#        -DMATH_INCLUDE_DIRS=${MATH_INCLUDE_DIRS}
#        -DLIBM_STATIC_LIBRARY=${LIBM_STATIC_LIBRARY}
#)

#add_subdirectory(raptor2-2.0.15)


message(STATUS "
    CMake flags:
        WIN32                           ${WIN32}
        UNIX                            ${UNIX}
        APPLE                           ${APPLE}

    CMake variables:
        CMAKE_SOURCE_DIR                ${CMAKE_SOURCE_DIR}
        CMAKE_SYSTEM                    ${CMAKE_SYSTEM}
        BUILD_SHARED_LIBS               ${BUILD_SHARED_LIBS}

    Windows specific variables:
        CSCRIPT_EXE                     ${CSCRIPT_EXE}

    Curl Directories:
        (only used on windows)
        CURL_SOURCE_DIR                 ${CURL_SOURCE_DIR}
        CURL_BUILD_DIR                  ${CURL_BUILD_DIR}
        CURL_INSTALL_DIR                ${CURL_INSTALL_DIR}
        (Used on all systems)
        CURL_STATIC_LIBRARY             ${CURL_STATIC_LIBRARY}
        CURL_INCLUDE_DIRS               ${CURL_INCLUDE_DIRS}

    LIBXML2 Directories
        LIBXML2_INCLUDE_DIRS            ${LIBXML2_INCLUDE_DIRS}
        LIBXML2_STATIC_LIBRARY          ${LIBXML2_STATIC_LIBRARY}

    LIBXSLT Directories
        LIBXSLT_STATIC_LIBRARY          ${LIBXSLT_STATIC_LIBRARY}
        LIBXSLT_INCLUDE_DIRS            ${LIBXSLT_INCLUDE_DIRS}

    Raptor2 Directories
        RAPTOR_SOURCE_DIR               ${RAPTOR_SOURCE_DIR}
        RAPTOR_BUILD_DIR                ${RAPTOR_BUILD_DIR}
        RAPTOR_INSTALL_PREFIX           ${RAPTOR_INSTALL_PREFIX}

    Zlib Directories:
        ZLIB_INCLUDE_DIRS               ${ZLIB_INCLUDE_DIRS}
        ZLIB_STATIC_LIBRARY             ${ZLIB_STATIC_LIBRARY}
        ZLIB_INSTALL_DIR                ${ZLIB_INSTALL_DIR}

    Math Library Directories
        MATH_INCLUDE_DIRS               ${MATH_INCLUDE_DIRS}
        LIBM_STATIC_LIBRARY             ${LIBM_STATIC_LIBRARY}

    Iconv Directories
        ICONV_LIBRARY_DIRS              ${ICONV_LIBRARY_DIRS}
        ICONV_STATIC_LIBRARY            ${ICONV_STATIC_LIBRARY}
        ICONV_INCLUDE_DIRS              ${ICONV_INCLUDE_DIRS}
")

































