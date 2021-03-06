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

# toolchain setup
set ( STATICLIB_TOOLCHAIN linux_amd64_gcc CACHE STRING "toolchain triplet" )
if ( NOT DEFINED STATICLIB_CMAKE )
    set ( STATICLIB_CMAKE ${CMAKE_CURRENT_LIST_DIR}/../../cmake CACHE INTERNAL "" )    
endif ( )
set ( CMAKE_TOOLCHAIN_FILE ${STATICLIB_CMAKE}/toolchains/${STATICLIB_TOOLCHAIN}.cmake CACHE INTERNAL "" )

# project
project ( external_libgit2_test CXX )
include ( ${STATICLIB_CMAKE}/staticlibs_common.cmake )
staticlib_enable_deplibs_cache ( )

# dependencies
if ( NOT DEFINED STATICLIB_DEPS )
    set ( STATICLIB_DEPS ${CMAKE_CURRENT_LIST_DIR}/../../ CACHE INTERNAL "" )    
endif ( )
if ( NOT STATICLIB_TOOLCHAIN MATCHES "linux_[^_]+_[^_]+" )
    staticlib_add_subdirectory ( ${STATICLIB_DEPS}/external_zlib )
    staticlib_add_subdirectory ( ${STATICLIB_DEPS}/external_openssl )
    staticlib_add_subdirectory ( ${STATICLIB_DEPS}/external_libssh2 )
endif ( )

staticlib_add_subdirectory ( ${CMAKE_CURRENT_LIST_DIR}/../../external_libgit2 )
staticlib_add_subdirectory ( ${STATICLIB_DEPS}/staticlib_config )
staticlib_add_subdirectory ( ${STATICLIB_DEPS}/staticlib_support )
set ( ${PROJECT_NAME}_DEPS_PUBLIC libgit2 staticlib_config staticlib_support )
staticlib_pkg_check_modules ( ${PROJECT_NAME}_DEPS_PUBLIC_PC REQUIRED ${PROJECT_NAME}_DEPS_PUBLIC )
set ( ${PROJECT_NAME}_DEPS_PRIVATE libssh2 zlib openssl )
staticlib_pkg_check_modules ( ${PROJECT_NAME}_DEPS_PRIVATE_PC REQUIRED ${PROJECT_NAME}_DEPS_PRIVATE )

# tests
set ( ${PROJECT_NAME}_TEST_INCLUDES ${${PROJECT_NAME}_DEPS_PUBLIC_PC_INCLUDE_DIRS} )
set ( ${PROJECT_NAME}_TEST_LIBS ${${PROJECT_NAME}_DEPS_PUBLIC_PC_LIBRARIES} ${${PROJECT_NAME}_DEPS_PRIVATE_PC_LIBRARIES} )
if ( STATICLIB_TOOLCHAIN MATCHES "macosx_[^_]+_[^_]+" )
    find_library ( ${PROJECT_NAME}_CORELIB CoreFoundation )
    find_library ( ${PROJECT_NAME}_SECLIB Security )
    list ( APPEND ${PROJECT_NAME}_TEST_LIBS
            ${${PROJECT_NAME}_CORELIB}
            ${${PROJECT_NAME}_SECLIB} )
endif ( )
set ( ${PROJECT_NAME}_TEST_OPTS ${${PROJECT_NAME}_DEPS_PUBLIC_PC_CFLAGS_OTHER} )
staticlib_enable_testing ( ${PROJECT_NAME}_TEST_INCLUDES ${PROJECT_NAME}_TEST_LIBS ${PROJECT_NAME}_TEST_OPTS )
