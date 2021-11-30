### 0.9.0-rc.5 (2021-11-30)

This is the host function extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.9.0-rc.5](https://github.com/WasmEdge/WasmEdge/releases/tag/0.9.0-rc.5) for more details.

Features:

* Update the `WasmEdge` dependency to `0.9.0-rc.5`.
* Remove the tool `wasmedger-tensorflow`. Please use `wasmedgec` in [WasmEdge](https://github.com/WasmEdge/WasmEdge) project instead.

### 0.9.0-rc.4 (2021-11-23)

This is the host function extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.9.0-rc.4](https://github.com/WasmEdge/WasmEdge/releases/tag/0.9.0-rc.4) for more details.

Features:

* Update the `WasmEdge` dependency to `0.9.0-rc.4`.
* Remove the tool `wasmedger-tensorflow`. Please use `wasmedgec` in [WasmEdge](https://github.com/WasmEdge/WasmEdge) project instead.

### 0.9.0-rc.3 (2021-11-15)

This is the host function extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.9.0-rc.3](https://github.com/WasmEdge/WasmEdge/releases/tag/0.9.0-rc.3) for more details.

Features:

* Update the `WasmEdge` dependency to `0.9.0-rc.3`.
* Remove the tool `wasmedger-tensorflow`. Please use `wasmedgec` in [WasmEdge](https://github.com/WasmEdge/WasmEdge) project instead.

### 0.9.0-rc.2 (2021-11-02)

This is the host function extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.9.0-rc.2](https://github.com/WasmEdge/WasmEdge/releases/tag/0.9.0-rc.2) for more details.

Features:

* Update the `WasmEdge` dependency to `0.9.0-rc.2`.
* Remove the tool `wasmedger-tensorflow`. Please use `wasmedgec` in [WasmEdge](https://github.com/WasmEdge/WasmEdge) project instead.

### 0.9.0-rc.1 (2021-11-02)

This is the host function extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.9.0-rc.2](https://github.com/WasmEdge/WasmEdge/releases/tag/0.9.0-rc.2) for more details.

Features:

* Update the `WasmEdge` dependency to `0.9.0-rc.2`.
* Remove the tool `wasmedger-tensorflow`. Please use `wasmedgec` in [WasmEdge](https://github.com/WasmEdge/WasmEdge) project instead.

### 0.8.2 (2021-09-06)

This is the host function extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.8.2](https://github.com/WasmEdge/WasmEdge/releases/tag/0.8.2) for more details.

Features:

* Update the `WasmEdge` dependency to `0.8.2`.
* Modified the CMake option `DISABLE_AOT_RUNTIME` to `WASMEDGE_BUILD_AOT_RUNTIME` for enabling AOT compiler compilation (`ON` by default).
* Added the `generic-binary` option for the AOT compiler for generating the generic binary.

### 0.8.2-rc2 (2021-07-29)

This is the host function extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.8.2-rc.2](https://github.com/WasmEdge/WasmEdge/releases/tag/0.8.2-rc.2) for more details.

Features:

* Update the `WasmEdge` dependency to `0.8.2-rc.2`.
* Modified the CMake option `DISABLE_AOT_RUNTIME` to `WASMEDGE_BUILD_AOT_RUNTIME` for enabling AOT compiler compilation (`ON` by default).
* Added the `generic-binary` option for the AOT compiler for generating the generic binary.

### 0.8.1 (2021-06-22)

This is the host function extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.8.1](https://github.com/WasmEdge/WasmEdge/releases/tag/0.8.1) for more details.

Features:

* Update the `WasmEdge`, `WasmEdge-image`, and `WasmEdge-tensorflow` dependencies to `0.8.1`.

### 0.8.0 (2021-05-14)

This is the tools extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.8.0](https://github.com/WasmEdge/WasmEdge/releases/tag/0.8.0) for more details.

Features:

* Moved tools from [WasmEdge-Tensorflow](https://github.com/second-state/WasmEdge-tensorflow).
* Renamed this project to `WasmEdge-tensorflow-tools` and updated the `WasmEdge` dependency.
* Add release and build CI.
* Renamed the CMake options.
  * Option `BUILD_AOT_RUNTIME` (formerly `SSVM_DISABLE_AOT_RUNTIME` and `OFF` by default), which is `ON` by default, is for enabling the compilation of the ahead-of-Time compiler.

### 0.8.0-rc2 (2021-05-10)

This is the tools extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.8.0](https://github.com/WasmEdge/WasmEdge/releases/tag/0.8.0) for more details.

Features:

* Moved tools from [WasmEdge-Tensorflow](https://github.com/second-state/WasmEdge-tensorflow).
* Renamed this project to `WasmEdge-tensorflow-tools` and updated the `WasmEdge` dependency.
* Add release and build CI.

### 0.8.0-rc1 (2021-04-30)

This is the tools extension for [SSVM](https://github.com/second-state/SSVM).
Please refer to the [SSVM 0.7.3](https://github.com/second-state/SSVM/releases/tag/0.7.3) for more details.

Features:

* Moved tools from [SSVM-Tensorflow](https://github.com/second-state/ssvm-tensorflow).
* Update the implementation of runners to the newest `SSVM`.
