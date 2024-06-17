# 第十章。使用 GPU 和其他特殊资源的 Dask

有时解决我们的扩展问题的答案并不是增加更多计算机，而是投入不同类型的资源。一个例子是一万只猴子试图复制莎士比亚的作品，与一个莎士比亚¹。尽管性能有所不同，但一些基准测试显示，使用 GPU 而不是 CPU 在模型训练时间上的提升可以高达 85%。继续其模块化传统，Dask 的 GPU 逻辑存在于围绕其构建的库和生态系统中。这些库可以在一组 GPU 工作节点上运行，也可以在一个主机上的不同 GPU 上并行工作。

我们在计算机上进行的大部分工作都是由 CPU 完成的。GPU 最初是用于显示视频，但涉及大量矢量化浮点（例如非整数）运算。通过矢量化运算，相同的操作并行应用于大量数据集，类似于`map`操作。张量处理单元（TPU）类似于 GPU，但不用于图形显示。

对于我们在 Dask 中的目的，我们可以将 GPU 和 TPU 视为专门用于卸载大规模矢量化计算的设备，但还有许多其他类型的加速器。虽然本章的大部分内容集中在 GPU 上，但相同的一般技术通常也适用于其他加速器，只是使用了不同的库。其他类型的特殊资源包括 NVMe 驱动器、更快（或更大）的 RAM、TCP/IP 卸载、Just-a-Bunch-of-Disks 扩展端口以及英特尔的 OPTAIN 内存。特殊资源/加速器可以改善从网络延迟到大文件写入到磁盘的各个方面。所有这些资源的共同点在于，Dask 并没有内置对这些资源的理解，您需要为 Dask 调度器提供这些信息，并利用这些资源。

本章将探讨 Python 中加速分析的当前状态以及如何与 Dask 结合使用这些工具。您将了解到哪些问题适合使用 GPU 加速，以及其他类型加速器的相关信息，以及如何将这些知识应用到您的问题中。

###### 警告

由于挖掘加密货币的相对简易性，云账户和拥有 GPU 访问权限的机器特别受到互联网上不良分子的关注。如果您习惯于仅使用公共数据和宽松的安全控制，这是一个审视您的安全流程并限制运行时访问仅限于需要的人的机会。否则可能会面临巨额云服务账单。

# 透明与非透明加速器

加速器主要分为两类：透明（无需代码或更改）和非透明优化器。加速器是否透明在很大程度上取决于我们在堆栈下面的某人是否使其对我们透明化。

在用户空间级别，TCP/IP 卸载通常是透明的，这意味着操作系统为我们处理它。NVMe 驱动器通常也是透明的，通常看起来与传统磁盘相同，但速度更快。仍然很重要的是让 Dask 意识到透明优化器；例如，应将磁盘密集型工作负载安排在具有更快磁盘的机器上。

非透明加速器包括 GPUs、Optane、QAT 等。使用它们需要修改我们的代码以充分利用它们。有时这可能只需切换到不同的库，但并非总是如此。许多非透明加速器要求要么复制我们的数据，要么进行特殊格式化才能运行。这意味着如果一个操作相对较快，转移到优化器可能会使其变慢。

# 理解 GPU 或 TPU 是否有助于解决问题

并非每个问题都适合 GPU 加速。 GPUs 特别擅长在同一时间对大量数据点执行相同的计算。如果一个问题非常适合矢量化计算，那么 GPU 可能非常适合。

一些通常受益于 GPU 加速的常见问题包括：

+   机器学习

+   线性代数

+   物理模拟

+   图形（毫不意外）

GPUs 不太适合分支繁多且非矢量化的工作流程，或者数据复制成本与计算成本相似或更高的工作流程。

# 使 Dask 资源感知

如果您已经确定您的问题非常适合专用资源，下一步是让调度器意识到哪些机器和进程拥有该资源。您可以通过添加环境变量或命令行标志到工作进程启动（例如 `--resources "GPU=2"` 或 `DASK_DISTRIBUTED_WORKER_RESOURCES_GPU=2`）来实现这一点。

对于 NVIDIA 用户来说，`dask-cuda` 软件包可以每个 GPU 启动一个工作进程，将 GPU 和线程捆绑在一起以提升性能。例如，在我们的带有 GPU 资源的 Kubernetes 集群上，我们配置工作进程使用 `dask-cuda-worker` 启动器，如 示例 10-1 所示。

