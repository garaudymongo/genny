# Copyright 2019-present MongoDB Inc.
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

cmake_minimum_required(VERSION 3.13 FATAL_ERROR)
cmake_policy(VERSION 3.13)
cmake_policy(SET CMP0110 OLD)

# NB: this version number is duplicated in
# src/CMakeList.txt, src/gennylib/CMakeLists.txt and src/gennylib/src/version.cpp
project(genny VERSION 0.0.1 LANGUAGES CXX)

# Include our local cmake folder in the module path
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

# Use our modules
include(Options)
include(CheckCompiler)
include(ConfigureInstall)
include(ParseAndAddCatchTests)
include(CreateGennyTargets)

if(${GENNY_BUILD_FLAT})
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY  ${CMAKE_BINARY_DIR}/lib CACHE PATH "")
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY  ${CMAKE_BINARY_DIR}/lib CACHE PATH "")
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY  ${CMAKE_BINARY_DIR}/bin CACHE PATH "")
endif()

find_package(Threads REQUIRED)

# Grab OpenSSL right away
include(GetSsl)

# <Boost>
if(GENNY_STATIC_BOOST_PATH)
    set(Boost_USE_STATIC_LIBS ON
        CACHE BOOL "Use static Boost"
    )
    set(Boost_NO_SYSTEM_PATHS ON
        CACHE BOOL "Use non-system Boost"
    )
    set(BOOST_ROOT "${GENNY_STATIC_BOOST_PATH}"
        CACHE PATH "Use a specific path for Boost"
    )
endif()

if(NOT Boost_USE_STATIC_LIBS)
  add_definitions(-DBOOST_ALL_DYN_LINK)
endif()

# Prevent cmake from finding system boost instead of toolchain boost.
set(Boost_NO_BOOST_CMAKE ON)
set(Boost_USE_MULTITHREADED ON)
set(Boost_USE_STATIC_RUNTIME OFF)
find_package(
        Boost
        1.75
        REQUIRED
        COMPONENTS
        filesystem
        log_setup
        log
        program_options
)
# </Boost>

# <yaml-cpp>
find_package(yaml-cpp CONFIG REQUIRED)
# <yaml-cpp>

# Required CMAKE options
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS                OFF     CACHE BOOL "")
set(CMAKE_POSITION_INDEPENDENT_CODE     ON)

enable_testing()
add_subdirectory(src)
