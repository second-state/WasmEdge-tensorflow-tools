# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2019-2022 Second State INC

# Copy the dependency to the target output directory.
if(APPLE)
  message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflowlite_c.dylib")
  configure_file(
    ${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_c.dylib
    ${CMAKE_CURRENT_BINARY_DIR}/libtensorflowlite_c.dylib
    COPYONLY
  )
  message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflowlite_flex.dylib")
  configure_file(
    ${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_flex.dylib
    ${CMAKE_CURRENT_BINARY_DIR}/libtensorflowlite_flex.dylib
    COPYONLY
  )
  message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_cc.2.12.0.dylib")
  configure_file(
    ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_cc.2.12.0.dylib
    ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_cc.2.12.0.dylib
    COPYONLY
  )
  message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.2.12.0.dylib")
  configure_file(
    ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.2.12.0.dylib
    ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.2.12.0.dylib
    COPYONLY
  )
  execute_process(
    COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_cc.2.12.0.dylib ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_cc.2.dylib
    COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_cc.2.dylib ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_cc.dylib
    COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.2.12.0.dylib ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.2.dylib
    COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.2.dylib ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.dylib
  )
elseif(UNIX)
  message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflowlite_c.so")
  configure_file(
    ${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_c.so
    ${CMAKE_CURRENT_BINARY_DIR}/libtensorflowlite_c.so
    COPYONLY
  )
  if(NOT ANDROID)
    message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflowlite_flex.so")
    configure_file(
      ${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_flex.so
      ${CMAKE_CURRENT_BINARY_DIR}/libtensorflowlite_flex.so
      COPYONLY
    )
    message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_cc.so.2.6.0")
    configure_file(
      ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_cc.so.2.12.0
      ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_cc.so.2.12.0
      COPYONLY
    )
    message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.so.2.6.0")
    configure_file(
      ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.so.2.12.0
      ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.so.2.12.0
      COPYONLY
    )
    execute_process(
      COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_cc.so.2.12.0 ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_cc.so.2
      COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_cc.so.2 ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_cc.so
      COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.so.2.12.0 ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.so.2
      COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.so.2 ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.so
    )
  endif()
endif()

if(WASMEDGE_TENSORFLOW_DEPS_TF_LIB)
  add_executable(wasmedge-tensorflow
    wasmedger.cpp
  )

  target_link_libraries(wasmedge-tensorflow
    PRIVATE
    ${WASMEDGE_TENSORFLOW_DEPS_TF_LIB}
    ${WASMEDGE_TENSORFLOW_DEPS_TFLITE_LIB}
    wasmedgeHostModuleWasmEdgeTensorflow
    wasmedgeHostModuleWasmEdgeTensorflowLite
    wasmedgeHostModuleWasmEdgeImage
    wasmedgePO
    wasmedgeVM
    dl
  )
  
  target_include_directories(wasmedge-tensorflow
    PUBLIC
    ${TENSORFLOW_INCLUDE}
  )
endif()

add_executable(wasmedge-tensorflow-lite
  wasmedger-lite.cpp
)

target_link_libraries(wasmedge-tensorflow-lite
  PRIVATE
  ${WASMEDGE_TENSORFLOW_DEPS_TFLITE_LIB}
  wasmedgeHostModuleWasmEdgeTensorflowLite
  wasmedgeHostModuleWasmEdgeImage
  wasmedgePO
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
  ${WASMEDGE_TENSORFLOW_DEPS_TFLITE_LIB}
)

target_include_directories(show-tflite-tensor
  PUBLIC
  ${TENSORFLOW_INCLUDE}
)
