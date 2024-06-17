# 附录 D. 使用 Streamz 和 Dask 进行流式处理

本书专注于使用 Dask 构建批处理应用程序，其中数据是从用户收集或提供的，并用于计算。 另一组重要的用例是需要在数据变得可用时处理数据的情况。¹ 处理数据变得可用时称为流式处理。

由于人们对其数据驱动产品有更高期望，流数据管道和分析变得越来越受欢迎。 想象一下，如果银行交易需要数周才能完成，那将显得极其缓慢。 或者，如果您在社交媒体上阻止某人，您期望该阻止立即生效。 虽然 Dask 擅长交互式分析，但我们认为它（目前）并不擅长对用户查询做出即时响应。²

流式作业与批处理作业在许多重要方面有所不同。 它们往往需要更快的处理时间，并且这些作业本身通常没有定义的终点（除非公司或服务被关闭）。 小批处理作业可能无法胜任的一种情况包括动态广告（几十到几百毫秒）。 许多其他数据问题可能会模糊界限，例如推荐系统，其中您希望根据用户互动更新它们，但是几分钟的延迟可能（主要）是可以接受的。

如 第八章 中所讨论的，Dask 的流式组件似乎比其他组件使用频率低。 在 Dask 中，流式处理在一定程度上是事后添加的，³ 在某些场合和时间，您可能会注意到这一点。 当加载和写入数据时，这一点最为明显 —— 一切都必须通过主客户端程序移动，然后分散或收集。

###### 警告

目前，Streamz 无法处理比客户端计算机内存中可以容纳的更多数据。

在本附录中，您将学习 Dask 流处理的基本设计，其限制以及与其他一些流式系统的比较。

###### 注意

截至本文撰写时，Streamz 在许多地方尚未实现 `ipython_display`，这可能导致在 Jupyter 中出现类似错误的消息。 您可以忽略这些消息（它会回退到 `repr`）。

# 在 Dask 上开始使用 Streamz

安装 Streamz 很简单。 它可以从 PyPI 获得，并且您可以使用 `pip` 安装它，尽管像所有库一样，您必须在所有工作节点上可用。 安装完 Streamz 后，您只需创建一个 Dask 客户端（即使是在本地模式下），然后导入它，如 示例 D-1 所示。

##### 示例 D-1\. 开始使用 Streamz

```py
import dask
import dask.dataframe as dd
from streamz import Stream
from dask.distributed import Client

client = Client()
```

###### 注意

当存在多个客户端时，Streamz 使用创建的最近的 Dask 客户端。

# 流数据源和接收器

到目前为止，在本书中，我们从本地集合或分布式文件系统加载了数据。虽然这些数据源确实可以作为流数据的来源（有一些限制），但在流数据世界中存在一些额外的数据源。流数据源不同于有定义结束的数据源，因此行为更像生成器而不是列表。流接收器在概念上类似于生成器的消费者。

###### 注意

Streamz 的接收端（或写入目的地）支持有限，这意味着在许多情况下，您需要使用自己的函数以流方式将数据写回。

一些流数据源具有重放或回溯已发布消息的能力（可配置的时间段），这对基于重新计算的容错方法尤为有用。两个流行的分布式数据源（和接收器）是 Apache Kafka 和 Apache Pulsar，两者都具备回溯查看先前消息的能力。一个没有此能力的示例流系统是 RabbitMQ。

