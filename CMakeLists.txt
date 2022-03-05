#
# Copyright (c) 2019-2020 CNRS
#
# This file is part of eiquadprog.
#
# eiquadprog is free software: you can redistribute it and/or modify
# it under the terms of the GNU Lesser General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# eiquadprog is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public License
# along with eiquadprog.  If not, see <https://www.gnu.org/licenses/>.
#

cmake_minimum_required(VERSION 3.5)

project(eiquadprog
  LANGUAGES CXX
  VERSION 1.2.3
)

set(TARGET_NAME ${PROJECT_NAME}Targets)

# Project options
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif(NOT CMAKE_BUILD_TYPE)
option(EIQUADPROG_TRACE_SOLVER "trace solver on stderr" OFF)
option(EIQUADPROG_BUILD_TESTS "Build tests." OFF)

#### Dependence
find_package(Eigen3 REQUIRED)

#### Library
add_library(${PROJECT_NAME}
  src/eiquadprog-fast.cpp
  src/eiquadprog.cpp
)

if(EIQUADPROG_TRACE_SOLVER)
  target_compile_definitions(${PROJECT_NAME} PRIVATE TRACE_SOLVER)
endif(EIQUADPROG_TRACE_SOLVER)

target_link_libraries(${PROJECT_NAME} PUBLIC Eigen3::Eigen)
target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

#### Install
set(INSTALL_CMAKEDIR share/cmake/${PROJECT_NAME})

install(
  TARGETS
    ${PROJECT_NAME}
  EXPORT ${TARGET_NAME}
  LIBRARY
    COMPONENT lib
  ARCHIVE
    COMPONENT lib
)

install(
  DIRECTORY
    "${CMAKE_SOURCE_DIR}/include/"
  TYPE INCLUDE
)

install(
  EXPORT
    ${TARGET_NAME}
  NAMESPACE
    "${PROJECT_NAME}::"
  DESTINATION
    ${INSTALL_CMAKEDIR}
  COMPONENT
    dev
)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
      COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
  ${CMAKE_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${INSTALL_CMAKEDIR}
)

install(
  FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  DESTINATION
    ${INSTALL_CMAKEDIR}
)

if(EIQUADPROG_BUILD_TESTS)
find_package(Boost REQUIRED COMPONENTS unit_test_framework)
find_program(MEMORYCHECK_COMMAND NAMES valgrind)
set(MEMORYCHECK_COMMAND_OPTIONS "--trace-children=yes --leak-check=full")

add_definitions(-DBOOST_TEST_DYN_LINK -DBOOST_TEST_MAIN)

#test paths and names (without .cpp extension)
set(TESTS
  eiquadprog-basic
  eiquadprog-fast
  eiquadprog-rt
  eiquadprog-both
  test-integration
  )

include (CTest)
enable_testing()
foreach(test ${TESTS})
  add_executable(${test} tests/${test}.cpp)
  target_link_libraries(${test} ${PROJECT_NAME} Boost::unit_test_framework)
  add_test(NAME ${test} COMMAND ${test})
endforeach(test ${TESTS})

add_library(testab tests/TestA.cpp tests/TestB.cpp)
target_link_libraries(testab ${PROJECT_NAME})
target_link_libraries(test-integration testab)
endif(EIQUADPROG_BUILD_TESTS)
