# 附录 C. 使用 Ray 进行调试

根据您的调试技术，迁移到分布式系统可能需要一套新的技术。幸运的是，像 Pdb 和 PyCharm 这样的工具允许您连接远程调试器，而 Ray 的本地模式则允许您在许多其他情况下使用现有的调试工具。有些错误发生在 Python 之外，使得它们更难调试，如容器内存不足（OOM）错误、分段错误和其他本地错误。

###### 注意

此附录的一些部分与[*使用 Dask 扩展 Python*](https://oreil.ly/Fk0I6)共享，因为它们是调试所有类型分布式系统的一般良好建议。

# 使用 Ray 的一般调试技巧

您可能有自己的标准 Python 代码调试技术，本附录不旨在替代它们。以下是一些在使用 Ray 时更有意义的一般技术：

+   将失败的函数拆分为较小的函数。由于`ray.remote`在函数块上调度，较小的函数使问题更容易隔离。

+   注意任何意外的作用域捕获。

+   使用示例数据尝试在本地重现它（本地调试通常更容易）。

+   使用 Mypy 进行类型检查。虽然我们并未在所有示例中包含类型，但在生产代码中，使用自由的类型可以捕获棘手的错误。

+   当问题无论并行化如何出现时，请在单线程模式下调试您的代码，这样更容易理解发生了什么。

现在，有了这些额外的一般技巧，是时候了解更多有关工具和技术，帮助您进行 Ray 调试了。

# 序列化错误

序列化在 Ray 中起着重要作用，但也可能是头痛的来源，因为小的更改可能导致意外的变量捕获和序列化失败。幸运的是，Ray 在 `ray.util` 中有一个实用函数 `inspect_serializability`，您可以使用它来调试序列化错误。如果您有意定义一个捕获非可序列化数据的函数，比如 示例 C-1，您可以运行 `inspect_serializability` 看看它如何报告失败（如 示例 C-2）。

##### 示例 C-1\. [错误的序列化示例](https://oreil.ly/eygeJ)

```py
pool = Pool(5)

def special_business(x):
    def inc(y):
        return y + x
    return pool.map(inc, range(0, x))
ray.util.inspect_serializability(special_business)
```

##### 示例 C-2\. 错误的序列化结果

```py
=========================================================================
Checking Serializability of <function special_business at 0x7f78802820d0>
=========================================================================
!!! FAIL serialization: pool objects cannot be passed between processes or pickled
Detected 1 global variables. Checking serializability...
    Serializing 'pool' <multiprocessing.pool.Pool state=RUN pool_size=5>...
    !!! FAIL serialization: pool objects cannot be passed between processes ...
...
```

在此示例中，Ray 检查元素的可序列化性，并指出非序列化值 `pool` 来自全局范围。

# 使用 Ray 本地调试

在本地模式下使用 Ray，可以让您使用习惯的工具，而无需处理设置远程调试的复杂性。我们不会涵盖各种本地 Python 调试工具，因此这一部分存在的意义仅在于提醒您首先尝试在本地模式下重现问题，然后再开始使用本附录其余部分涵盖的高级调试技术。

# 远程调试

远程调试可以是一个很好的工具，但需要更多对集群的访问权限，这在某些情况下可能并不总是可用。Ray 的特殊集成工具 `ray debug` 支持跨整个集群的追踪。不幸的是，其他远程 Python 调试器一次只能附加到一个机器上，因此你不能简单地将调试器指向整个集群。

###### 警告

远程调试可能会导致性能变化和安全隐患。在启用集群上的远程调试之前，通知所有用户非常重要。

如果你控制自己的环境，设置远程调试相对比较简单，但在企业部署中，你可能会遇到启用这一功能的阻力。在这种情况下，使用本地集群或请求一个开发集群进行调试是你最好的选择。

###### 提示

对于交互式调试器，你可能需要与系统管理员合作，以公开集群中额外的端口。

## Ray 的集成调试器（通过 Pdb）

Ray 支持通过 Pdb 进行调试的集成，允许你跨集群跟踪代码。你仍然需要修改启动命令 (`ray start`) 来包括 (`ray start --ray-debugger-external`) 来加载调试器。启用 Ray 的外部调试器后，Pdb 将在额外端口上监听（无需任何认证），以供调试器连接。

一旦你配置并启动了集群，你可以在主节点上启动 Ray 调试器。¹ 要启动调试器，只需运行 `ray debug`，然后你可以使用所有你喜欢的 [Pdb 调试命令](https://oreil.ly/mR50g)。

## 其他工具

对于非集成工具，由于每次对远程函数的调用可能被安排在不同的工作节点上，你可能会发现将你的无状态函数暂时转换为执行器会更容易调试。这将会对性能产生真实的影响，因此在生产环境中可能不适用，但确实意味着重复调用将路由到同一台机器，从而使调试任务更加简单。

### PyCharm

PyCharm 是一个集成调试器的流行 Python IDE。虽然它不像 Pdb 那样集成，但你可以通过几个简单的更改使其工作。第一步是将 `pydevd-pycharm` 包添加到你的容器/要求中。然后，在你想要调试的执行器中，你可以像 示例 C-3 中展示的那样启用 PyCharm 调试。

##### 示例 C-3\. [启用 PyCharm 远程调试](https://oreil.ly/eygeJ)

```py
@ray.remote
class Bloop():

    def __init__(self, dev_host):
        import pydevd_pycharm
        # Requires ability to connect to dev from prod.
        try:
            pydevd_pycharm.settrace(
                dev_host, port=7779, stdoutToServer=True, stderrToServer=True)
        except ConnectionRefusedError:
            print("Skipping debug")
            pass

    def dothing(x):
        return x + 1
```

你的执行器将会从执行者回到你的 PyCharm IDE 创建连接。

### Python 分析器

Python 分析器可以帮助追踪内存泄漏、热代码路径以及其他重要但不是错误状态。

从安全角度来看，性能分析器比远程实时调试问题少，因为它们不需要从您的计算机直接连接到集群。 相反，分析器运行并生成报告，您可以离线查看。 性能分析仍然会带来性能开销，因此在决定是否启用时要小心。

要在执行器上启用 Python 内存分析，您可以更改启动命令以添加前缀`mprof run -E --include-children, -o memory​_pro⁠file.dat --python`。 然后，您可以收集`memory_profile`并在您的计算机上用`matplotlib`绘制它们，以查看是否有什么异常。

类似地，您可以通过在启动命令中用`echo "from ray.scripts.scripts import main; main()" > launch.py; python -m cProfile -o stats launch.py`替换`ray start`来在`ray execute`中启用函数分析。 这比使用`mprof`略微复杂，因为默认的 Ray 启动脚本与`cProfile`不兼容，因此您需要创建一个不同的入口点—但在概念上是等效的。

###### 警告

用于基于注释的分析的`line_profiler`包与 Ray 不兼容，因此您必须使用整体程序分析。

# Ray 和容器退出代码

*退出代码*是程序退出时设置的数字代码，除了 0 以外的任何值通常表示失败。 这些代码（按照惯例）通常有意义，但不是 100%一致。 以下是一些常见的退出代码：

0

成功（但通常误报，特别是在 shell 脚本中）

1

通用错误

127

命令未找到（在 shell 脚本中）

130

用户终止（Ctrl-C 或 kill）

137

Out-of-memory error *或* kill -9（强制终止，不可忽略）

139

段错误（通常是本地代码中的空指针解引用）

您可以使用`echo $?`打印出上次运行命令的退出代码。 在运行严格模式脚本（如某些 Ray 启动脚本）时，您可以打印出退出代码，同时传播错误`[raycommand] || (error=$?; echo $error; exit $error)`。²

# Ray 日志

Ray 的日志与许多其他分布式应用程序的日志行为不同。 由于 Ray 倾向于在容器启动之后在容器上启动工作进程，³ 与容器关联的 stdout 和 stderr 通常不包含您需要的调试信息。 相反，您可以通过查找 Ray 在头节点上创建符号链接到的最新会话目录*/tmp/ray/session_latest*来访问工作容器日志。

# 容器错误

调试容器错误可能特别具有挑战性，因为到目前为止探索的许多标准调试技术都有挑战。 这些错误可以从常见的错误（如 OOM 错误）到更神秘的错误。 很难区分容器错误或退出的原因，因为容器退出有时会删除日志。

在 Kubernetes 上，有时可以通过在日志请求中添加`-p`来获取已经退出的容器的日志（例如，`kubectl logs -p`）。你还可以配置`terminationMessagePath`来指向包含有关终止退出信息的文件。如果你的 Ray worker 退出，定制 Ray 容器启动脚本以增加更多日志记录可能是有意义的。常见的附加日志类型包括从*syslog*或*dmesg*中获取的最后几行（查找 OOMs），将其记录到稍后可以用于调试的文件位置。

最常见的容器错误之一，本地内存泄漏，可能很难调试。像[Valgrind](https://oreil.ly/8E9iG)这样的工具有时可以追踪本地内存泄漏。使用类似 Valgrind 的工具的详细信息超出了本书的范围，请查看[Python Valgrind 文档](https://oreil.ly/jzRwT)。你可能想尝试的另一个“技巧”是有效地二分你的代码；因为本地内存泄漏最常发生在库调用中，你可以尝试注释它们并运行测试，看看哪个库调用是泄漏的来源。

# 本地错误

本地错误和核心转储与容器错误具有相同的调试挑战。由于这些类型的错误通常导致容器退出，因此访问调试信息变得更加困难。这个问题的“快速”解决方案是在 Ray 启动脚本中（失败时）添加`sleep`，以便你可以连接到容器（例如，`[raylaunchcommand] || sleep 100000`）并使用本地调试工具。

然而，在许多生产环境中，访问容器的内部可能比表面看起来更容易。出于安全原因，你可能无法获得远程访问（例如，在 Kubernetes 上使用`kubectl exec`）。如果是这种情况，你可以（有时）向容器规范添加关闭脚本，将核心文件复制到容器关闭后仍然存在的位置（例如，s3、HDFS 或 NFS）。

# 结论

在 Ray 中，启动调试工具可能需要更多工作，如果可能的话，Ray 的本地模式是远程调试的一个很好的选择。你可以利用 Ray 的 actors 来使远程函数调度更可预测，这样更容易知道在哪里附加你的调试工具。并非所有错误都是平等的，一些错误，比如本地代码中的分段错误，尤其难以调试。祝你找到 Bug(s)！我们相信你。

¹ Ray 有`ray attach`命令来创建到主节点的 SSH 连接；但并非所有主节点都会有 SSH 服务器。在 Ray on Kubernetes 上，你可以通过运行`kubectl exec -it -n [rayns] [podname] – /bin/bash`来到达主节点。每个集群管理器在此处略有不同，因此你可能需要查阅你的集群管理器的文档。

² 这个更改的确切配置位置取决于所使用的集群管理器。对于在 Kube 上使用自动缩放器的 Ray，你可以修改 `workerStartRayCommands`。对于在 AWS 上的 Ray，修改 `worker_start_ray_commands`，等等。

³ 这可以通过 `ssh` 或 `kubectl exec` 完成。
