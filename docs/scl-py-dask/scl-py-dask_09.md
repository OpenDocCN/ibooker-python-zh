# 第九章：迁移现有分析工程

许多用户已经部署了当前正在使用的分析工作，他们希望将其迁移到 Dask。本章将讨论用户进行切换时的考虑、挑战和经验。本章主要探讨将现有大数据工程作业从其他分布式框架（如 Spark）迁移到 Dask 的主要迁移路径。

# 为什么选择 Dask？

以下是考虑从现有在 pandas 中实现的作业或 PySpark 等分布式库迁移到 Dask 的一些理由：

Python 和 PyData 堆栈

许多数据科学家和开发人员更喜欢使用 Python 本地堆栈，他们不需要在不同语言或风格之间切换。

与 Dask API 更丰富的 ML 集成

Futures、delayed 和 ML 集成要求开发人员减少粘合代码的编写，由于 Dask 提供更灵活的任务图管理，性能有所提升。

精细化任务管理

Dask 的任务图在运行时实时生成和维护，并且用户可以同步访问任务字典。

调试开销

一些开发团队更喜欢 Python 中的调试体验，而不是混合 Python 和 Java/Scala 堆栈跟踪。

开发开销

在 Dask 中进行开发步骤可以轻松在开发者的笔记本电脑上完成，而不需要连接到强大的云机器以进行实验。

管理用户体验

Dask 的可视化工具往往更具视觉吸引力和直观性，具有用于任务图的本地 graphviz 渲染。

这些并非所有的优势，但如果其中任何一个对你有说服力，考虑将工作负载转移到 Dask 可能是值得投资时间考虑的。总是会有权衡，因此接下来的部分将讨论一些限制，并提供一个路线图，以便让你了解迁移到 Dask 所涉及的工作规模。

# Dask 的限制

Dask 是比较新的技术，使用 Python 数据堆栈执行大规模抽取、转换和加载操作也是相对较新的。Dask 存在一些限制，主要是因为 PyData 堆栈传统上并不用于执行大规模数据工作负载。在撰写本文时，系统存在一些限制。然而，开发人员正在解决这些问题，许多这些不足将会被弥补。你应该考虑一些精细化的注意事项，如下所述：

Parquet 的规模限制

如果 Parquet 数据超过 10 TB，fastparquet 和 PyArrow 层面会出现问题，这会拖慢 Dask 的速度，并且元数据管理的开销可能会很大。

在 Parquet 文件达到 10 TB 以上的 ETL 工作负载中，包括追加和更新等变异，会遇到一致性问题。

弱数据湖集成

PyData 堆栈在传统上并没有在大数据领域大量使用，并且在数据湖管理方面的集成，如 Apache Iceberg，尚未完善。

高级查询优化

Spark 的用户可能熟悉 Catalyst 优化器，该优化器推动优化执行器上的物理工作。目前 Dask 还缺少这种优化层。Spark 在早期也没有写 Catalyst 引擎，目前正在进行相关工作，以为 Dask 构建此功能。

像 Dask 这样快速发展的项目的任何限制列表，在你阅读时可能已经过时，因此如果这些限制是您迁移的阻碍因素，请确保检查 Dask 的状态跟踪器。

# 迁移路线图

虽然没有工程工作是线性进行的，但随时掌握路线图始终是个好主意。我们已经列出了迁移步骤的示例，作为团队在计划迁移时可能需要考虑的非穷尽列表项：

+   我们将希望在什么类型的机器和容器化框架上部署 Dask，它们各自的优缺点是什么？

+   我们是否有测试来确保我们的迁移正确性和我们期望的目标？

+   Dask 能够摄取什么类型的数据，在什么规模下，以及这与其他平台有何不同？

+   Dask 的计算框架是什么，以及我们如何以 Dask 和 Pythonic 的方式思考来完成任务？

+   我们将如何在运行时监控和排除代码问题？

我们将从查看集群类型开始，这与部署框架相关，因为这通常是需要与其他团队或组织合作的问题之一。

## 集群类型

