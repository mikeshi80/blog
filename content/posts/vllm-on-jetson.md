+++
title = '在Jetson上安装vLLM'
date = 2024-07-30T14:52:42+08:00
summary = '如果想用 vllm，官方提供的 PyTorch 二进制安装包是不行的，因为它需要用到 distributed 以及 NCCL 的支持，官方提供的包中是不包含这些特性的，甚至一些民间爱好者提供的安装包也只支持了 distributed 而未支持 NCCL，所以需要自行安装。本文介绍了如何在 Jetson Orin AGX 下编译相关 vLLM 软件。'
+++

# Jetson Orin AGX 下编译相关 vLLM 软件

# 编译 PyTorch

如果想用 vllm，官方提供的 PyTorch 二进制安装包是不行的，因为它需要用到 distributed 以及 NCCL 的支持，官方提供的包中是不包含这些特性的，甚至一些民间爱好者提供的安装包也只支持了 distributed 而未支持 NCCL，所以需要自行安装。

## 下载必要的工具

需要下载安装必要的工具，因为 Jetson Linux 是 Ubuntu 系统，所以直接使用 apt-get 即可。

```bash
sudo apt-get install python3-pip cmake libopenblas-dev libopenmpi-dev
```

注：Jetson Linux 自带开发相关的软件包，比如 git, gcc 等，如果是在普通的 PC 版 Linux 下，也可以编译，但需要执行多安装 `git build-essential`这两个 package。

## 克隆 PyTorch 代码

系统自带 Git，可以直接用以下命令克隆 PyTorch 代码

```bash
git clone --recursive http://github.com/pytorch/pytorch
```

由于默认克隆的是当前开发版本，所以通常需要指定一个版本，如 `v2.3.1`，则执行

```xml
git checkout v2.3.1
```

切换到该版本，也可直接在克隆项目时添加 `---branch v2.3.1`参数指定。

## 编译 PyTorch 代码

首先确保必须的 Python 包都有安装，并且为了不影响其他环境，先要利用 env 来创建一个虚拟环境。

```bash
$ pip3 install -r requirements.txt
$ pip3 install scikit-build
$ pip3 install ninja
```

编译代码需要设定一些环境变量，所以最方便的做法是编写一个编译脚本`build.sh`

```bash
export USE_NCCL=1
# export USE_SYSTEM_NCCL=0        # if you do not use system nccl, nvidia-smi may do not work
export USE_DISTRIBUTED=1                    # set 0 if you want to disable OpenMPI backend
export USE_QNNPACK=0
export USE_PYTORCH_QNNPACK=0
export TORCH_CUDA_ARCH_LIST="5.2;6.1;7.0;7.2;7.5;8.0;8.6;8.7"
export PYTORCH_BUILD_VERSION=2.3.1  # without the leading 'v', e.g. 2.3.0 for PyTorch v2.3.0
export PYTORCH_BUILD_NUMBER=1

python3 setup.py install
```

# 编译 vllm

vllm 需要先编译以下几个依赖库：

- triton
- vllm-flash-attn
  所以我们将一一介绍。

## Triton

triton 实际可以直接编译，但它需要下载大量文件，而这些文件是在境外服务器的，所以，最好是设置好代理服务器，比如

```bash
export http_proxy=http://localhost:7890
export https_proxy=http://localhost:7890
```

然后需要将项目 clone 到本地，因为 pypi.org 未提供代码，也只提供了常用平台的二进制安装包，所以直接 pip 是会报错说找不到相应的安装包的。可以按照以下命令

```bash
git clone https://github.com/triton-lang/triton.git;
cd triton;

pip install ninja cmake wheel; # build-time dependencies
pip install -e python
```

来进行安装。

注意，由于要下载好几个 G 的东西，所以非常耗时，需要做好心理准备。

## vllm-flash-attn

虽然实际上，没有`vllm-flash-attn`，vllm 也会使用内置的 xformers 支持来进行加速。但实际使用下来，在 Orin AGX 下，xformers 是无法正常运行的。所以必须安装`vll-flash-attn`。

但该库不能直接从 pypi.org 上下载编译，需要修改部分设置，所以先需要 clone 下来。

```bash
git clone --recurse-submodules https://github.com/vllm-project/flash-attention.git
```

然后修改 setup.py 中的两行：

一个是`if bare_metal_version >= Version("11.8"):`下，将 90 改成 87。即改为：

```python
cc_flag.append("arch=compute_87,code=sm_87")
```

因为现在的 vllm 是需要的 PyTorch 版本是 v2.3.0 的，而最新的已经需要 v2.3.1，所以需要修改后面`PYTORCH_VERSION`的值从 2.3.1 改成 2.3.0。

修改完之后执行

```bash
python setup.py install
```

进行安装。如果发生内存不足编译失败，可以加`MAX_JOBS`来限制最大的并行编译量，比如

```bash
MAX_JOBS=4 python setup.py install
```

来限制最大 4 进程编译。

## 编译 vllm

先将 vllm 的代码 clone 到本地

```bash
git clone --recurse-submodules https://github.com/vllm-project/vllm.git
```

修改 vllm 的 CMakeLists.txt，将其中的`CUDA_SUPPORTED_ARCHS`中，添加 8.7，可以改成如下：

```Plain Text
set(CUDA_SUPPORTED_ARCHS "7.0;7.5;8.0;8.6;8.7;8.9;9.0")
```

然后可以进行编译

```bash
python setup.py install
```

# 生成 wheel 安装包

注意，以上的示例都是最后执行`python setup.py install`直接进行安装的。但如果想生成 wheel 安装包供以后再次安装，或分发给其他用户，那么可以将其改为：

```bash
python setup.py bdist_wheel
```

这样就会在 dist 目录下生成二进制安装包，然后进入该目录，执行

```bash
pip install xxx.whl # xxx为该安装包名称
```
