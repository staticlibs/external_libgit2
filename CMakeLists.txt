# Copyright 2019, alex at staticlibs.net
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required ( VERSION 2.8.12 )

# project setup
project ( external_libgit2 C CXX )
set ( ${PROJECT_NAME}_STATICLIB_VERSION 0.28.0-2 )
set ( ${PROJECT_NAME}_DESCRIPTION "libgit2 library build for Staticlibs" )
set ( ${PROJECT_NAME}_URL https://github.com/staticlibs/external_libgit2.git )
include ( ${CMAKE_CURRENT_LIST_DIR}/resources/macros.cmake )

# standalone build
if ( NOT DEFINED CMAKE_LIBRARY_OUTPUT_DIRECTORY )
    set ( CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR} )
    set ( ZLIB_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR} )
endif ( )
if ( NOT DEFINED STATICLIB_TOOLCHAIN )
    if ( NOT ${CMAKE_SYSTEM_NAME} STREQUAL "Linux" )
        external_libgit2_add_subdirectory ( ${CMAKE_CURRENT_LIST_DIR}/../external_zlib )
        external_libgit2_add_subdirectory ( ${CMAKE_CURRENT_LIST_DIR}/../external_openssl )
        external_libgit2_add_subdirectory ( ${CMAKE_CURRENT_LIST_DIR}/../external_libssh2 )
    endif ( )
    if ( NOT DEFINED ${PROJECT_NAME}_TOOLCHAIN )
        if ( WIN32 )
            if ( CMAKE_SIZEOF_VOID_P EQUAL 8 )
                set ( ${PROJECT_NAME}_TOOLCHAIN "windows_amd64_msvc" )
            else ( )
                set ( ${PROJECT_NAME}_TOOLCHAIN "windows_i386_msvc" )
            endif ( )
        elseif ( APPLE )
            set ( ${PROJECT_NAME}_TOOLCHAIN "macosx_amd64_clang" )
        elseif ( UNIX )
            set ( ${PROJECT_NAME}_TOOLCHAIN "linux_amd64_gcc" )
        else ( )
            message ( FATAL_ERROR "Cannot determine toolchain to use, please specify [${PROJECT_NAME}_TOOLCHAIN]" )
        endif ( )
    endif ( )
else ( )
    set ( ${PROJECT_NAME}_TOOLCHAIN ${STATICLIB_TOOLCHAIN} )
endif ( )

# libgit2 subproject
add_subdirectory ( ${CMAKE_CURRENT_LIST_DIR}/resources/libgit2_cmake ${CMAKE_CURRENT_BINARY_DIR}/libgit2 )
set_target_properties ( git2 PROPERTIES FOLDER "deps" )
add_custom_target ( ${PROJECT_NAME} DEPENDS git2 )

# pkg-config
set ( ${PROJECT_NAME}_MODULE libgit2 )
set ( ${PROJECT_NAME}_PC_CFLAGS "-I${git2_LOOKASIDE_DIR}/include" )
set ( ${PROJECT_NAME}_PC_CFLAGS "${${PROJECT_NAME}_PC_CFLAGS} -I${CMAKE_CURRENT_LIST_DIR}/resources/libgit2_cmake/include_${${PROJECT_NAME}_TOOLCHAIN}" )
if ( ${PROJECT_NAME}_TOOLCHAIN MATCHES "windows_[^_]+_[^_]+" )
    set ( ${PROJECT_NAME}_PC_CFLAGS "${${PROJECT_NAME}_PC_CFLAGS} /wd4510 /wd4512 /wd4610" )
endif ( )
set ( ${PROJECT_NAME}_PC_LIBS "-L${CMAKE_LIBRARY_OUTPUT_DIRECTORY} -lgit2 -lssh2" )
if ( ${PROJECT_NAME}_TOOLCHAIN MATCHES "linux_[^_]+_[^_]+" )
    set ( ${PROJECT_NAME}_PC_LIBS "${${PROJECT_NAME}_PC_LIBS} -lrt -lpthread" )
elseif ( ${PROJECT_NAME}_TOOLCHAIN MATCHES "windows_[^_]+_[^_]+" )
    set ( ${PROJECT_NAME}_PC_LIBS "${${PROJECT_NAME}_PC_LIBS} -lwinhttp -lrpcrt4 -lcrypt32" )
elseif ( ${PROJECT_NAME}_TOOLCHAIN MATCHES "macosx_[^_]+_[^_]+" )
    set ( ${PROJECT_NAME}_PC_LIBS "${${PROJECT_NAME}_PC_LIBS} -liconv" )
endif ( )
configure_file ( ${CMAKE_CURRENT_LIST_DIR}/resources/pkg-config.in 
        ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/pkgconfig/libgit2.pc )
