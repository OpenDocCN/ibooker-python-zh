# 第十一章。使用 Ray 与 GPU 和加速器

虽然 Ray 主要专注于水平扩展，但有时使用像 GPU 这样的特殊加速器可能比仅仅投入更多“常规”计算节点更便宜和更快。GPU 特别适合执行向量化操作，一次对数据块执行相同操作。机器学习，以及更广泛的线性代数，是一些顶级用例，¹ 因为深度学习极易向量化。

通常情况下，GPU 资源比 CPU 资源更昂贵，因此 Ray 的架构使得在必要时仅需请求 GPU 资源变得更加容易。要利用 GPU，您需要使用专门的库，并且由于这些库涉及直接内存访问，它们的结果可能并不总是可串行化的。在 GPU 计算世界中，NVIDIA 和 AMD 是两个主要选项，具有不同的集成库。

# GPU 擅长什么？

并非每个问题都适合 GPU 加速。GPU 特别擅长同时在许多数据点上执行相同计算。如果一个问题非常适合向量化，那么 GPU 可能非常适合解决这个问题。

以下是从 GPU 加速中受益的常见问题：

+   机器学习

+   线性代数

+   物理学模拟

+   图形（这不奇怪）

GPU 不适合分支密集的非向量化工作流程，或者数据复制成本与计算成本相似或更高的工作流程。

# 构建模块

使用 GPU 需要额外的开销，类似于分发任务的开销（尽管速度稍快）。这些开销来自于数据串行化以及通信，尽管 CPU 和 GPU 之间的链接通常比网络链接更快。与 Ray 的分布式任务不同，GPU 没有 Python 解释器。相反，您的高级工具通常会生成或调用本机 GPU 代码。CUDA 和 Radeon Open Compute（ROCm）是与 GPU 交互的两个事实上的低级库，分别来自 NVIDIA 和 AMD。

NVIDIA 首先发布了 CUDA，并迅速在许多高级库和工具中获得了广泛应用，包括 TensorFlow。AMD 的 ROCm 起步较慢，并未见到同样程度的采纳。一些高级工具，包括 PyTorch，现在已经集成了 ROCm 支持，但许多其他工具需要使用特殊分支的 ROCm 版本，例如 TensorFlow（tensorflow-rocm）或 LAPACK（rocSOLVER）。

