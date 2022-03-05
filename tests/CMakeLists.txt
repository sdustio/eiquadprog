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

FIND_PACKAGE(Boost REQUIRED COMPONENTS unit_test_framework)

ADD_DEFINITIONS(-DBOOST_TEST_DYN_LINK -DBOOST_TEST_MAIN)

#test paths and names (without .cpp extension)
SET(TESTS
  eiquadprog-basic
  eiquadprog-fast
  eiquadprog-rt
  eiquadprog-both
  test-integration
  )
ENABLE_TESTING()
FOREACH(test ${TESTS})
  ADD_EXECUTABLE(${test} ${test}.cpp)
  TARGET_LINK_LIBRARIES(${test} ${PROJECT_NAME} Boost::unit_test_framework)
  ADD_TEST(NAME ${test} COMMAND ${test})
ENDFOREACH(test ${TESTS})

ADD_LIBRARY(testab TestA.cpp TestB.cpp)
TARGET_LINK_LIBRARIES(testab ${PROJECT_NAME})
TARGET_LINK_LIBRARIES(test-integration testab)