如果您考虑迁移您的分析工程工作，您可能拥有一个由您的组织提供的系统。Dask 在许多常用的部署和开发环境中受到支持，其中一些允许更灵活的扩展、依赖管理和支持异构工作类型。我们在学术环境、通用云和直接在虚拟机/容器上使用了 Dask；我们详细说明了各自的优缺点以及一些广泛使用和支持的环境，详见 附录 A。

示例 9-1 展示了 YARN 部署的示例。更多示例和深入讨论可见于 第十二章。

##### 示例 9-1\. 使用 Dask-Yarn 和 skein 在 YARN 上部署 Dask

```py
from dask_yarn import YarnCluster
from dask.distributed import Client

# Create a cluster where each worker has two cores and 8 GiB of memory
cluster = YarnCluster(
    environment='your_environment.tar.gz',
    worker_vcores=2,
    worker_memory="4GiB")

# Scale out to num_workers such workers
cluster.scale(num_workers)

# Connect to the cluster
client = Client(cluster)
```

如果您的组织有多个受支持的集群，选择一个可以自助依赖管理的集群，如 Kubernetes，将是有益的。

对于使用 PBS、Slurm、MOAB、SGE、LSF 和 HTCondor 等作业队列系统进行高性能计算部署，应使用 Dask-jobqueue，如 示例 9-2 所示。

##### 示例 9-2\. 使用 jobqueue 在 Slurm 上部署 Dask

```py
from dask_jobqueue import SLURMCluster
from dask.distributed import Client

cluster = SLURMCluster(
    queue='regular',
    account="slurm_caccount",
    cores=24,
    memory="500 GB"
)
cluster.scale(jobs=SLURM_JOB_COUNT)  # Ask for N jobs from Slurm

client = Client(cluster)

# Auto-scale between 10 and 100 jobs
cluster.adapt(minimum_jobs=10, maximum_jobs=100)
cluster.adapt(maximum_memory="10 TB")  # Or use core/memory limits
```

你可能已经由你的组织管理员设置了共享文件系统。企业用户可能已经习惯了在 HDFS 或像 S3 这样的 Blob 存储上运行的健全配置的分布式数据源，而 Dask 能够无缝地与之配合（参见示例 9-3）。Dask 也与网络文件系统良好集成。

##### 示例 9-3\. 使用 MinIO 读取和写入 Blob 存储

```py
import s3fs
import pyarrow as pa
import pyarrow.parquet as pq

minio_storage_options = {
    "key": MINIO_KEY,
    "secret": MINIO_SECRET,
    "client_kwargs": {
        "endpoint_url": "http://ENDPOINT_URL",
        "region_name": 'us-east-1'
    },
    "config_kwargs": {"s3": {"signature_version": 's3v4'}},
}

df.to_parquet(f's3://s3_destination/{filename}',
              compression="gzip",
              storage_options=minio_storage_options,
              engine="fastparquet")

df = dd.read_parquet(
    f's3://s3_source/',
    storage_options=minio_storage_options,
    engine="pyarrow"
)
```

我们发现一个令人惊讶地有用的用例是直接连接到网络存储，如 NFS 或 FTP。在处理大型且难以处理的学术数据集时（例如直接由另一个组织托管的神经影像数据集），我们可以直接连接到源文件系统。使用 Dask 这种方式时，你应该测试并考虑网络超时的允许。此外，请注意，截至本文撰写时，Dask 尚未具备与 Iceberg 等数据湖的连接器。

## 开发：考虑因素

将现有逻辑转换为 Dask 是一个相当直观的过程。以下部分介绍了如果你来自 R、pandas 和 Spark 等库，并且 Dask 可能与它们有何不同的一些考虑因素。其中一些差异来自于从不同的低级实现（如 Java）移动，其他差异来自于从单机代码移动到扩展实现，例如从 pandas 移动而来。

### DataFrame 性能