弄清楚构建模块可能会出人意料地具有挑战性。例如，在我们的经验中，让 NVIDIA GPU Docker 容器在 Linux4Tegra 上与 Ray 构建需要几天的时间。 ROCm 和 CUDA 库有支持特定硬件的特定版本，同样，您可能希望使用的更高级程序可能仅支持某些版本。如果您正在运行 Kubernetes 或类似的容器化平台，则可以从像 NVIDIA 的 [CUDA 映像](https://oreil.ly/klaV4) 或 AMD 的 [ROCm 映像](https://oreil.ly/IKLF9) 这样的预构建容器开始受益，作为基础。

# 更高级的库

除非您有特殊需求，否则您可能会发现与为您生成 GPU 代码的更高级库一起工作最容易，例如基本线性代数子程序（BLAS）、TensorFlow 或 Numba。您应尝试将这些库安装在您正在使用的基础容器或机器映像中，因为它们在安装期间通常需要大量编译时间。

一些库，例如 Numba，执行动态重写您的 Python 代码。要使 Numba 对您的代码起作用，您需要在函数中添加装饰器（例如 `@numba.jit`）。不幸的是，`numba.jit` 和其他对函数的动态重写在 Ray 中不受直接支持。相反，如果您使用此类库，只需像示例 11-1 中所示包装调用即可。

##### 示例 11-1\. [简单的 CUDA 示例](https://oreil.ly/xjpkD)

```py
from numba import cuda, float32

# CUDA kernel
@cuda.jit
def mul_two(io_array):
    pos = cuda.grid(1)
    if pos < io_array.size:
        io_array[pos] *= 2 # do the computation

@ray.remote
def remote_mul(input_array):
    # This implicitly transfers the array into the GPU and back, which is not free
    return mul_two(input_array)
```

###### 注意

与 Ray 的分布式函数类似，这些工具通常会为您处理数据复制，但重要的是要记住移动数据进出 GPU 不是免费的。由于这些数据集可能很大，大多数库尝试在相同数据上执行多个操作。如果您有一个重复使用数据的迭代算法，则使用 actor 来持有 GPU 资源并将数据保留在 GPU 中可以减少此成本。

无论您选择哪个库（或者是否决定编写自己的 GPU 代码），都需要确保 Ray 将您的代码调度到具有 GPU 的节点上。

# 获取和释放 GPU 和加速器资源

您可以通过将 `num_gpus` 添加到 `ray.remote` 装饰器来请求 GPU 资源，与内存和 CPU 的方式类似。与 Ray 中的其他资源（包括内存）一样，Ray 中的 GPU 不能保证，并且 Ray 不会自动为您清理资源。虽然 Ray 不会自动为您清理内存，但 Python（在一定程度上）会，这使得 GPU 泄漏比内存泄漏更有可能。

许多高级库在 Python VM 退出之前不会释放 GPU。您可以在每次调用后强制 Python VM 退出，从而释放任何 GPU 资源，方法是在您的 `ray.remote` 装饰器中添加 `max_calls=1`，如示例 11-2 所示。

##### 示例 11-2\. [请求和释放 GPU 资源](https://oreil.ly/xjpkD)

```py
# Request a full GPU, like CPUs we can request fractional
@ray.remote(num_gpus=1)
def do_serious_work():
# Restart entire worker after each call
@ray.remote(num_gpus=1, max_calls=1)
def do_serious_work():
```

重新启动的一个缺点是它会移除你在 GPU 或加速器中重用现有数据的能力。你可以通过使用长生命周期的 actors 来解决这个问题，但这会牺牲资源在这些 actors 中的锁定。

# Ray 的机器学习库

你也可以配置 Ray 的内置机器学习库来使用 GPU。为了让 Ray Train 启动 PyTorch 使用 GPU 资源进行训练，你需要在`Trainer`构造函数调用中设置`use_gpu=True`，就像你配置工作进程数量一样。Ray Tune 为资源请求提供了更大的灵活性，你可以在`tune.run`中指定资源，使用与`ray.remote`相同的字典。例如，要在每次试验中使用两个 CPU 和一个 GPU，你会调用`tune.run(trainable, num_samples=10, resources_per_trial={"cpu": 2, "gpu": 2})`。

# 带有 GPU 和加速器的自动缩放器

Ray 的自动缩放器具有理解不同类型节点并选择基于请求资源调度的能力。这在 GPU 方面尤为重要，因为 GPU 比其他资源更昂贵（且供应更少）。在我们的集群中，由于只有四个带有 GPU 的节点，我们配置自动缩放器如下(https://oreil.ly/juA4y)：

```py
imagePullSecrets: []
# In practice you _might_ want an official Ray image
# but this is for a bleeding-edge mixed arch cluster,
# which still is not fully supported by Ray's official
# wheels & containers.
image: holdenk/ray-ray:nightly
operatorImage: holdenk/ray-ray:nightly
podTypes:
  rayGPUWorkerType:
    memory: 10Gi
    maxWorkers: 4
    minWorkers: 1
# Normally you'd ask for a GPU but NV auto labeler is...funky on ARM
    CPU: 1
    rayResources:
      CPU: 1
      GPU: 1
      memory: 1000000000
    nodeSelector:
      node.kubernetes.io/gpu: gpu
  rayWorkerType:
    memory: 10Gi
    maxWorkers: 4
    minWorkers: 1
    CPU: 1
  rayHeadType:
    memory: 3Gi
    CPU: 1
```

这样，自动缩放器可以分配不带 GPU 资源的容器，从而使 Kubernetes 能够将这些 pod 放置在仅 CPU 节点上。

# CPU 回退作为一种设计模式

大多数可以通过 GPU 加速的高级库也具有 CPU 回退功能。Ray 没有内置的表达 CPU 回退或“如果可用则使用 GPU”的方法。在 Ray 中，如果你请求一个资源，调度程序找不到它，并且自动缩放器无法为其创建一个实例，该函数或 actor 将永远阻塞。通过一些创意，你可以在 Ray 中构建自己的 CPU 回退代码。

如果你希望在集群有 GPU 资源时使用它们，并在没有 GPU 时回退到 CPU，你需要做一些额外的工作。确定集群是否有可用的 GPU 资源的最简单方法是请求 Ray 运行一个带有 GPU 的远程任务，然后基于此设置资源，如示例 11-3 所示。

##### 示例 11-3\. [如果不存在 GPU，则回退到 CPU](https://oreil.ly/xjpkD)

```py
# Function that requests a GPU
@ray.remote(num_gpus=1)
def do_i_have_gpus():
    return True

# Give it at most 4 minutes to see if we can get a GPU
# We want to give the autoscaler some time to see if it can spin up
# a GPU node for us.
futures = [do_i_have_gpus.remote()]
ready_futures, rest_futures = ray.wait(futures, timeout=240)

resources = {"num_cpus": 1}
# If we have a ready future, we have a GPU node in our cluster
if ready_futures:
    resources["num_gpus"] =1

# "splat" the resources
@ray.remote(** resources)
def optional_gpu_task():
```

你使用的任何库也需要回退到基于 CPU 的代码。如果它们不会自动这样做（例如，根据 CPU 与 GPU 调用不同的两个函数，如`mul_two_cuda`和`mul_two_np`），你可以通过传递一个布尔值来指示集群是否有 GPU。

###### 警告

如果 GPU 资源没有得到正确释放，这仍可能导致在 GPU 集群上失败。理想情况下，你应该修复 GPU 释放问题，但在多租户集群上，这可能不是一个选项。你还可以在每个函数内部尝试/捕获获取 GPU 的异常。

# 其他（非 GPU）加速器

尽管本章的大部分内容集中在 GPU 加速器上，但相同的一般技术也适用于其他类型的硬件加速。例如，Numba 能够利用特殊的 CPU 特性，而 TensorFlow 则可以利用张量处理单元（TPU）。在某些情况下，资源可能不需要代码更改，而只是通过相同的 API 提供更快的性能，例如具有非易失性存储器快速执行（NVMe）驱动器的机器。在所有这些情况下，您可以配置自动扩展器来标记并使这些资源可用，方式与 GPU 类似。

# 结论

GPU 是在 Ray 上加速某些工作流程的绝佳工具。虽然 Ray 本身没有用于利用 GPU 加速代码的钩子，但它与各种您可以用于 GPU 计算的库集成良好。许多这些库并非为共享计算而创建，因此需要特别注意意外资源泄漏，尤其是由于 GPU 资源往往更昂贵。

¹ 其他顶级用例之一是加密货币挖矿，但你不需要像 Ray 这样的系统。使用 GPU 进行加密货币挖矿导致需求增加，许多显卡售价高于官方价格，而 NVIDIA 已经在[尝试用其最新的 GPU 抑制加密货币挖矿](https://oreil.ly/tG6qH)。