##### 示例 10-1\. 在 Dask Kubernetes 模板中使用 `dask-cuda-worker` 软件包

```py
worker_template = make_pod_spec(image='holdenk/dask:latest',
                                memory_limit='8G', memory_request='8G',
                                cpu_limit=1, cpu_request=1)
worker_template.spec.containers[0].resources.limits["gpu"] = 1
worker_template.spec.containers[0].resources.requests["gpu"] = 1
worker_template.spec.containers[0].args[0] = "dask-cuda-worker --resources 'GPU=1'"
worker_template.spec.containers[0].env.append("NVIDIA_VISIBLE_DEVICES=ALL")
# Or append --resources "GPU=2"
```

在这里，我们仍然添加 `--resources` 标志，以便在混合环境中仅选择 GPU 工作进程。

如果您使用 Dask 在单台计算机上安排多个 GPU 的工作（例如使用带有 CUDA 的 Dask 本地模式），同样的 `dask-cuda` 软件包提供了 `LocalCUDACluster`。与 `dask-cuda-worker` 类似，您仍然需要手动添加资源标记，如 示例 10-2 所示，但 `LocalCUDACluster` 可以启动正确的工作进程并将它们固定在线程上。

##### 示例 10-2\. 带有资源标记的 `LocalCUDACluster`

```py
from dask_cuda import LocalCUDACluster
from dask.distributed import Client
#NOTE: The resources= flag is important; by default the 
# LocalCUDACluster *does not* label any resources, which can make
# porting your code to a cluster where some workers have GPUs and 
# some don't painful.
cluster = LocalCUDACluster(resources={"GPU": 1})
client = Client(cluster)
```

###### 注

对于均质集群来说，可能会有诱惑避免标记这些资源，但除非您始终将工作进程/线程与加速器进行 1:1 映射（或加速器可以同时被所有工作进程使用），否则标记这些资源仍然是有益的。这对于像 GPU/TPU 这样的非共享（或难以共享）资源尤为重要，因为 Dask 可能会安排两个试图访问 GPU 的任务。但对于像 NVMe 驱动器或 TCP/IP 卸载这样的共享资源，如果它在集群中的每个节点上都存在且始终存在，则可能可以跳过标记。

需要注意的是，Dask 不管理自定义资源（包括 GPU）。如果另一个进程在没有经过 Dask 请求的情况下使用了所有 GPU 核心，这是无法保护的。在某些方面，这类似于早期的计算方式，即我们使用“协作式”多任务处理；我们依赖于邻居的良好行为。

###### 警告

Dask 依赖于行为良好的 Python 代码，即不会使用未请求的资源，并在完成时释放资源。这通常发生在内存泄漏（包括加速和非加速）时，尤其是在像 CUDA 这样的专门库中分配 Python 之外的内存。这些库通常在完成任务后需要调用特定步骤，以便让资源可供其他人使用。

# 安装库

现在 Dask 已经意识到集群上的特殊资源，是时候确保您的代码能够利用这些资源了。通常情况下，这些加速器会要求安装某种特殊库，可能需要较长的编译时间。在可能的情况下，从 conda 安装加速库，并在工作节点（容器或主机上）预先安装，可以帮助减少这种开销。

对于 Kubernetes（或其他 Docker 容器用户），您可以通过创建一个预先安装了加速器库的自定义容器来实现这一点，如示例 10-3 所示。

##### 示例 10-3\. 预安装 cuDF

```py
# Use the Dask base image; for arm64, though, we have to use custom built
# FROM ghcr.io/dask/dask
FROM holdenk/dask:latest

# arm64 channel
RUN conda config --add channels rpi
# Numba and conda-forge channels
RUN conda config --add channels numba
RUN conda config --add channels conda-forge
# Some CUDA-specific stuff
RUN conda config --add channels rapidsai
# Accelerator libraries often involve a lot of native code, so it's
# faster to install with conda
RUN conda install numba -y
# GPU support (NV)
RUN conda install cudatoolkit -y
# GPU support (AMD)
RUN conda install roctools -y || echo "No roc tools on $(uname -a)"
# A lot of GPU acceleration libraries are in the rapidsai channel
# These are not installable with pip
RUN conda install cudf -y
```