如果你已经在不同平台上运行作业，很可能已经在运行时使用列存储格式，例如 Parquet。从 Parquet 到 Python 的数据类型映射固有地不精确。建议在运行时读取任何数据时检查数据类型，DataFrame 亦如此。如果类型推断失败，列会默认为对象。一旦检查并确定类型推断不精确，指定数据类型可以显著加快作业速度。此外，检查字符串、浮点数、日期时间和数组总是个好主意。如果出现类型错误，牢记上游数据源及其数据类型是一个好的开始。例如，如果 Parquet 是从协议缓冲生成的，根据使用的编码和解码引擎，该堆栈中引入了空检查、浮点数、双精度和混合精度类型的差异。

当从云存储读取大文件到 DataFrame 时，在 DataFrame 读取阶段预先选择列可能非常有用。来自其他平台（如 Spark）的用户可能熟悉谓词下推，即使你没有完全指定所需的列，平台也会优化并仅读取计算所需的列。Dask 目前尚未提供这种优化。

在 DataFrame 转换早期设置智能索引，在复杂查询之前，可以加快速度。请注意，Dask 尚不支持多索引。对于来自其他平台的多索引 DataFrame 的常见解决方法是映射为单一连接列。例如，从非 Dask 列数据集（如 pandas 的 `pd.MultiIndex`，其索引有两列 `col1` 和 `col2`）来时的一个简单解决方法是在 Dask DataFrame 中引入一个新列 `col1_col2`。

在转换阶段，调用 `.compute()` 方法将大型分布式 Dask DataFrame 合并为一个单一分区，应该可以放入 RAM 中。如果不行，可能会遇到问题。另一方面，如果您已经将大小为 100 GB 的输入数据过滤到了 10 GB（假设您的 RAM 是 15 GB），那么在过滤操作后减少并行性可能是个好主意，方法是调用 `.compute()`。您可以通过调用 `df.memory_usage(deep=True).sum()` 来检查 DataFrame 的内存使用情况，以确定是否需要进行此操作。如果在过滤操作后有复杂且昂贵的洗牌操作，比如与新的更大数据集的 `.join()` 操作，这样做尤其有用。

###### 提示

与 pandas DataFrame 用户熟悉的内存中值可变不同，Dask DataFrame 不支持这种方式的值可变。由于无法在内存中修改特定值，唯一的改变值的方式将是对整个 DataFrame 列进行映射操作。如果经常需要进行内存中值的更改，最好使用外部数据库。

### 将 SQL 迁移到 Dask

Dask 并不原生支持 SQL 引擎，尽管它原生支持从 SQL 数据库读取数据的选项。有许多不同的库可以用来与现有的 SQL 数据库交互，并且将 Dask DataFrame 视为 SQL 表格并直接运行 SQL 查询（参见 示例 9-4）。一些库甚至允许您直接构建和提供 ML 模型，使用类似于 Google BigQuery ML 的 SQL ML 语法。在示例 11-14 和 11-15 中，我们将展示使用 Dask 的原生 `read_sql()` 函数以及使用 Dask-SQL 运行 SQL ML 的用法。

##### 示例 9-4\. 从 Postgres 数据库读取

```py
df = dd.read_sql_table('accounts', 'sqlite:///path/to/your.db',
                       npartitions=10, index_col='id')
```

FugueSQL 为 PyData 栈（包括 Dask）提供了 SQL 兼容性。该项目处于起步阶段，但似乎很有前途。FugueSQL 的主要优势在于代码可以在 pandas、Dask 和 Spark 之间进行移植，提供了更多的互操作性。FugueSQL 可以使用 `DaskExecutionEngine` 运行其 SQL 查询，或者在已经使用的 Dask DataFrame 上运行 FugueSQL 查询。或者，你也可以在笔记本上快速在 Dask DataFrame 上运行 SQL 查询。示例 9-5 展示了在笔记本中使用 FugueSQL 的示例。FugueSQL 的缺点是需要 ANTLR 库，而 ANTLR 又依赖于 Java 运行时。

##### 示例 9-5\. 使用 FugueSQL 在 Dask DataFrame 上运行 SQL

