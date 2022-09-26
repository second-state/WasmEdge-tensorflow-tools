### 0.11.1-alpha.1 (2022-09-26)

This is the tools extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.11.1-alpha.1](https://github.com/WasmEdge/WasmEdge/releases/tag/0.11.1-alpha.1) for more details.

Features:

* Update the `WasmEdge` dependency to `0.11.1-alpha.1`.

### 0.11.0 (2022-08-31)

This is the tools extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.11.0](https://github.com/WasmEdge/WasmEdge/releases/tag/0.11.0) for more details.

Features:

* Update the `WasmEdge` dependency to `0.11.0`.

### 0.10.1 (2022-07-28)

This is the tools extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.10.1](https://github.com/WasmEdge/WasmEdge/releases/tag/0.10.1) for more details.

Features:

* Update the `WasmEdge` dependency to `0.10.1`.
* Supported the WASM `threads` proposal.
  * Users can use the `--enable-threads` to enable the proposal.

### 0.10.0 (2022-05-26)

This is the tools extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.10.0](https://github.com/WasmEdge/WasmEdge/releases/tag/0.10.0) for more details.

Features:

* Update the `WasmEdge` dependency to `0.10.0`.
* Added the Darwin x86_64 support for `wasmedge-tensorflow` and `wasmedge-tensorflow-lite`.
* Supported the WASM `tail-call` proposal.
  * Users can use the `--enable-tail-call` to enable the proposal.
* Supported the WASM `extended-const` proposal.
  * Users can use the `--enable-extended-const` to enable the proposal.

### 0.9.1 (2022-02-10)

This is the tools extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.9.1](https://github.com/WasmEdge/WasmEdge/releases/tag/0.9.1) for more details.

Features:

* Added the copyright text.
* Update the `WasmEdge` dependency to `0.9.1`.
* Supported the interruptible execution.
  * Users can use the `--time-limit` to assign timeout value.
* Supported the WASM `multi-memories` proposal.
  * Users can use the `--enable-multi-memory` to enable the proposal.
* Enabled the gas limitation.
  * Users can use the `--gas-limit` to assign the limitation of costs.
* Added the Linux aarch64 support for `wasmedge-tensorflowlite`.
* Added the Android aarch64 support for `wasmedge-tensorflowlite`.

### 0.9.0 (2021-12-09)

This is the tools extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.9.0](https://github.com/WasmEdge/WasmEdge/releases/tag/0.9.0) for more details.

Features:

* Update the `WasmEdge` dependency to `0.9.0`.
* Remove the tool `wasmedger-tensorflow`. Please use `wasmedgec` in [WasmEdge](https://github.com/WasmEdge/WasmEdge) project instead.

### 0.8.2 (2021-09-06)

This is the tools extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
Please refer to the [WasmEdge 0.8.2](https://github.com/WasmEdge/WasmEdge/releases/tag/0.8.2) for more details.

Features:

* Update the `WasmEdge` dependency to `0.8.2`.
* Modified the CMake option `DISABLE_AOT_RUNTIME` to `WASMEDGE_BUILD_AOT_RUNTIME` for enabling AOT compiler compilation (`ON` by default).
* Added the `generic-binary` option for the AOT compiler for generating the generic binary.

### 0.8.1 (2021-06-22)

This is the tools extension for [WasmEdge](https://github.com/WasmEdge/WasmEdge).
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
