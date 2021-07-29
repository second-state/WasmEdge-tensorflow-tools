# WasmEdge Tools for Tensorflow Extension

The [WasmEdge](https://github.com/WasmEdge/WasmEdge) (formerly `SSVM`) is a high performance WebAssembly runtime optimized for server side applications. This project provides WasmEdge tools accessing with [Tensorflow C library](https://www.tensorflow.org/install/lang_c).

# Getting Started

## Prepare the environment

### Use our docker image (Recommanded)

Our docker image is based on `ubuntu 20.04`.

```bash
$ docker pull wasmedge/wasmedge
```

### Or setup the environment manually

Please notice that WasmEdge-Tensorflow-Tools requires cmake>=3.11 and libboost>=1.68.

```bash
# Tools and libraries
$ sudo apt install -y \
  software-properties-common \
  cmake \
  libboost-all-dev

# And you will need to install llvm for wasmedgec tool
$ sudo apt install -y \
  llvm-dev \
  liblld-10-dev

# WasmEdge supports both clang++ and g++ compilers
# You can choose one of them for building this project
$ sudo apt install -y gcc g++
$ sudo apt install -y clang
```

## Get WasmEdge-Tensorflow-Tools Source Code

```bash
$ git clone git@github.com:second-state/WasmEdge-tensorflow-tools.git
$ cd WasmEdge-tensorflow-tools
$ git checkout 0.8.2-rc2
```

## Build WasmEdge-Tensorflow-Tools

`WasmEdge-Tensorflow-Tools` depends on `WasmEdge-core`, `WasmEdge-image`, and `WasmEdge-tensorflow`.
You have to prepare the `WasmEdge-core`, `WasmEdge-image`, and the `WasmEdge-tensorflow` before you build `WasmEdge-Tensorflow-Tools`.
We provides two options for setting up the `WasmEdge-core`:

### Create and Enter the Build Folder

```bash
# After pulling our WasmEdge docker image
$ docker run -it --rm \
    -v <path/to/your/WasmEdge-tensorflow/source/folder>:/root/WasmEdge-tensorflow-tools \
    wasmedge/wasmedge:latest
(docker)$ cd /root/WasmEdge-tensorflow-tools
(docker)$ mkdir -p build && cd build
```

### Option 1. Use built-in CMakeLists to get WasmEdge-Core, WasmEdge-image, and WasmEdge-tensorflow (Recommended)

```bash
(docker)$ cmake -DCMAKE_BUILD_TYPE=Release .. && make
```

### Option 2. Use specific version of WasmEdge-Core by giving WASMEDGE_CORE_PATH

You can also specify the `WasmEdge-image` by assigning `WASMEDGE_IMAGE_PATH`, and the `WasmEdge-tensorflow` by assigning `WASMEDGE_TENSORFLOW_PATH`. The dependencies not assigned will be fetched with the default versions automatically.

```bash
(docker)$ cmake \
          -DWASMEDGE_CORE_PATH=<path/to/WasmEdge/source> \
          -DWASMEDGE_IMAGE_PATH=<path/to/WasmEdge-image/source> \
          -DWASMEDGE_TENSORFLOW_PATH=<path/to/WasmEdge-tensorflow/source> \
          -DCMAKE_BUILD_TYPE=Release .. && make
```

### If you don't want to build Ahead-of-Time runtime/compiler

If users don't need Ahead-of-Time runtime/compiler support, they can set the CMake option `WASMEDGE_DISABLE_AOT_RUNTIME` to `ON`.

```bash
(docker)$ cmake -DCMAKE_BUILD_TYPE=Release -DWASMEDGE_DISABLE_AOT_RUNTIME=ON ..
```

The executable `build/tools/wasmedgec-tensorflow` is the AOT compiler for WASM files.
The executable `build/tools/wasmedge-tensorflow` is the runner for executing WASM or compiled WASM.
The executable `build/tools/wasmedge-tensorflow-lite` is the runner for executing WASM or compiled WASM with only TensorFlow-lite host functions supported.

## Run WasmEdge-tensorflow-tools

```bash
wget https://github.com/second-state/WasmEdge-tensorflow-tools/releases/download/0.8.2-rc2/WasmEdge-tensorflow-tools-0.8.2-rc2-manylinux2014_x86_64.tar.gz
tar -zxvf WasmEdge-tensorflow-tools-0.8.2-rc2-manylinux2014_x86_64.tar.gz
# Download the required shared libraries and make the symbolic links.
./download_dependencies_all.sh
LD_LIBRARY_PATH=. ./wasmedge-tensorflow
# For developers want to use TensorFlow-Lite only
./download_dependencies_tflite.sh
LD_LIBRARY_PATH=. ./wasmedge-tensorflow-lite
```

We provide a simple tool for showing the tensors information of TensorFlow-Lite models.
Take the [bird v1 tflite model](https://github.com/second-state/wasm-learning/blob/master/rust/birds_v1/lite-model_aiy_vision_classifier_birds_V1_3.tflite) for example.

```bash
LD_LIBRARY_PATH=. ./show-tflite-tensor lite-model_aiy_vision_classifier_birds_V1_3.tflite
```

The output will be as following:

```bash
Input tensor nums: 1
    Input tensor name: module/hub_input/images_uint8
        dimensions: [1 , 224 , 224 , 3]
        data type: UInt8
        tensor byte size: 150528
Output tensor nums: 1
    Output tensor name: module/prediction
        dimensions: [1 , 965]
        data type: UInt8
        tensor byte size: 965
```
