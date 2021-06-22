# SPDX-License-Identifier: Apache-2.0

configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/download_dependencies_all.sh
  ${CMAKE_CURRENT_BINARY_DIR}/download_dependencies_all.sh
  COPYONLY
)
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/download_dependencies_tf.sh
  ${CMAKE_CURRENT_BINARY_DIR}/download_dependencies_tf.sh
  COPYONLY
)
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/download_dependencies_tflite.sh
  ${CMAKE_CURRENT_BINARY_DIR}/download_dependencies_tflite.sh
  COPYONLY
)
message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflowlite_c.so")
configure_file(
  ${wasmedgetensorflowdepslite_SOURCE_DIR}/libtensorflowlite_c.so
  ${CMAKE_CURRENT_BINARY_DIR}/libtensorflowlite_c.so
  COPYONLY
)
message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow.so.2.4.0")
configure_file(
  ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow.so.2.4.0
  ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow.so.2.4.0
  COPYONLY
)
message(STATUS "Copying shared library to directory ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.so.2.4.0")
configure_file(
  ${wasmedgetensorflowdeps_SOURCE_DIR}/libtensorflow_framework.so.2.4.0
  ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.so.2.4.0
  COPYONLY
)
execute_process(
  COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow.so.2.4.0 ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow.so.2
  COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow.so.2 ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow.so
  COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.so.2.4.0 ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.so.2
  COMMAND ${CMAKE_COMMAND} -E create_symlink libtensorflow_framework.so.2 ${CMAKE_CURRENT_BINARY_DIR}/libtensorflow_framework.so
)

if (NOT WASMEDGE_DISABLE_AOT_RUNTIME)
  add_executable(wasmedgec-tensorflow
    wasmedgec.cpp
  )

  target_link_libraries(wasmedgec-tensorflow
    PRIVATE
    wasmedgeLoader
    wasmedgeCommon
    wasmedgeValidator
    wasmedgeAOT
  )
endif()

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
  pthread
)

target_link_options(wasmedge-tensorflow
  PUBLIC
  "-static-libstdc++"
)

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
  pthread
)

target_include_directories(wasmedge-tensorflow-lite
  PUBLIC
  ${TENSORFLOW_INCLUDE}
)

target_link_options(wasmedge-tensorflow-lite
  PUBLIC
  "-static-libstdc++"
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