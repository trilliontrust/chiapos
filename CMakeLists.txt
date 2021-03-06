CMAKE_MINIMUM_REQUIRED(VERSION 3.14 FATAL_ERROR)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE "RELEASE")
ENDIF()

project(chiapos)

set(FSE_LIB ${CMAKE_CURRENT_SOURCE_DIR}/lib/FiniteStateEntropy/lib)
set(FSE_FILES
    ${FSE_LIB}/fse_compress.c
    ${FSE_LIB}/fse_decompress.c
    ${FSE_LIB}/entropy_common.c
    ${FSE_LIB}/hist.c
)

include_directories(
  ${INCLUDE_DIRECTORIES}
  ${CMAKE_CURRENT_SOURCE_DIR}/../lib/include
  ${CMAKE_CURRENT_SOURCE_DIR}/../lib/FiniteStateEntropy/lib
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${CMAKE_CURRENT_SOURCE_DIR}/test
  )

add_library(fse ${FSE_FILES})

# CMake 3.14+
include(FetchContent)

FetchContent_Declare(
  pybind11-src
  GIT_REPOSITORY https://github.com/pybind/pybind11.git
  GIT_TAG        v2.5.0
)
FetchContent_MakeAvailable(pybind11-src)

pybind11_add_module(chiapos ${CMAKE_CURRENT_SOURCE_DIR}/python-bindings/chiapos.cpp)

set (CMAKE_CXX_FLAGS "-g -O3 -Wall -std=c++11 -maes")
try_run(CMAKE_AESNI_TEST_RUN_RESULT
    CMAKE_AESNI_TEST_COMPILE_RESULT
    ${CMAKE_CURRENT_BINARY_DIR}/cmake_aesni_test
    ${CMAKE_CURRENT_SOURCE_DIR}/src/cmake_aesni_test.cpp)

# Did compilation succeed and process return 0 (success)?
IF("${CMAKE_AESNI_TEST_COMPILE_RESULT}" AND ("${CMAKE_AESNI_TEST_RUN_RESULT}" EQUAL 0))
    message(STATUS "AESNI Enabled")
    set (CMAKE_CXX_FLAGS "-g -O3 -Wall -std=c++11 -maes")
ELSE()
    message(STATUS "AESNI Disabled")
    add_compile_definitions (DISABLE_AESNI)
    set (CMAKE_CXX_FLAGS "-g -O3 -Wall -std=c++11")
ENDIF()

add_executable(ProofOfSpace
    src/cli.cpp
)

add_executable(RunTests
    tests/test-main.cpp
    tests/test.cpp
)

if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  target_link_libraries(chiapos PRIVATE fse)
  target_link_libraries(ProofOfSpace fse)
  target_link_libraries(RunTests fse)
elseif (${CMAKE_SYSTEM_NAME} MATCHES "OpenBSD")
  target_link_libraries(chiapos PRIVATE fse)
  target_link_libraries(ProofOfSpace fse)
  target_link_libraries(RunTests fse)
elseif (${CMAKE_SYSTEM_NAME} MATCHES "FreeBSD")
  target_link_libraries(chiapos PRIVATE fse)
  target_link_libraries(ProofOfSpace fse)
  target_link_libraries(RunTests fse)
elseif (WIN32)
  target_link_libraries(chiapos PRIVATE fse)
  target_link_libraries(ProofOfSpace fse)
  target_link_libraries(RunTests fse)
else()
  target_link_libraries(chiapos PRIVATE fse stdc++fs)
  target_link_libraries(ProofOfSpace fse stdc++fs)
  target_link_libraries(RunTests fse stdc++fs)
endif()