[Streamz API 文档](https://oreil.ly/LOpPJ) 涵盖了支持的数据源；为简便起见，我们这里聚焦于 Apache Kafka 和本地可迭代数据源。Streamz 在主进程中进行所有加载，然后您必须分散结果。加载流数据应该看起来很熟悉，加载本地集合的示例见 示例 D-2，加载 Kafka 的示例见 示例 D-3。

##### 示例 D-2\. 加载本地迭代器

```py
local_stream = Stream.from_iterable(
    ["Fight",
     "Flight",
     "Freeze",
     "Fawn"])
dask_stream = local_stream.scatter()
```

##### 示例 D-3\. 从 Kafka 加载

```py
batched_kafka_stream = Stream.from_kafka_batched(
    topic="quickstart-events",
    dask=True, # Streamz will call scatter internally for us
    max_batch_size=2, # We want this to run quickly, so small batches
    consumer_params={
        'bootstrap.servers': 'localhost:9092',
        'auto.offset.reset': 'earliest', # Start from the start
        # Consumer group id
        # Kafka will only deliver messages once per consumer group
        'group.id': 'my_special_streaming_app12'},
    # Note some sources take a string and some take a float :/
    poll_interval=0.01) 
```

在这两个示例中，Streamz 将从最近的消息开始读取。如果您希望 Streamz 回到存储消息的起始位置，则需添加 ```py `` ```。

###### 警告

Streamz 在单头进程上进行读取是可能遇到瓶颈的地方，尤其在扩展时。

与本书的其余部分一样，我们假设您正在使用现有的数据源。如果情况不是这样，请查阅 Apache Kafka 或 Apache Pulsar 文档（以及 Kafka 适配器）以及 Confluent 的云服务。

# 词频统计

没有流部分会完整无缺地涉及到词频统计，但重要的是要注意我们在 示例 D-4 中的流式词频统计——除了数据加载的限制外——无法以分布式方式执行聚合。

##### 示例 D-4\. 流式词频统计

```py
local_wc_stream = (batched_kafka_stream
                   # .map gives us a per batch view, starmap per elem
                   .map(lambda batch: map(lambda b: b.decode("utf-8"), batch))
                   .map(lambda batch: map(lambda e: e.split(" "), batch))
                   .map(list)
                   .gather()
                   .flatten().flatten() # We need to flatten twice.
                   .frequencies()
                   ) # Ideally, we'd call flatten frequencies before the gather, 
                     # but they don't work on DaskStream
local_wc_stream.sink(lambda x: print(f"WC {x}"))
# Start processing the stream now that we've defined our sinks
batched_kafka_stream.start()
```

在前述示例中，您可以看到 Streamz 的一些当前限制，以及一些熟悉的概念（如 `map`）。如果您有兴趣了解更多，请参阅 [Streamz API 文档](https://oreil.ly/VpkEz)；然而，请注意，根据我们的经验，某些组件在非本地流上可能会随机失效。

# 在 Dask 流式处理中的 GPU 管道

如果您正在使用 GPU，[cuStreamz 项目](https://oreil.ly/QCk7O) 简化了 cuDF 与 Streamz 的集成。cuStreamz 使用了许多自定义组件来提高性能，例如将数据从 Kafka 加载到 GPU 中，而不是先在 Dask DataFrame 中着陆，然后再转换。cuStreamz 还实现了一个灵活性比默认 Streamz 项目更高的自定义版本的检查点技术。该项目背后的开发人员大多受雇于希望向您销售更多 GPU 的人，[声称可以实现高达 11 倍的加速](https://oreil.ly/6wUpB)。

# 限制、挑战和解决方法

大多数流处理系统都具有某种形式的状态检查点，允许在主控程序失败时从上一个检查点重新启动流应用程序。Streamz 的检查点技术仅限于不丢失任何未处理的记录，但累积状态可能会丢失。如果您的状态随着时间的推移而累积，则需要您自己构建状态检查点/恢复机制。这一点尤为重要，因为在足够长的时间窗口内，遇到单个故障点的概率接近 100%，而流应用程序通常打算永久运行。

这种无限期运行时间导致了许多其他挑战。小内存泄漏会随着时间的推移而累积，但您可以通过定期重新启动工作进程来减轻它们。

流式处理程序通常在处理聚合时会遇到晚到达的数据问题。这意味着，虽然您可以根据需要定义窗口，但您可能会遇到本该在窗口内的记录，但由于时间原因未及时到达处理过程。Streamz 没有针对晚到达数据的内置解决方案。您的选择是在处理过程中手动跟踪状态（并将其持久化到某处），忽略晚到达的数据，或者使用另一个支持晚到达数据的流处理系统（包括 kSQL、Spark 或 Flink）。

在某些流应用程序中，确保消息仅被处理一次很重要（例如，银行账户）。由于失败时需要重新计算，Dask 通常不太适合这种情况。这同样适用于在 Dask 上的 Streamz，其中唯一的选项是 *至少一次* 执行。您可以通过使用外部系统（如数据库）来跟踪已处理的消息来解决此问题。

# 结论

在我们看来，在 Dask 内支持流数据的 Streamz 取得了有趣的开始。它目前的限制使其最适合于流入数据量较小的情况。尽管如此，在许多情况下，流数据量远小于面向批处理数据的量，能够在一个系统中同时处理二者可以避免重复的代码或逻辑。如果 Streamz 不能满足您的需求，还有许多其他的 Python 流处理系统可供选择。您可能希望了解的一些 Python 流处理系统包括 Ray 流处理、Faust 或 PySpark。根据我们的经验，Apache Beam 的 Python API 比 Streamz 有更大的发展空间。

¹ 尽管通常会有一些（希望是小的）延迟。

² 尽管这两者都是“互动的”，但访问您的网站并下订单的人的期望与数据科学家试图制定新广告活动的期望有很大不同。

³ Spark 流处理也是如此，但 Dask 的流处理甚至比 Spark 的流处理集成性更低。