```py
from fugue_notebook import setup
setup (is_lab=True)
ur = ('https://d37ci6vzurychx.cloudfront.net/trip-data/'
      'yellow_tripdata_2018-01.parquet')
df = dd.read_parquet(url)

%%fsql dask
tempdf = SELECT VendorID, AVG (total_amount) AS average_fare FROM df
GROUP BY VendorID

SELECT *
FROM tempdf
ORDER BY average fare DESC
LIMIT 5
PRINT
```

|  | VendorID | average_fare |
| --- | --- | --- |
| **0** | 1 | 15.127384 |
| **1** | 2 | 15.775723 |

```py
schema: VendorID:long, average_fare:double
```

另一种方法是使用 Dask-SQL 库。该软件包使用 Apache Calcite 提供 SQL 解析前端，并用于查询 Dask 数据帧。使用该库，你可以将大多数基于 SQL 的操作传递给 Dask-SQL 上下文，并进行处理。引擎处理标准 SQL 输入，如 `SELECT`、`CREATE TABLE`，同时还支持使用 `CREATE MODEL` 语法进行 ML 模型创建。

## 部署监控

像许多其他分布式库一样，Dask 提供日志记录功能，你可以配置 Dask 日志将其发送到存储系统。部署环境会影响方法的选择，以及是否涉及 Jupyter。

Dask 客户端暴露了 `get_worker_logs()` 和 `get_scheduler_logs()` 方法，如果需要可以在运行时访问。此外，类似于其他分布式系统的日志记录，你可以按主题记录事件，使其易于按事件类型访问。

示例 9-6 是在客户端添加自定义日志事件的玩具示例。

##### 示例 9-6\. 按主题进行基本日志记录

```py
from dask.distributed import Client

client = Client()
client.log_event(topic="custom_events", msg="hello world")
client.get_events("custom_events")
```

示例 9-7 在前一个示例的基础上构建，但是将执行上下文切换到分布式集群设置中，以处理可能更复杂的自定义结构化事件。Dask 客户端监听并累积这些事件，我们可以进行检查。我们首先从一个 Dask 数据帧开始，然后执行一些计算密集型任务。本示例使用 `softmax` 函数，这是许多 ML 应用中常见的计算。常见的 ML 困境是是否使用更复杂的激活或损失函数来提高准确性，牺牲性能（从而运行更少的训练周期，但获得更稳定的梯度），反之亦然。为了弄清楚这一点，我们插入一个代码来记录定制的结构化事件，以计算特定函数的计算开销。

##### 示例 9-7\. 工作节点上的结构化日志

```py
from dask.distributed import Client, LocalCluster

client = Client(cluster)  # Connect to distributed cluster and override default

d = {'x': [3.0, 1.0, 0.2], 'y': [2.0, 0.5, 0.1], 'z': [1.0, 0.2, 0.4]}
scores_df = dd.from_pandas(pd.DataFrame(data=d), npartitions=1)

def compute_softmax(partition, axis=0):
    """ computes the softmax of the logits
 :param logits: the vector to compute the softmax over
 :param axis: the axis we are summing over
 :return: the softmax of the vector
 """
    if partition.empty:
        return
    import timeit
    x = partition[['x', 'y', 'z']].values.tolist()
    start = timeit.default_timer()
    axis = 0
    e = np.exp(x - np.max(x))
    ret = e / np.sum(e, axis=axis)
    stop = timeit.default_timer()
    partition.log_event("softmax", {"start": start, "x": x, "stop": stop})
    dask.distributed.get_worker().log_event(
        "softmax", {"start": start, "input": x, "stop": stop})
    return ret

scores_df.apply(compute_softmax, axis=1, meta=object).compute()
client.get_events("softmax")
```

# 结论

在本章中，您已经审查了迁移现有分析工程工作的重要问题和考虑因素。您还了解了 Dask 与 Spark、R 和 pandas 之间的一些特征差异。一些特性尚未由 Dask 实现，一些特性则由 Dask 更为稳健地实现，还有一些是在将计算从单机迁移到分布式集群时固有的翻译差异。由于大规模数据工程倾向于在许多库中使用类似的术语和名称，往往容易忽视导致更大性能或正确性问题的细微差异。记住它们将有助于您在 Dask 中迈出第一步的旅程。
