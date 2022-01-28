# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2019-2022 Second State INC

message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflowlite_c.so")
configure_file(
  ${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_c.so
  ${CMAKE_CURRENT_BINARY_DIR}/libtensorflowlite_c.so
  COPYONLY
)
if(UNIX AND NOT ANDROID)
  if(WASMEDGE_TENSORFLOW_SYSTEM_NAME STREQUAL "manylinux2014_x86_64")
    message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow.so.2.6.0")
    configure_file(
      ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow.so.2.6.0
      ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow.so.2.6.0
      COPYONLY
    )
    message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.so.2.6.0")
    configure_file(
      ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.so.2.6.0
      ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.so.2.6.0
      COPYONLY
    )
    execute_process(
      COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow.so.2.6.0 ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow.so.2
      COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow.so.2 ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow.so
      COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.so.2.6.0 ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.so.2
      COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.so.2 ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.so
    )
  endif()
endif()

if(UNIX AND NOT ANDROID)
  if(WASMEDGE_TENSORFLOW_SYSTEM_NAME STREQUAL "manylinux2014_x86_64")
    add_executable(wasmedge-tensorflow
      wasmedger.cpp
    )

    target_link_libraries(wasmedge-tensorflow
      PRIVATE
      ${TENSORFLOW_LIB}
      ${TENSORFLOW_LITE_LIB}
      wasmedgeHostModuleWasmEdgeTensorflow
      wasmedgeHostModuleWasmEdgeTensorflowLite
      wasmedgeHostModuleWasmEdgeImage
      wasmedgeVM
      dl
    )
  endif()
endif()

add_executable(wasmedge-tensorflow-lite
  wasmedger-lite.cpp
)

target_link_libraries(wasmedge-tensorflow-lite
  PRIVATE
  ${TENSORFLOW_LITE_LIB}
  wasmedgeHostModuleWasmEdgeTensorflowLite
  wasmedgeHostModuleWasmEdgeImage
  wasmedgeVM
  dl
)

target_include_directories(wasmedge-tensorflow-lite
  PUBLIC
  ${TENSORFLOW_INCLUDE}
)

add_executable(show-tflite-tensor
  show-tflite-tensor.cpp
)

target_link_libraries(show-tflite-tensor
  PRIVATE
  ${TENSORFLOW_LITE_LIB}
)

target_include_directories(show-tflite-tensor
  PUBLIC
  ${TENSORFLOW_INCLUDE}
)
