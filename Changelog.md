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
