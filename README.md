# WasmEdge Tools for Tensorflow Extension

The [WasmEdge](https://github.com/WasmEdge/WasmEdge) (formerly `SSVM`) is a high performance WebAssembly runtime optimized for server side applications. This project provides WasmEdge tools accessing with [Tensorflow C library](https://www.tensorflow.org/install/lang_c).

## Getting Started

### Install the WasmEdge-Tensorflow-Tools dependency

```bash
wget -qO- https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- -e all -v 0.9.0-rc.2
```

### Install WasmEdge-Tensorflow-Tools

```bash
wget https://github.com/second-state/WasmEdge-tensorflow-tools/releases/download/0.9.0-rc.2/WasmEdge-tensorflow-tools-0.9.0-rc.2-manylinux2014_x86_64.tar.gz
tar -zxvf WasmEdge-tensorflow-tools-0.9.0-rc.2-manylinux2014_x86_64.tar.gz
```

We provide a simple tool for showing the tensors information of TensorFlow-Lite models.
Take the [bird v1 tflite model](https://github.com/second-state/wasm-learning/blob/master/rust/birds_v1/lite-model_aiy_vision_classifier_birds_V1_3.tflite) for example.

```bash
./show-tflite-tensor lite-model_aiy_vision_classifier_birds_V1_3.tflite
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

## Build

### Get Source Code

```bash
$ git clone https://github.com/second-state/WasmEdge-tensorflow-tools.git
$ cd WasmEdge-tensorflow-tools
$ git checkout 0.9.0-rc.1
```

### Check Dependencies

WasmEdge will try to use the latest LLVM release to create our nightly build. 
If you want to build from source, you may need to install these dependencies 
by yourself or using our docker images which provides several Linux distribution 
support.

- LLVM 12.0.0 (>= 10.0.0), because LLVM 13 has breaking changes, WasmEdge project is not support LLVM 13 now.
- GCC 11.1.0 (>= 9.4.0)

### Prepare the Environment

#### Option 1: Use Docker Images

Repository on dockerhub `wasmedge/wasmedge`

You can use the following commands to get our latest docker image:

```bash
$ docker pull wasmedge/wasmedge # Equals to wasmedge/wasmedge:latest
```

The followings are available tags of docker images to build and run `WasmEdge-Tensorflow-Tools`:

| tag name                | arch    | based operating system | LLVM version | ENVs                  | compatibility            | comments                                                                            |
| ---                     | ---     | ---                    | ---          | ---                   | ---                      | ---                                                                                 |
| `latest`                | x86\_64 | Ubuntu 20.04 LTS       | 12.0.0       | CC=clang, CXX=clang++ | Ubuntu 20.04+            | This is for CI, will always use the latest Ubuntu release                           |
| `ubuntu-build-gcc`      | x86\_64 | Ubuntu 20.04 LTS       | 12.0.0       | CC=gcc, CXX=g++       | Ubuntu 20.04+            | This is for CI, will always use the latest Ubuntu release                           |
| `ubuntu-build-clang`    | x86\_64 | Ubuntu 20.04 LTS       | 12.0.0       | CC=clang, CXX=clang++ | Ubuntu 20.04+            | This is for CI, will always use the latest Ubuntu release                           |
| `manylinux2014_x86_64`  | x86\_64 | CentOS 7, 7.9.2009     | 12.0.0       | CC=gcc, CXX=g++       | Ubuntu 16.04+, CentOS 7+ | This is for developers who familiar with CentOS on x86\_64 architecture             |

#### Option 2: Install dependencies on Ubuntu 20.04 manually

```bash
# Tools and libraries
$ sudo apt install -y \
	software-properties-common \
	cmake \
	libboost-all-dev

# And you will need to install llvm for wasmedgec tool
$ sudo apt install -y \
	llvm-12-dev \
	liblld-12-dev

# WasmEdge supports both clang++ and g++ compilers
# You can choose one of them for building this project
# If you prefer GCC
$ sudo apt install -y gcc g++
# Else you can choose clang
$ sudo apt install -y clang-12
```

### Build WasmEdge-Tensorflow-Tools

`WasmEdge-Tensorflow-Tools` depends on `WasmEdge-core`, `WasmEdge-image`, and `WasmEdge-tensorflow`.
You have to prepare the `WasmEdge-core`, `WasmEdge-image`, and the `WasmEdge-tensorflow` before you build `WasmEdge-Tensorflow-Tools`.
We provides two options for setting up the `WasmEdge-core`:

#### Create and Enter the Build Folder

```bash
# After pulling our WasmEdge docker image
$ docker run -it --rm \
    -v <path/to/your/WasmEdge-tensorflow/source/folder>:/root/WasmEdge-tensorflow-tools \
    wasmedge/wasmedge:latest
(docker)$ cd /root/WasmEdge-tensorflow-tools
(docker)$ mkdir -p build && cd build
```

#### Option 1. Use built-in CMakeLists to get WasmEdge-Core, WasmEdge-image, and WasmEdge-tensorflow (Recommended)

```bash
(docker)$ cmake -DCMAKE_BUILD_TYPE=Release .. && make
```

#### Option 2. Use specific version of WasmEdge-Core by giving WASMEDGE_CORE_PATH

You can also specify the `WasmEdge-image` by assigning `WASMEDGE_IMAGE_PATH`, and the `WasmEdge-tensorflow` by assigning `WASMEDGE_TENSORFLOW_PATH`. The dependencies not assigned will be fetched with the default versions automatically.

```bash
(docker)$ cmake \
          -DWASMEDGE_CORE_PATH=<path/to/WasmEdge/source> \
          -DWASMEDGE_IMAGE_PATH=<path/to/WasmEdge-image/source> \
          -DWASMEDGE_TENSORFLOW_PATH=<path/to/WasmEdge-tensorflow/source> \
          -DCMAKE_BUILD_TYPE=Release .. && make
```

The executable `build/tools/wasmedge-tensorflow` is the runner for executing WASM or compiled WASM.
The executable `build/tools/wasmedge-tensorflow-lite` is the runner for executing WASM or compiled WASM with only TensorFlow-lite host functions supported.