然后，为了构建这个，我们运行示例 10-4 中显示的脚本。

##### 示例 10-4\. 构建自定义 Dask Docker 容器

```py
#/bin/bash
set -ex

docker buildx build -t holdenk/dask-extended  --platform \
  linux/arm64,linux/amd64 --push . -f Dockerfile
docker buildx build -t holdenk/dask-extended-notebook  --platform \
  linux/arm64,linux/amd64 --push . -f NotebookDockerfile
```

# 在您的 Dask 任务中使用自定义资源

您必须确保需要加速器的任务在具有加速器的工作进程上运行。您可以在使用 Dask 调度任务时通过 `client.submit` 显式地请求特殊资源，如示例 10-5 所示，或通过向现有代码添加注释，如示例 10-6 所示。

##### 示例 10-5\. 提交一个请求使用 GPU 的任务

```py
future = client.submit(how_many_gpus, 1, resources={'GPU': 1})
```

##### 示例 10-6\. 注释需要 GPU 的一组操作

```py
with dask.annotate(resources={'GPU': 1}):
    future = client.submit(how_many_gpus, 1)
```

如果从具有 GPU 资源的集群迁移到没有 GPU 资源的集群，则此代码将无限期挂起。后面将介绍的 CPU 回退设计模式可以缓解这个问题。

## 装饰器（包括 Numba）

Numba 是一个流行的高性能 JIT 编译库，也支持各种加速器。 大多数 JIT 代码以及许多装饰器函数通常不会直接序列化，因此尝试直接使用`dask.submit` Numba（如示例 10-7 所示）不起作用。 相反，正确的方法是像示例 10-8 中所示那样包装函数。

##### 示例 10-7\. 装饰器难度

```py
# Works in local mode, but not distributed
@dask.delayed
@guvectorize(['void(float64[:], intp[:], float64[:])'],
             '(n),()->(n)')
def delayed_move_mean(a, window_arr, out):
    window_width = window_arr[0]
    asum = 0.0
    count = 0
    for i in range(window_width):
        asum += a[i]
        count += 1
        out[i] = asum / count
    for i in range(window_width, len(a)):
        asum += a[i] - a[i - window_width]
        out[i] = asum / count

arr = np.arange(20, dtype=np.float64).reshape(2, 10)
print(arr)
print(dask.compute(delayed_move_mean(arr, 3)))
```

##### 示例 10-8\. 装饰器技巧

```py
@guvectorize(['void(float64[:], intp[:], float64[:])'],
             '(n),()->(n)')
def move_mean(a, window_arr, out):
    window_width = window_arr[0]
    asum = 0.0
    count = 0
    for i in range(window_width):
        asum += a[i]
        count += 1
        out[i] = asum / count
    for i in range(window_width, len(a)):
        asum += a[i] - a[i - window_width]
        out[i] = asum / count

arr = np.arange(20, dtype=np.float64).reshape(2, 10)
print(arr)
print(move_mean(arr, 3))

def wrapped_move_mean(*args):
    return move_mean(*args)

a = dask.delayed(wrapped_move_mean)(arr, 3)
```

###### 注意

示例 10-7 在本地模式下可以工作，但在扩展时不能工作。

## GPU

像 Python 中的大多数任务一样，有许多不同的库可用于处理 GPU。 这些库中的许多支持 NVIDIA 的计算统一设备架构（CUDA），并试验性地支持 AMD 的新开放 HIP / Radeon Open Compute 模块（ROCm）接口。 NVIDIA 和 CUDA 是第一个出现的，并且比 AMD 的 Radeon Open Compute 模块采用得多，以至于 ROCm 主要集中于支持将 CUDA 软件移植到 ROCm 平台上。

