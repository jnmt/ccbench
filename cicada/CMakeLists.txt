cmake_minimum_required(VERSION 3.17)

option(ENABLE_SANITIZER "enable sanitizer on debug build" ON)
option(ENABLE_UB_SANITIZER "enable undefined behavior sanitizer on debug build" OFF)
option(ENABLE_COVERAGE "enable coverage on debug build" OFF)

project(cicada
        VERSION 0.0.1
        DESCRIPTION "cicada"
        LANGUAGES CXX)

find_package(Doxygen)
find_package(Threads REQUIRED)
find_package(gflags REQUIRED)
find_package(glog REQUIRED)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fno-omit-fname-pointer")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} -fno-omit-frame-pointer")

set(sanitizers "address")

if (ENABLE_UB_SANITIZER)
  set(sanitizers "${sanitizers},undefined")
endif()

if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
elseif (CMAKE_CXX_COMPILER_ID MATCHES "^(Clang|AppleClang)$")
  set(sanitizers "${sanitizers},nullability")
else()
  message(FATAL_ERROR "unsupported compiler ${CMAKE_CXX_COMPILER_ID}")
endif()

if (ENABLE_SANITIZER)
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fsanitize=${sanitizers}")
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fno-sanitize=alignment")
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fno-sanitize-recover=${sanitizers}")
endif()
if (ENABLE_COVERAGE)
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} --coverage")
endif()

if(CMAKE_SYSTEM_NAME MATCHES "Linux")
  add_definitions(-DLinux)
endif()

add_definitions(-DKEY_SIZE)
add_definitions(-DVAL_SIZE)
add_definitions(-DADD_ANALYSIS)
add_definitions(-DBACK_OFF)
add_definitions(-DINLINE_VERSION_OPT)
add_definitions(-DINLINE_VERSION_PROMOTION)
add_definitions(-DMASSTREE_USE)
add_definitions(-DPARTITION_TABLE)
add_definitions(-DSINGLE_EXEC)
add_definitions(-WRITE_LATEST_ONLY)
add_definitions(-DWORKER1_INSERT_DELAY_RPHASE)

add_executable(cicada.exe)

target_compile_options(cicada.exe
        -Wall -Wextra -Werror -march=native -Wdangling-else -Wchkp -Winvalid-memory-model)

file(GLOB SOURCES
        "*.cc"
        "${CMAKE_CURRENT_SOURCE_DIR}/../common/result.cc"
        "${CMAKE_CURRENT_SOURCE_DIR}/../common/util.cc")
