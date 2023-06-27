# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2019-2022 Second State INC

cmake_minimum_required(VERSION 3.11)
project(WasmEdge-Tensorflow-Tools)
unset(WASMEDGE_CORE_CONFIGURED)
unset(WASMEDGE_IMAGE_CONFIGURED)
unset(WASMEDGE_TENSORFLOW_CONFIGURED)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE RelWithDebInfo)
endif()

set(CMAKE_INTERPROCEDURAL_OPTIMIZATION OFF)
if(CMAKE_BUILD_TYPE STREQUAL Release OR CMAKE_BUILD_TYPE STREQUAL RelWithDebInfo)
  set(CMAKE_INTERPROCEDURAL_OPTIMIZATION ON)
  if (CMAKE_GENERATOR STREQUAL Ninja)
    if(CMAKE_COMPILER_IS_GNUCXX)
      list(TRANSFORM CMAKE_C_COMPILE_OPTIONS_IPO REPLACE "^-flto$" "-flto=auto")
      list(TRANSFORM CMAKE_CXX_COMPILE_OPTIONS_IPO REPLACE "^-flto$" "-flto=auto")
    endif()
    set(CMAKE_JOB_POOLS "link=2")
    set(CMAKE_JOB_POOL_LINK link)
  endif()
endif()
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_VISIBILITY_PRESET hidden)
set(CMAKE_ENABLE_EXPORTS ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_VISIBILITY_INLINES_HIDDEN ON)
set(CMAKE_SKIP_RPATH ON)

# List of WasmEdge options
option(WASMEDGE_BUILD_TOOLS "Generate wasmedge and wasmedgec tools." OFF)
# Tools will be built in this project, hence the wasmedge and wasmedgec are not needed.
option(WASMEDGE_BUILD_AOT_RUNTIME "Enable WasmEdge LLVM-based ahead of time compilation runtime." OFF)
# Tools not need the AOT compiler.
option(WASMEDGE_BUILD_SHARED_LIB "Generate the WasmEdge shared library." OFF)
# Tools not need the libwasmedge_c.so.
option(WASMEDGE_IMAGE_BUILD_SHARED_LIB "Generate the libwasmedge-image_c shared library." OFF)
# Tools not need the libwasmedge-image_c.so.
option(WASMEDGE_TENSORFLOW_BUILD_SHARED_LIB "Generate the libwasmedge-image_c shared library." OFF)
# Tools not need the libwasmedge-tensorflow_c.so.

# WasmEdge repositories versions
if(NOT WASMEDGE_REPO_VERSION)
  set(WASMEDGE_REPO_VERSION "0.13.0-alpha.1")
endif()
# WasmEdge dependencies versions
if(NOT WASMEDGE_DEPS_VERSION)
  set(WASMEDGE_DEPS_VERSION "TF-2.12.0-CC")
endif()

# Clone required shared libraries
if(ANDROID)
  if(CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64")
    set(WASMEDGE_TENSORFLOW_SYSTEM_NAME "android_aarch64")
    set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_HASH "2d7dcd7381479d9ffc0968ea66e24a5207b404c7f2ccbdddec6f2a4d6f9813f2")
  elseif()
    message(FATAL_ERROR "Unsupported architecture: ${CMAKE_SYSTEM_PROCESSOR}")
  endif()
elseif(APPLE)
  if(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64" OR CMAKE_SYSTEM_PROCESSOR STREQUAL "AMD64")
    set(WASMEDGE_TENSORFLOW_SYSTEM_NAME "darwin_x86_64")
    set(WASMEDGE_TENSORFLOW_DEPS_TF_HASH "60da72a093cf65d733ca2cb9f331356a1637acfe1645050809bd0cf056b1520f")
    set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_HASH "04b58f4b97220633a8e299a63aba73d9a1f228904081e7d5f18e78d1e38d5f00")
  elseif(CMAKE_SYSTEM_PROCESSOR STREQUAL "arm64")
    set(WASMEDGE_TENSORFLOW_SYSTEM_NAME "darwin_arm64")
    set(WASMEDGE_TENSORFLOW_DEPS_TF_HASH "2ede6d96c7563eb826331469d7d0a1f51c9b1ca311f4398d841f679a5b96705a")
    set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_HASH "cb4562a80ac2067bdabe2464b80e129b9d8ddc6d97ad1a2d7215e06a1e1e8cda")
  else()
    message(FATAL_ERROR "Unsupported architecture: ${CMAKE_SYSTEM_PROCESSOR}")
  endif()
elseif(UNIX)
  if(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64" OR CMAKE_SYSTEM_PROCESSOR STREQUAL "AMD64")
    set(WASMEDGE_TENSORFLOW_SYSTEM_NAME "manylinux2014_x86_64")
    set(WASMEDGE_TENSORFLOW_DEPS_TF_HASH "266465acd642a9d2d80e56c93aa0a255597bfb3034a826bb2225e61f2bebe2e2")
    set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_HASH "110a06bcda1fdc3e744b1728157b66981e235de130f3a34755684e6adcf08341")
  elseif(CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64")
    set(WASMEDGE_TENSORFLOW_SYSTEM_NAME "manylinux2014_aarch64")
    set(WASMEDGE_TENSORFLOW_DEPS_TF_HASH "9c15a3aeeda614c9677fe8980d8fa2cd9600072c4701b8a8189225855b9ca1a8")
    set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_HASH "672b81d3f4b5a6c25dc9bbc3b8c6ac1c0357cfab8105b2a85b8bb8c0b59afcb4")
  else()
    message(FATAL_ERROR "Unsupported architecture: ${CMAKE_SYSTEM_PROCESSOR}")
  endif()