我们不会深入探讨 Python GPU 库的世界，但您可能想查看[Numba 的 GPU 支持](https://oreil.ly/i-FVO)、[TensorFlow 的 GPU 支持](https://oreil.ly/vChSG)和[PyTorch 的 GPU 支持](https://oreil.ly/sdLjo)。

大多数具有某种形式 GPU 支持的库都需要编译大量非 Python 代码。 因此，通常最好使用 conda 安装这些库，因为 conda 通常具有更完整的二进制包装，允许您跳过编译步骤。

# 构建在 Dask 之上的 GPU 加速

扩展 Dask 的三个主要 CUDA 库是 cuDF（之前称为 dask-cudf）、BlazingSQL 和 cuML。² 目前这些库主要关注的是 NVIDIA GPU。

###### 注意

Dask 目前没有任何库来支持与 OpenCL 或 HIP 的集成。 这并不妨碍您使用支持它们的库（如 TensorFlow）来使用 GPU，正如前面所示。

## cuDF

[cuDF](https://oreil.ly/BZ9x2)是 Dask 的 DataFrame 库的 GPU 加速版本。 一些[基准测试显示性能提升了 7 倍到 50 倍](https://oreil.ly/unpvl)。 并非所有的 DataFrame 操作都会有相同的速度提升。 例如，如果您是逐行操作而不是矢量化操作，那么使用 cuDF 而不是 Dask 的 DataFrame 库时可能会导致性能较慢。 cuDF 支持您可能使用的大多数常见数据类型，但并非所有数据类型都支持。

###### 注意

在内部，cuDF 经常将工作委托给 cuPY 库，但由于它是由 NVIDIA 员工创建的，并且他们的重点是支持 NVIDIA 硬件，因此 cuDF 不直接支持 ROCm。

## BlazingSQL

BlazingSQL 使用 GPU 加速来提供超快的 SQL 查询。 BlazingSQL 在 cuDF 之上运行。

###### 注意

虽然 BlazingSQL 是一个很棒的工具，但它的大部分文档都有问题。例如，在撰写本文时，主 README 中链接的所有示例都无法正确解析，而且文档站点完全处于离线状态。

## cuStreamz

另一个 GPU 加速的流式处理库是 cuStreamz，它基本上是 Dask 流式处理和 cuDF 的组合；我们在附录 D 中对其进行了更详细的介绍。

# 释放加速器资源

在 GPU 上分配内存往往很慢，因此许多库会保留这些资源。在大多数情况下，如果 Python VM 退出，资源将被清理。最后的选择是使用 `client.restart` 弹出所有工作进程。在可能的情况下，手动管理资源是最好的选择，这取决于库。例如，cuPY 用户可以通过调用 `free_all_blocks()` 来释放所使用的块，如[内存管理文档](https://oreil.ly/hpxkg)所述。

# 设计模式：CPU 回退

CPU 回退是指尝试使用加速器，如 GPU 或 TPU，如果加速器不可用，则回退到常规 CPU 代码路径。在大多数情况下，这是一个好的设计模式，因为加速器（如 GPU）可能很昂贵，并且可能不总是可用。然而，在某些情况下，CPU 和 GPU 性能之间的差异是如此之大，以至于回退到 CPU 很难在实际时间内成功；这在深度学习算法中最常发生。

面向对象编程和鸭子类型在这种设计模式下有些适合，因为只要两个类实现了您正在使用的相同接口的部分，您就可以交换它们。然而，就像在 Dask DataFrames 中替换 pandas DataFrames 一样，它并不完美，特别是在性能方面。

###### 警告

在一个更好的世界里，我们可以提交一个请求 GPU 资源的任务，如果没有被调度，我们可以切换回 CPU-only 资源。不幸的是，Dask 的资源调度更接近于“尽力而为”，³ 因此我们可能被调度到没有我们请求的资源的节点上。

# 结论

专用加速器，如 GPU，可能会对您的工作流程产生很大影响。选择适合您工作流程的正确加速器很重要，有些工作流程不太适合加速。Dask 不会自动使用任何加速器，但有各种库可用于 GPU 计算。许多这些库创建时并没有考虑到共享计算的概念，因此要特别注意意外资源泄漏，特别是由于 GPU 资源往往更昂贵。

¹ 假设莎士比亚仍然活着，但事实并非如此。

² BlazingSQL 可能已接近其生命周期的尽头；已经有相当长一段时间没有提交更新，而且其网站看起来就像那些上世纪 90 年代的 GeoCities 网站一样简陋。

³ 这并没有像[文档中](https://oreil.ly/p1Ldf)描述的那样详细，因此可能在未来发生变化。
