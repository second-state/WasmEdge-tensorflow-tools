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

# WasmEdge dependencies versions
if(NOT WASMEDGE_DEPS_VERSION)
  set(WASMEDGE_DEPS_VERSION "0.11.1-alpha.1")
endif()

# Clone required shared libraries
if(ANDROID)
  if(CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64")
    set(WASMEDGE_TENSORFLOW_SYSTEM_NAME "android_aarch64")
    set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_HASH "a25dafad049cbc998c1f9682c57aec22b2fe5799eeffdd4ed19793a734cde8a4")
  elseif()
    message(FATAL_ERROR "Unsupported architecture: ${CMAKE_SYSTEM_PROCESSOR}")
  endif()
elseif(APPLE)
  if(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64" OR CMAKE_SYSTEM_PROCESSOR STREQUAL "AMD64")
    set(WASMEDGE_TENSORFLOW_SYSTEM_NAME "darwin_x86_64")
    set(WASMEDGE_TENSORFLOW_DEPS_TF_HASH "df8720bb089528dd6930983d0ff23adbae75ad7b92451245702ce24002b5f3c4")
    set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_HASH "2593772df440a768e79d87e74a860378f46fb0b7d1e7805879ab2ec26a093b57")
  else()
    message(FATAL_ERROR "Unsupported architecture: ${CMAKE_SYSTEM_PROCESSOR}")
  endif()
elseif(UNIX)
  if(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64" OR CMAKE_SYSTEM_PROCESSOR STREQUAL "AMD64")
    set(WASMEDGE_TENSORFLOW_SYSTEM_NAME "manylinux2014_x86_64")
    set(WASMEDGE_TENSORFLOW_DEPS_TF_HASH "de0ad0baee9911515e5025906c73e39a885afea6dd9767ea7b3503d97b7afe8f")
    set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_HASH "43b2a782efb58b047c6d33f64d7ac711b24426959f91287d910edb8937c11dea")
  elseif(CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64")
    set(WASMEDGE_TENSORFLOW_SYSTEM_NAME "manylinux2014_aarch64")
    set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_HASH "1f47dcd05f32907848253e0f4b0eb3a6276802dae41d2b7de61214b75ba02395")
  else()
    message(FATAL_ERROR "Unsupported architecture: ${CMAKE_SYSTEM_PROCESSOR}")
  endif()
else()
  message(FATAL_ERROR "Unsupported system: ${CMAKE_SYSTEM_NAME}")
endif()

include(FetchContent)

# Fetch Tensorflow library. Tensorflow has only x86_64.
if(WASMEDGE_TENSORFLOW_SYSTEM_NAME STREQUAL "manylinux2014_x86_64" OR WASMEDGE_TENSORFLOW_SYSTEM_NAME STREQUAL "darwin_x86_64")
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
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow.2.6.0.dylib ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow.2.dylib
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow.2.dylib ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow.dylib
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.2.6.0.dylib ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.2.dylib
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.2.dylib ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.dylib
      )
    else()
      execute_process(
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow.so.2.6.0 ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow.so.2
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow.so.2 ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow.so
        COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.so.2.6.0 ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.so.2
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

# Setup Tensorflow library. Tensorflow has only x86_64.
if(WASMEDGE_TENSORFLOW_SYSTEM_NAME STREQUAL "manylinux2014_x86_64")
  set(WASMEDGE_TENSORFLOW_DEPS_TF_LIB
    "${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow.so.2.6.0"
    "${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.so.2.6.0"
  )
elseif(WASMEDGE_TENSORFLOW_SYSTEM_NAME STREQUAL "darwin_x86_64")
  set(WASMEDGE_TENSORFLOW_DEPS_TF_LIB
    "${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow.2.6.0.dylib"
    "${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.2.6.0.dylib"
  )
endif()
# Setup Tensorflow-lite library.
if(APPLE)
  set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_LIB
    "${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_c.dylib"
  )
elseif(UNIX)
  set(WASMEDGE_TENSORFLOW_DEPS_TFLITE_LIB
    "${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_c.so"
  )
endif()

# Clone wasmedge-core
if(NOT WASMEDGE_CORE_PATH)
  include(FetchContent)
  FetchContent_Declare(
    wasmedge
    GIT_REPOSITORY https://github.com/WasmEdge/WasmEdge.git
    GIT_TAG ${WASMEDGE_DEPS_VERSION}
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
    GIT_TAG ${WASMEDGE_DEPS_VERSION}
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
    GIT_TAG ${WASMEDGE_DEPS_VERSION}
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