else()
  message(FATAL_ERROR "Unsupported system: ${CMAKE_SYSTEM_NAME}")
endif()

include(FetchContent)

# Fetch Tensorflow library.
if(WASMEDGE_TENSORFLOW_DEPS_TF_HASH)
  FetchContent_Declare(
    wasmedgetensorflowdeps
    URL      "https://github.com/second-state/WasmEdge-tensorflow-deps/releases/download/${WASMEDGE_DEPS_VERSION}/WasmEdge-tensorflow-deps-TF-${WASMEDGE_DEPS_VERSION}-${WASMEDGE_TENSORFLOW_SYSTEM_NAME}.tar.gz"
    URL_HASH "SHA256=${WASMEDGE_TENSORFLOW_DEPS_TF_HASH}"
  )
  FetchContent_GetProperties(wasmedgetensorflowdeps)
  if(NOT wasmedgetensorflowdeps_POPULATED)
    message(STATUS "Downloading dependency: libtensorflow")
    FetchContent_Populate(wasmedgetensorflowdeps)
    if(APPLE)
      execute_process(
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_cc.2.12.0.dylib ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_cc.2.dylib
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_cc.2.dylib ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_cc.dylib
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.2.12.0.dylib ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.2.dylib
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.2.dylib ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.dylib
      )
    else()
      execute_process(
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_cc.so.2.12.0 ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_cc.so.2
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_cc.so.2 ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_cc.so
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.so.2.12.0 ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.so.2
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.so.2 ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.so
      )
    endif()
    message(STATUS "Downloading dependency: libtensorflow - done")
  endif()
endif()
# Fetch Tensorflow-lite library.
FetchContent_Declare(
  wasmedgetensorflowdepslite
  URL      "https://github.com/second-state/WasmEdge-tensorflow-deps/releases/download/${WASMEDGE_DEPS_VERSION}/WasmEdge-tensorflow-deps-TFLite-${WASMEDGE_DEPS_VERSION}-${WASMEDGE_TENSORFLOW_SYSTEM_NAME}.tar.gz"
  URL_HASH "SHA256=${WASMEDGE_TENSORFLOW_DEPS_TFLITE_HASH}"
)
FetchContent_GetProperties(wasmedgetensorflowdepslite)
if (NOT wasmedgetensorflowdepslite_POPULATED)
  message(STATUS "Downloading dependency: libtensorflowlite")
  FetchContent_Populate(wasmedgetensorflowdepslite)
  message(STATUS "Downloading dependency: libtensorflowlite - done")
endif ()

# Setup Tensorflow library.
if(ANDROID)
elseif(APPLE)
  set(WASMEDGE_TENSORFLOW_DEPS_TF_LIB
    "${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_cc.2.12.0.dylib"
    "${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.2.12.0.dylib"
  )
elseif(UNIX)
  set(WASMEDGE_TENSORFLOW_DEPS_TF_LIB
    "${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_cc.so.2.12.0"
    "${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.so.2.12.0"
  )
endif()
# Setup Tensorflow-lite library.
if(ANDROID)
  set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_LIB
    "${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_c.so"
  )
elseif(APPLE)
  set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_LIB
    "${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_c.dylib"
    "${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_flex.dylib"
  )
elseif(UNIX)
  set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_LIB
    "${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_c.so"
    "${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_flex.so"
  )
endif()

# Clone wasmedge-core
if(NOT WASMEDGE_CORE_PATH)
  include(FetchContent)
  FetchContent_Declare(
    wasmedge
    GIT_REPOSITORY https://github.com/WasmEdge/WasmEdge.git
    GIT_TAG ${WASMEDGE_REPO_VERSION}
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
    GIT_TAG master
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
    GIT_TAG master
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

message(STATUS "WasmEdge-tensorflow-tools: Set WasmEdge-core source path: ${WASMEDGE_CORE_PATH}")
message(STATUS "WasmEdge-tensorflow-tools: Set WasmEdge-core binary path: ${WASMEDGE_CORE_BIN_PATH}")
message(STATUS "WasmEdge-tensorflow-tools: Set WasmEdge-image source path: ${WASMEDGE_IMAGE_PATH}")
message(STATUS "WasmEdge-tensorflow-tools: Set WasmEdge-image binary path: ${WASMEDGE_IMAGE_BIN_PATH}")
message(STATUS "WasmEdge-tensorflow-tools: Set WasmEdge-tensorflow source path: ${WASMEDGE_TENSORFLOW_PATH}")
message(STATUS "WasmEdge-tensorflow-tools: Set WasmEdge-tensorflow binary path: ${WASMEDGE_TENSORFLOW_BIN_PATH}")

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
  ${WASMEDGE_TENSORFLOW_PATH}/utils/WasmEdge-tensorflow-deps
)

add_subdirectory(tools)
