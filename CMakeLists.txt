# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.11)
project(WasmEdge-Tensorflow-Tools)
unset(WASMEDGE_CORE_CONFIGURED)
unset(WASMEDGE_IMAGE_CONFIGURED)
unset(WASMEDGE_TENSORFLOW_CONFIGURED)

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/cmake)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE RelWithDebInfo)
endif()

if(CMAKE_BUILD_TYPE STREQUAL Release OR CMAKE_BUILD_TYPE STREQUAL RelWithDebInfo)
  set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
endif()
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fuse-ld=gold")
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_ENABLE_EXPORTS ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
option(BUILD_TOOLS "Generate wasmedge and wasmedgec tools." OFF)
option(WASMEDGE_DISABLE_AOT_RUNTIME "Disable WasmEdge LLVM-based ahead of time compilation runtime." OFF)

find_package(Boost "1.68.0" REQUIRED)
find_package(Filesystem REQUIRED Final Experimental)

# Clone required shared libraries
include(FetchContent)
FetchContent_Declare(
  wasmedgetensorflowdeps
  URL      https://github.com/second-state/WasmEdge-tensorflow-deps/releases/download/0.8.0/WasmEdge-tensorflow-deps-TF-0.8.0-manylinux2014_x86_64.tar.gz
  URL_HASH MD5=50b619f81131b504668e4c0f2d127891
)
FetchContent_GetProperties(wasmedgetensorflowdeps)
if (NOT wasmedgetensorflowdeps_POPULATED)
  message(STATUS "Downloading dependency: libtensorflow")
  FetchContent_Populate(wasmedgetensorflowdeps)
  execute_process(
    COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow.so.2.4.0 ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow.so.2
    COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow.so.2 ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow.so
    COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.so.2.4.0 ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.so.2
    COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.so.2 ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.so
  )
  message(STATUS "Downloading dependency: libtensorflow - done")
endif ()
FetchContent_Declare(
  wasmedgetensorflowdepslite
  URL      https://github.com/second-state/WasmEdge-tensorflow-deps/releases/download/0.8.0/WasmEdge-tensorflow-deps-TFLite-0.8.0-manylinux2014_x86_64.tar.gz
  URL_HASH MD5=2734fd657e89965b6f4faa19e58a7b3e
)
FetchContent_GetProperties(wasmedgetensorflowdepslite)
if (NOT wasmedgetensorflowdepslite_POPULATED)
  message(STATUS "Downloading dependency: libtensorflowlite")
  FetchContent_Populate(wasmedgetensorflowdepslite)
  message(STATUS "Downloading dependency: libtensorflowlite - done")
endif ()

set(TENSORFLOW_LIB
  "${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow.so.2.4.0"
  "${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.so.2.4.0"
)
set(TENSORFLOW_LITE_LIB
  "${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_c.so"
)

# Clone wasmedge-core
if(NOT WASMEDGE_CORE_PATH)
  include(FetchContent)
  FetchContent_Declare(
    wasmedge
    GIT_REPOSITORY https://github.com/WasmEdge/WasmEdge.git
    GIT_TAG master
  )

  FetchContent_GetProperties(wasmedge)
  if(NOT wasmedge_POPULATED)
    message(STATUS "Fetching WasmEdge repository")
    FetchContent_Populate(wasmedge)
    message(STATUS "Fetching WasmEdge repository - done")
  endif()
  add_subdirectory(${wasmedge_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR}/utils/WasmEdge-core)
  set(WASMEDGE_CORE_PATH ${wasmedge_SOURCE_DIR})
  set(WASMEDGE_CORE_BIN_PATH ${CMAKE_CURRENT_BINARY_DIR}/utils/WasmEdge-core)
  set(WASMEDGE_CORE_CONFIGURED ON)
else()
  get_filename_component(
    WASMEDGE_CORE_PATH "${WASMEDGE_CORE_PATH}"
    REALPATH
    BASE_DIR "${CMAKE_CURRENT_BINARY_DIR}")
  if(NOT WASMEDGE_CORE_BIN_PATH)
    add_subdirectory(${WASMEDGE_CORE_PATH} ${CMAKE_CURRENT_BINARY_DIR}/utils/WasmEdge-core)
    set(WASMEDGE_CORE_BIN_PATH ${CMAKE_CURRENT_BINARY_DIR}/utils/WasmEdge-core)
    set(WASMEDGE_CORE_CONFIGURED ON)
  else()
    get_filename_component(
      WASMEDGE_CORE_BIN_PATH "${WASMEDGE_CORE_BIN_PATH}"
      REALPATH
      BASE_DIR "${CMAKE_CURRENT_BINARY_DIR}")
  endif()
endif()

