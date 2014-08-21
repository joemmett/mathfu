# Copyright 2014 Google Inc. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
cmake_minimum_required(VERSION 2.8)

get_filename_component(gtest_dir_default
                       "${CMAKE_CURRENT_SOURCE_DIR}/../../googletest" REALPATH)

# Set some variables describing file locations.
set(dependencies_gtest_dir "${gtest_dir_default}"
    CACHE PATH "Directory containing the googletest (gtest) library.")
set(gtest_incdir ${dependencies_gtest_dir}/include)
set(gtest_libdir ${dependencies_gtest_dir})

# Include helper functions and macros used by Google Test.
include(${gtest_libdir}/cmake/internal_utils.cmake)
config_compiler_and_linker()
string(REPLACE "-W4" "-W3" cxx_default "${cxx_default}")
string(REPLACE "-Wshadow" "" cxx_default "${cxx_default}")
string(REPLACE "-Wextra" "" cxx_default "${cxx_default}")

# This is the directory into which the executables are built.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})

include_directories(${gtest_incdir} ${CMAKE_CURRENT_SOURCE_DIR}
                                    ${CMAKE_CURRENT_SOURCE_DIR}/..)

# Common libraries for tests.
if(NOT MSVC)
  find_package(Threads)
endif()
set(COMMON_LIBS "gtest;${CMAKE_THREAD_LIBS_INIT}")

# Generate a rule to build unit test executables.  This only builds
# ${test_name}_tests if SIMD is disabled (see ${mathfu_enable_simd}) or
# ${test_name}_tests and ${test_name}_no_simd_tests if SIMD is enabled where
# the no_simd_tests binary has SIMD disabled.
function(test_executable test_name source)
  cxx_executable_with_flags(${test_name}_tests "${cxx_default}"
    "${COMMON_LIBS}" ${source} ${MATHFU_HEADERS})
  if(mathfu_enable_simd)
    cxx_executable_with_flags(${test_name}_no_simd_tests
      "${cxx_default} ${mathfu_disable_simd_defines}" "${COMMON_LIBS}"
      ${source} ${MATHFU_HEADERS})
  endif()
endfunction()

test_executable(vector vector_test/vector_test.cpp)
test_executable(quaternion quaternion_test/quaternion_test.cpp)
test_executable(matrix matrix_test/matrix_test.cpp)