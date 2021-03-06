cmake_minimum_required(VERSION 3.12)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_COVERAGE "Build coverage" OFF)

set(
  HUNTER_CACHE_SERVERS
  "https://github.com/bmstu-iu8-cpp-sem-3/hunter-cache"
  CACHE STRING "Default cache server"
)

include("tools/gate/cmake/HunterGate.cmake")

HunterGate(
  URL "https://github.com/cpp-pm/hunter/archive/v0.23.278.tar.gz"
  SHA1 "35fa55bc1dfcf9f4d7264f8bb26cd1759ce1ae07"
  # FILEPATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/config.cmake"
)

project(suggestclient)
string(APPEND CMAKE_CXX_FLAGS " -pedantic -Werror -Wall -Wextra")
string(APPEND CMAKE_CXX_FLAGS " -Wno-unused-command-line-argument -Wno-unused-parameter")
string(APPEND CMAKE_CXX_FLAGS " -Wshadow -Wnon-virtual-dtor")

hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)

hunter_add_package(gRPC)
find_package(gRPC CONFIG REQUIRED)

set(API_GENERATED_DIR ${CMAKE_CURRENT_SOURCE_DIR}/third-party/suggest/ccpkg)

set(PROTO_GENERATED_SOURCES
  "${API_GENERATED_DIR}/suggest.pb.cc"
  "${API_GENERATED_DIR}/google/api/annotations.pb.cc"
  "${API_GENERATED_DIR}/google/api/http.pb.cc"
  "${API_GENERATED_DIR}/google/rpc/status.pb.cc")

set(PROTO_GENERATED_HEADERS
  "${API_GENERATED_DIR}/suggest.pb.h"
  "${API_GENERATED_DIR}/google/api/annotations.pb.h"
  "${API_GENERATED_DIR}/google/api/http.pb.h"
  "${API_GENERATED_DIR}/google/rpc/status.pb.h")

set(PROTO_GENERATED_GRPC_SOURCES
  "${API_GENERATED_DIR}/suggest.grpc.pb.cc"
  "${API_GENERATED_DIR}/google/api/annotations.grpc.pb.cc"
  "${API_GENERATED_DIR}/google/api/http.grpc.pb.cc"
  "${API_GENERATED_DIR}/google/rpc/status.grpc.pb.cc")

set(PROTO_GENERATED_GRPC_HEADERS
  "${API_GENERATED_DIR}/suggest.grpc.pb.h"
  "${API_GENERATED_DIR}/google/api/annotations.grpc.pb.h"
  "${API_GENERATED_DIR}/google/api/http.grpc.pb.h"
  "${API_GENERATED_DIR}/google/rpc/status.grpc.pb.h")

add_executable(grpc_client
  ${CMAKE_CURRENT_SOURCE_DIR}/sources/main.cpp
  ${PROTO_GENERATED_SOURCES}
  ${PROTO_GENERATED_GRPC_SOURCES})

target_include_directories(grpc_client PRIVATE "${API_GENERATED_DIR}")

target_link_libraries(grpc_client
  gRPC::grpc++ gRPC::grpc++_reflection)

add_executable(tests
    # TODO: enum your files and delete this comment
    ${CMAKE_CURRENT_SOURCE_DIR}/tests/test.cpp)

target_link_libraries(tests GTest::gtest_main)

enable_testing()
add_test(NAME unit_tests COMMAND tests)

if(BUILD_COVERAGE)
    function(ENABLE_PROJECT_COVERAGE PROJECT)
        target_compile_options(${PROJECT} PRIVATE --coverage)
        target_link_libraries(${PROJECT} --coverage)
        add_coverage(${PROJECT})
    endfunction()

    set(ENABLE_COVERAGE ON CACHE BOOL "Enable coverage build." FORCE)
    list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
    find_package(codecov)
    # TODO: change template word to project name and delete this comment
    enable_project_coverage(grpc_client)
    enable_project_coverage(tests)
    list(APPEND LCOV_REMOVE_PATTERNS "'${PROJECT_SOURCE_DIR}/tests/*'")
    coverage_evaluate()
endif()