# Clone wasmedge-image
if(NOT WASMEDGE_IMAGE_PATH)
  include(FetchContent)
  FetchContent_Declare(
    wasmedge_image
    GIT_REPOSITORY https://github.com/second-state/WasmEdge-image.git
    GIT_TAG 0.8.0-rc2
  )

  FetchContent_GetProperties(wasmedge_image)
  if(NOT wasmedge_image_POPULATED)
    message(STATUS "Fetching WasmEdge-image repository")
    FetchContent_Populate(wasmedge_image)
    message(STATUS "Fetching WasmEdge-image repository - done")
  endif()
  add_subdirectory(${wasmedge_image_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR}/utils/WasmEdge-image)
  set(WASMEDGE_IMAGE_PATH ${wasmedge_image_SOURCE_DIR})
  set(WASMEDGE_IMAGE_BIN_PATH ${CMAKE_CURRENT_BINARY_DIR}/utils/WasmEdge-image)
  set(WASMEDGE_IMAGE_CONFIGURED ON)
else()
  get_filename_component(
    WASMEDGE_IMAGE_PATH "${WASMEDGE_IMAGE_PATH}"
    REALPATH
    BASE_DIR "${CMAKE_CURRENT_BINARY_DIR}")
  if(NOT WASMEDGE_IMAGE_BIN_PATH)
    add_subdirectory(${WASMEDGE_IMAGE_PATH} ${CMAKE_CURRENT_BINARY_DIR}/utils/WasmEdge-image)
    set(WASMEDGE_IMAGE_BIN_PATH ${CMAKE_CURRENT_BINARY_DIR}/utils/WasmEdge-image)
    set(WASMEDGE_IMAGE_CONFIGURED ON)
  else()
    get_filename_component(
      WASMEDGE_IMAGE_BIN_PATH "${WASMEDGE_IMAGE_BIN_PATH}"
      REALPATH
      BASE_DIR "${CMAKE_CURRENT_BINARY_DIR}")
  endif()
endif()

# Clone wasmedge-tensorflow
if(NOT WASMEDGE_TENSORFLOW_PATH)
  include(FetchContent)
  FetchContent_Declare(
    wasmedge_tensorflow
    GIT_REPOSITORY https://github.com/second-state/WasmEdge-tensorflow.git
    GIT_TAG 0.8.0-rc2
  )

  FetchContent_GetProperties(wasmedge_tensorflow)
  if(NOT wasmedge_tensorflow_POPULATED)
    message(STATUS "Fetching WasmEdge-tensorflow repository")
    FetchContent_Populate(wasmedge_tensorflow)
    message(STATUS "Fetching WasmEdge-tensorflow repository - done")
  endif()
  add_subdirectory(${wasmedge_tensorflow_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR}/utils/WasmEdge-tensorflow)
  set(WASMEDGE_TENSORFLOW_PATH ${wasmedge_tensorflow_SOURCE_DIR})
  set(WASMEDGE_TENSORFLOW_BIN_PATH ${CMAKE_CURRENT_BINARY_DIR}/utils/WasmEdge-tensorflow)
  set(WASMEDGE_TENSORFLOW_CONFIGURED ON)
else()
  get_filename_component(
    WASMEDGE_TENSORFLOW_PATH "${WASMEDGE_TENSORFLOW_PATH}"
    REALPATH
    BASE_DIR "${CMAKE_CURRENT_BINARY_DIR}")
  if(NOT WASMEDGE_TENSORFLOW_BIN_PATH)
    add_subdirectory(${WASMEDGE_TENSORFLOW_PATH} ${CMAKE_CURRENT_BINARY_DIR}/utils/WasmEdge-tensorflow)
    set(WASMEDGE_TENSORFLOW_BIN_PATH ${CMAKE_CURRENT_BINARY_DIR}/utils/WasmEdge-tensorflow)
    set(WASMEDGE_TENSORFLOW_CONFIGURED ON)
  else()
    get_filename_component(
      WASMEDGE_TENSORFLOW_BIN_PATH "${WASMEDGE_TENSORFLOW_BIN_PATH}"
      REALPATH
      BASE_DIR "${CMAKE_CURRENT_BINARY_DIR}")
  endif()
endif()

message(STATUS "Set WasmEdge-core source path: ${WASMEDGE_CORE_PATH}")
message(STATUS "Set WasmEdge-core binary path: ${WASMEDGE_CORE_BIN_PATH}")
message(STATUS "Set WasmEdge-image source path: ${WASMEDGE_IMAGE_PATH}")
message(STATUS "Set WasmEdge-image binary path: ${WASMEDGE_IMAGE_BIN_PATH}")
message(STATUS "Set WasmEdge-tensorflow source path: ${WASMEDGE_TENSORFLOW_PATH}")
message(STATUS "Set WasmEdge-tensorflow binary path: ${WASMEDGE_TENSORFLOW_BIN_PATH}")

# TensorFlow dependencies library
add_subdirectory(${WASMEDGE_TENSORFLOW_PATH}/utils/WasmEdge-tensorflow-deps)

include_directories(BEFORE
  ${CMAKE_CURRENT_BINARY_DIR}/include
  ${CMAKE_CURRENT_SOURCE_DIR}/include
)
include_directories(BEFORE
  ${WASMEDGE_IMAGE_BIN_PATH}/include
  ${WASMEDGE_IMAGE_PATH}/include
)
include_directories(BEFORE
  ${WASMEDGE_TENSORFLOW_BIN_PATH}/include
  ${WASMEDGE_TENSORFLOW_PATH}/include
)

add_subdirectory(tools)
