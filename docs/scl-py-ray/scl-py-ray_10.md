# 第九章：使用 Ray 进行高级数据处理

尽管数据生态系统的快速发展，您可能最终需要使用多种工具作为数据管道的一部分。Ray Datasets 允许在数据和 ML 生态系统中的工具之间共享数据。这使您可以在不复制或移动数据的情况下切换工具。Ray Datasets 支持 Spark、Modin、Dask 和 Mars，还可以与 TensorFlow 等 ML 工具一起使用。您还可以使用 Arrow 与 Ray 结合使用，使更多工具可以在数据集上运行，如 R 或 MATLAB。Ray Datasets 作为 ML 管道各个步骤的通用格式，简化了传统管道。

一切都归结为这一点：您可以在多个工具中使用相同的数据集，而不必担心细节。在内部，许多这些工具都有自己的格式，但 Ray 和 Arrow 会透明地进行转换管理。

除了简化您使用不同工具的过程之外，Ray 还拥有一个不断增长的内置操作集合，专为数据集设计。这些内置操作正在积极开发中，不打算与基于 Ray 构建的数据工具一样全面。

###### 提示

如《光线对象》一文所述（“Ray Objects”），Ray Datasets 的默认行为可能与您的期望不同。您可以通过在 `ray.init` 中设置 `enable_object_reconstruction=True` 来启用对象恢复，使得 Ray Datasets 更加弹性化。

Ray Datasets 仍然是积极开发的一个领域，包括在次要版本之间的大型功能添加，到您阅读本章时可能会添加更多功能。尽管如此，分区和多工具互操作性的基本原则将保持不变。

# 创建和保存 Ray Datasets

正如您在 Example 2-9 中看到的，您可以通过调用 `ray.data.from_items` 从本地集合创建数据集。然而，本地集合自然地限制了您可以处理的数据范围，因此 Ray 支持许多其他选项。

Ray 使用 [Arrow](https://oreil.ly/GY0at) 将外部数据加载到数据集中，支持多种文件格式和文件系统。目前支持的格式包括 CSV、JSON、Parquet、NumPy、文本和原始二进制格式。加载数据的函数遵循 `read_[*format*]` 模式，并位于 `ray.data` 模块中，如 Example 9-1 所示。

##### Example 9-1\. [加载本地数据](https://oreil.ly/HP05n)

```py
    ds = ray.data.read_csv(
        "2021",
        partition_filter=None # Since the file doesn't end in .csv
    )                   
```

在加载时，您可以指定目标 `parallelism`，但 Ray 可能会受到加载文件数量的限制。为您的目标并行度选择一个好的值是复杂的，并取决于多个因素。您希望确保您的数据可以轻松放入内存，并充分利用集群中的所有机器，同时又不要选择一个任务启动开销超过收益的数字。一般来说，导致分割在数百兆到数十吉字节之间的并行性常被认为是一个甜蜜点。

###### 提示

如果您希望自定义 Arrow 加载数据的方式，可以通过 `arrow_open_stream_args` 参数向 Arrow 传递额外的参数，如 `compression` 或 `buffer_size`。

Arrow 在 S3、HDFS 和常规文件系统上具有内置的本机（快速）支持。Ray 会根据路径自动选择正确的内置文件系统驱动程序。

###### 警告

当从本地文件系统加载时，在分布式模式下运行时，您需要确保文件在所有工作器上都可用。

Arrow，以及其扩展 Ray，还使用 [`fsspec`](https://oreil.ly/Tz32F)，它支持更广泛的文件系统，包括 HTTPS（当安装 aiohttp 时）。与“内置”文件系统不同，您需要手动指定文件系统，如 示例 9-2 所示。

##### 示例 9-2\. [通过 HTTPS 加载数据](https://oreil.ly/HP05n)

```py
fs = fsspec.filesystem('https')
ds = ray.data.read_csv(
    "https://https://gender-pay-gap.service.gov.uk/viewing/download-data/2021",
    filesystem=fs,
    partition_filter=None # Since the file doesn't end in .csv
    )
```

###### 警告

目前，协议被错误地剥离了，所以您需要将其重复两次。例如，当从 HTTPS 网站加载数据时，您应该从 `https://https://[someurlhere].com` 加载。

Ray 有能力以其可以从中读取的所有格式进行写入。写入函数与读取函数类似，遵循 `write_[*format*]` 模式。读取路径和写入路径之间存在一些细微差异。写入路径始终使用输入数据集的并行性，而不是接收 `parallelism` 参数：

```py
word_count.write_csv("s3://ray-demo/wc")
```

如果 Ray 没有对您期望的格式或文件系统提供 I/O 支持，请检查 Ray 支持的其他工具中是否有适用的。然后，如下一节所述，您可以将数据集从/转换为所需的工具。

# 使用不同工具的 Ray 数据集

Ray 具有内置的工具，可在运行在 Ray 上的各种数据工具之间共享数据。这些工具中的大多数都有其自己的数据内部表示，但 Ray 会根据需要转换数据。在首次使用数据集与 Spark 或 Dask 之前，您需要运行一些设置代码，以便它们将其执行委托给 Ray，如示例 9-3 和 9-4 所示。

##### 示例 9-3\. [在 Ray 上设置 Dask](https://oreil.ly/HP05n)

```py
from ray.util.dask import enable_dask_on_ray, disable_dask_on_ray
enable_dask_on_ray() # Routes all Dask calls through the Ray scheduler
```

##### 示例 9-4\. [在 Spark 上设置 Dask](https://oreil.ly/HP05n)

```py
import raydp
spark = raydp.init_spark(
  app_name = "sleepy",
  num_executors = 2,
  executor_cores = 1,
  executor_memory = "2GB"
)
```

与用于读取和加载数据集的功能类似，将数据传输至 Ray 的功能在 `ray.data` 模块上定义，并遵循 `from_[*x*]` 模式，其中 `[*x*]` 是工具名称。与写入数据类似，我们使用数据集上定义的 `to_[*x*]` 函数将数据集转换为工具，其中 `[*x*]` 是工具名称。示例 9-5 展示了如何使用此模式将 Ray 数据集转换为 Dask DataFrame。

###### 注意

数据集不使用 Ray 的运行时环境来处理依赖关系，因此您必须在工作器映像中安装所需的工具；请参阅 附录 B。对于 Spark 而言，这更加复杂，因为它需要 Java 虚拟机 (JVM) 和其他非 Python 组件。

##### 示例 9-5\. [Dask 中的 Ray 数据集](https://oreil.ly/HP05n)

```py
dask_df = ds.to_dask()
```

您不仅限于 Ray 内置的工具。如果您有一个支持 Arrow 的新工具，并且正在使用 [支持 Arrow 的类型](https://oreil.ly/qNbFA)，`to_arrow_refs` 可以为您的数据集提供零拷贝的 Arrow 表示。然后，您可以使用 Ray Arrow 对象列表将其传递给您的工具，无论是用于模型训练还是其他任何目的。您将在 “使用内置 Ray 数据集操作” 中了解更多信息。

许多工具和语言可以与 Arrow 和 Ray 连接，包括：

+   [Apache Spark](https://oreil.ly/o2m4s)

+   [Dask](https://oreil.ly/tqbJY)

+   [Apache Parquet](https://oreil.ly/Mj0N8)

+   [Modin](https://oreil.ly/uSMt5)

+   [pandas](https://oreil.ly/oX5FO)

+   [TensorFlow](https://oreil.ly/7sweI)

+   [R](https://oreil.ly/41btG)

+   [JSON](https://oreil.ly/shYH5)

+   [MATLAB](https://oreil.ly/tqqR2)

###### 提示

Dask 和 Spark 都有非 DataFrame 集合——bags、arrays 和 resilient distributed datasets (RDDs)，这些无法使用这些 API 转换。

# 在 Ray 数据集上使用工具

本节假定您已经对将要在 Ray 上使用的数据整理工具——无论是 pandas 还是 Spark——有很好的理解。Pandas 对于扩展 Python 分析是理想的选择，而 Spark 则非常适合连接大数据工具的用户。如果您对 pandas API 不熟悉，建议查阅 [*Python 数据分析*](https://oreil.ly/IoSNu)（Wes McKinney 著，O’Reilly 出版）。新的 Spark 用户应查阅 [*学习 Spark*](https://oreil.ly/wmypt)（Jules Damji 等著，O’Reilly 出版）。如果您想深入了解，Holden 推荐 [*高性能 Spark*](https://oreil.ly/lyZ99)（Holden 和 Rachel Warren 著，O’Reilly 出版）。¹

## 使用 Dask 的类似 pandas 的 DataFrame

[Ray 上的 Dask](https://oreil.ly/ylNqR) 是为 ML 数据准备或扩展现有 pandas 代码的极佳选择。许多最初的 Dask 开发者也参与了 pandas 的开发，因此具有相对稳固的分布式 pandas 接口。

###### 注意

本节部分内容基于 [*Scaling Python with Dask*](https://oreil.ly/Fk0I6) 中的 DataFrame 章节。

在 Ray 上使用 Dask 可以从 Ray 的每个节点/容器共享内存存储数据中获益。在执行像广播连接这样的操作时尤为重要；在 Dask 中，相同的数据将需要存储在每个工作进程中。² 然而，在 Ray 中，只需要在每个节点或容器中存储一次。

###### 警告

与 Ray 不同，Dask 通常是惰性的，这意味着它在被迫之前不会评估数据。这可能会增加调试的难度，因为错误可能出现在距离其根本原因几行的地方。

Dask 的 DataFrame 的大多数分布式组件使用三个核心构建块 `map_partitions`、`reduction` 和 `rolling`。你通常不需要直接调用这些函数；而是会使用更高级别的 API，但理解它们以及它们的工作原理对于理解 Dask 的工作方式至关重要。`shuffle` 是重新组织数据的分布式 DataFrame 的关键构建块。与其他构建块不同，你可能更频繁地直接使用它，因为 Dask 无法抽象分区。

## 索引

在 pandas 中，对 DataFrame 进行索引是其强大功能之一，但在进入像 Dask 这样的分布式系统时，会有一些限制。由于 Dask 默认不跟踪每个分区的大小，因此不支持按行进行位置索引。你可以使用对列的位置索引，以及对列或行的标签索引。

索引经常用于过滤数据，仅保留你需要的组件。我们通过查看只包括所有疫苗接种状态的病例率的方式，在 San Francisco COVID-19 数据中执行了这项工作，如 示例 9-6 所示。

##### 示例 9-6\. [Dask DataFrame 索引](https://oreil.ly/IJaQ2)

```py
mini_sf_covid_df = sf_covid_df[ sf_covid_df['vaccination_status'] == 
  'All'][['specimen_collection_date', 'new_cases']]
```

如果你确实需要按行进行位置索引，可以通过计算每个分区的大小并使用它来选择所需的分区子集来实现自己的方法。这种方法非常低效，因此 Dask 避免直接实现，所以在执行之前你需要做出有意识的选择。

## Shuffles

如前一章节所述，shuffles 是昂贵的。造成 shuffle 昂贵的主要原因是网络速度相对于从内存读取数据而言较慢，以及序列化开销。随着被洗牌的数据量增加，这些成本会按比例增加，因此 Dask 有技术手段来减少被洗牌的数据量。这些技术依赖于某些数据属性或正在执行的操作。

###### 注意

虽然理解 shuffle 对性能很重要，但如果你的代码运行良好，可以跳过本节。

### 滚动窗口和 map_overlap

触发需要 shuffle 的一种情况是滚动窗口，在分区的边缘，你的函数需要一些来自其邻居的记录。Dask DataFrame 具有一个特殊的 `map_overlap` 函数，你可以在其中指定一个*向后查看*窗口（也称为*向前查看*窗口）和一个*向前查看*窗口（也称为*向后查看*窗口）的行来传输（可以是整数或时间增量）。利用这一点的最简单示例是滚动平均，如 示例 9-7 所示。

##### 示例 9-7\. [Dask DataFrame 滚动平均](https://oreil.ly/IJaQ2)

```py
def process_overlapped(df):
     df.rolling('5D').mean()
rolling_avg = partitioned_df.map_overlap(process_overlapped, pd.Timedelta('5D'), 0)
```

使用 `map_overlap` 允许 Dask 仅传输所需的数据。为了确保该实现能够正确运行，你的最小分区大小需要大于最大的窗口大小。

###### 警告

Dask 的滚动窗口不会跨越多个分区。如果你的 DataFrame 分区方式使得向后或向前查看的长度大于相邻分区的长度，结果将会失败或者不正确。Dask 对于时间增量的向后查看进行了验证，但对于向前查看或整数增量则没有进行此类检查。

### 聚合

聚合是另一种可以减少需要通过网络传输的数据量的特殊情况。聚合是结合记录的函数。如果你来自于 map/reduce 或 Spark 的背景，`reduceByKey` 是经典的聚合方法。聚合可以是*按键*进行的，也可以是整个 DataFrame 的全局聚合。

要按键进行聚合，首先需要使用表示键的列（或用于聚合的键函数）调用 `groupby`。例如，调用 `df.groupby("PostCode")` 将根据邮政编码对 DataFrame 进行分组，或者调用 `df.groupby(["PostCode", "SicCodes"])` 使用多列组合进行分组。在函数上，许多与 pandas 相同的聚合函数是可用的，但在 Dask 中聚合的性能与本地 pandas DataFrame 有很大不同。

###### 提示

如果按分区键进行聚合，Dask 可以在不需要洗牌的情况下计算聚合结果。

加速聚合的第一种方法是减少进行聚合的列，因为处理速度最快的数据是没有数据。最后，如果可能，同时执行多个聚合可以减少多次洗牌相同数据的次数。因此，如果需要计算平均值和最大值，应该像 示例 9-8 中显示的那样同时计算两者。

##### 示例 9-8\. [同时计算 Dask DataFrame 的最大值和平均值](https://oreil.ly/IJaQ2)

```py
dask.compute(
    raw_grouped[["new_cases"]].max(),
    raw_grouped[["new_cases"]].mean())
```

对于像 Dask 这样的分布式系统，如果可以部分评估然后合并聚合，你可以在预洗牌前合并一些记录。并非所有部分聚合都是相等的。部分聚合重要的是当合并具有相同键的值时减少的数据量，与原始多个值使用的存储空间相比。

最有效的聚合方式可以在不考虑记录数量的情况下占用亚线性的空间量。其中一些可以占用恒定空间，如 sum、count、first、minimum、maximum、mean 和 standard deviation。更复杂的任务，如分位数和不同计数，也有亚线性近似选项。这些近似选项非常有效，因为精确答案可能需要线性增长的存储空间。

一些聚合函数的增长不是亚线性的，但“倾向于”或“可能”增长不会太快。计算不同值的数量属于此类别，但如果所有值都是唯一的，则没有节省空间。

要利用高效的聚合功能，你需要使用 Dask 提供的内置聚合，或者使用 Dask 的聚合类自己编写。在可能的情况下，使用内置聚合。内置聚合不仅需要更少的工作量，而且通常更快。并非所有的 pandas 聚合在 Dask 中都直接支持，因此有时你的唯一选择是编写自己的聚合。

如果选择编写自己的聚合，你需要定义三个函数：`chunk` 处理每个组/分区块，`agg` 组合分区之间 `chunk` 的结果，以及（可选的）`finalize` 用于获取 `agg` 的结果并生成最终值。

理解如何使用部分聚合的最快方法是查看一个使用所有三个函数的示例。在 Example 9-9 中使用加权平均值可以帮助你思考每个函数所需的内容。第一个函数需要计算加权值和权重。`agg` 函数通过对元组的各部分求和来组合这些值。最后，`finalize` 函数将总和除以权重。

##### Example 9-9\. [Dask 自定义聚合](https://oreil.ly/IJaQ2)

```py
# Write a custom weighted mean, we get either a DataFrameGroupBy with
# multiple columns or SeriesGroupBy for each chunk
def process_chunk(chunk):
    def weighted_func(df):
        return (df["EmployerSize"] * df["DiffMeanHourlyPercent"]).sum()
    return (chunk.apply(weighted_func), chunk.sum()["EmployerSize"])

def agg(total, weights):
    return (total.sum(), weights.sum())

def finalize(total, weights):
    return total / weights

weighted_mean = dd.Aggregation(
    name='weighted_mean',
    chunk=process_chunk,
    agg=agg,
    finalize=finalize)

aggregated = df_diff_with_emp_size.groupby("PostCode")
    ["EmployerSize", "DiffMeanHourlyPercent"].agg(weighted_mean)
```

在某些情况下，比如纯求和，你不需要在 `agg` 的输出上进行任何后处理，因此可以跳过 `finalize` 函数。

并非所有的聚合都必须按键进行；你还可以跨所有行计算聚合。然而，Dask 的自定义聚合接口仅在按键操作中暴露。

Dask 的内置完整 DataFrame 聚合使用一个称为 `apply_contact_apply` 的低级接口，用于部分聚合。与学习两种不同的部分聚合 API 相比，我们更倾向于通过提供一个常量分组函数来进行静态 `groupby`。这样，我们只需了解一个聚合接口。你可以使用这个方法在整个 DataFrame 中找到聚合 COVID-19 数字的方式，如 Example 9-10 所示。

##### Example 9-10\. [聚合整个 DataFrame](https://oreil.ly/IJaQ2)

```py
raw_grouped = sf_covid_df.groupby(lambda x: 0)
```

如果存在内置的聚合方法，它们很可能比我们能写的任何东西都要好。有时部分聚合是部分实现的，就像 Dask 的 HyperLogLog 一样：它仅适用于完整的 DataFrames。你可以通过复制 `chunk` 函数，使用 `agg` 的 `combine` 参数，以及使用 `finalize` 的 `aggregate` 参数来转换简单的聚合。这在将 Dask 的 HyperLogLog 实现移植到 Example 9-11 中有所展示。

##### Example 9-11\. [将 Dask 的 HyperLogLog 封装在 `dd.Aggregation` 中](https://oreil.ly/IJaQ2)

```py
# Wrap Dask's hyperloglog in dd.Aggregation

from dask.dataframe import hyperloglog

approx_unique = dd.Aggregation(
    name='aprox_unique',
    chunk=hyperloglog.compute_hll_array,
    agg=hyperloglog.reduce_state,
    finalize=hyperloglog.estimate_count)

aggregated = df_diff_with_emp_size.groupby("PostCode")
    ["EmployerSize", "DiffMeanHourlyPercent"].agg(weighted_mean)
```

慢/低效的聚合操作（或者可能导致内存不足异常的操作）会使用与正在聚合的记录成比例的存储空间。这些慢操作的例子包括制作列表和简单地计算精确分位数。³ 对于这些慢聚合操作，使用 Dask 的聚合类并不能比`apply` API 带来好处，后者可能更简单。例如，如果你只想通过邮政编码获取雇主 ID 列表，而不想编写三个函数，你可以使用像`df.groupby("PostCode")["EmployerId"].apply(lambda g: list(g))`这样的一行代码。Dask 将`apply`函数实现为完全的洗牌操作，这将在下一节中讨论。

###### 警告

当你使用`apply`函数时，Dask 无法应用部分聚合。

### 完全洗牌和分区化

在分布式系统中，排序通常是昂贵的，因为它通常需要进行*完全洗牌*。有时，完全洗牌是使用 Dask 时不可避免的部分。有趣的是，虽然完全洗牌本身是慢的，但你可以用它来加速未来在相同分组键上进行的操作。正如在聚合部分提到的，通过在分区不对齐时使用`apply`方法，就会触发完全洗牌的一种方式。

### 分区

在重新分区数据时，你最常使用完全洗牌。在处理聚合、滚动窗口或查找/索引时，拥有正确的分区是很重要的。如“滚动窗口和 map_overlap”章节中讨论的，Dask 不能做超过一个分区的前视或后视，因此需要正确的分区才能获得正确的结果。对于大多数其他操作来说，分区不正确会减慢作业速度。

Dask 有三种主要方法来控制 DataFrame 的分区：`set_index`、`repartition`和`shuffle`。当分区正在更改为新的键/索引时，使用`set_index`。`repartition`保持相同的键/索引，但更改分片。`repartition`和`set_index`使用类似的参数，但`repartition`不接受索引键名。`shuffle`有些不同，因为它不生成“已知”的分区方案，像`groupby`这样的操作无法利用它。

获取 DataFrame 的正确分区的第一步是决定是否需要索引。索引对于几乎任何按键类型的操作都很有用，包括数据过滤、分组和当然索引。一个这样的按键操作可能是`groupby`；正在分组的列可能是一个很好的键的候选。如果在某列上使用滚动窗口，则该列必须是键，这使得选择键相对容易。一旦确定了索引，可以使用索引列名调用`set_index`（例如，`set_index("PostCode")`）。在大多数情况下，这将导致重新分区，因此现在是调整分区大小的好时机。

###### 提示

如果您不确定当前用于分区的键，可以检查`index`属性来查看分区键。

一旦选择了键，下一个问题是如何确定分区的大小。在“分区”中的建议通常适用于这里：努力使每台计算机保持忙碌，但要记住一般的最佳区间为 100 MB 到 1 GB。如果给定目标分区数，Dask 通常会计算出相当均匀的分割。⁴ 幸运的是，`set_index`也会接受`npartitions`。要通过邮政编码重新分区数据，并设定为 10 个分区，可以添加`set_index("PostCode", npartitions=10)`；否则，Dask 将默认使用输入分区的数量。

如果您计划使用滚动窗口，可能需要确保每个分区中涵盖了正确大小的（按键范围）记录。为了在`set_index`的一部分中执行此操作，您需要计算自己的分区，以确保每个分区中包含正确范围的记录。分区被指定为一个列表，从第一个分区的最小值到最后一个分区的最大值。每个值之间是用于分割构成 Dask DataFrame 的 pandas DataFrame 的“切点”。要创建一个包含`[0, 100) [100, 200), (300, 500]`的 DataFrame，您可以编写`df.set_index("Num​Employees", divisions=[0, 100, 200, 300, 500])`。类似地，为了支持从大流行开始到本文撰写时长达七天的滚动窗口，日期范围如示例 9-12 所示，需要写成 Example 9-12。

##### 示例 9-12\. [使用`set_index`的 Dask DataFrame 滚动窗口](https://oreil.ly/IJaQ2)

```py
divisions = pd.date_range(
    start="2021-01-01", end=datetime.today(), freq='7D').tolist()
partitioned_df_as_part_of_set_index = mini_sf_covid_df.set_index(
    'specimen_collection_date', divisions=divisions)
```

###### 警告

Dask，包括滚动时间窗口，在处理时假定您的分区索引是*单调递增*的——严格递增，没有重复的值（例如，1、4、7 是单调递增的，但 1、4、4、7 不是）。

到目前为止，您必须指定分区的数量或特定的分区，但您可能想知道 Dask 是否可以自行确定这一点。幸运的是，Dask 的 `repartition` 函数可以从目标大小选择分区。然而，这样做是一个不小的成本，因为 Dask 必须评估 DataFrame 以及重新分区本身。示例 9-13 展示了如何让 Dask 根据所需的分区大小（以字节为单位）计算分区。

##### 示例 9-13\. [Dask DataFrame 自动分区](https://oreil.ly/IJaQ2)

```py
reparted = indexed.repartition(partition_size="20kb")
```

###### 警告

截至撰写时，Dask 的 `set_index` 有一个类似的 `partition_size` 参数，但尚不起作用。

在编写 DataFrames 时，每个分区都有自己的文件，但有时这可能导致文件过大或过小。有些工具只能接受单个文件作为输入，因此您需要将所有内容重新分区为单个分区。其他情况下，数据存储系统针对特定文件大小进行了优化，例如 Hadoop 分布式文件系统（HDFS）的默认块大小为 128 MB。好消息是，您可以使用 `repartition` 或 `set_index` 来获得所需的输出结构。

## 尴尬并行操作

Dask 的 `map_partitions` 函数将函数应用于每个底层 pandas DataFrame 的分区，结果也是一个 pandas DataFrame。使用 `map_partitions` 实现的函数由于不需要任何工作节点之间的数据传输，因此是尴尬并行的。在 [尴尬并行问题](https://oreil.ly/NFYHB) 中，分布式计算和通信的开销很低。

`map_partitions` 实现了 `map` 和许多逐行操作。如果您想使用一个在逐行操作中找不到的函数，您可以像 示例 9-14 中所示自行实现它。

##### 示例 9-14\. [Dask DataFrame `fillna`](https://oreil.ly/IJaQ2)

```py
def fillna(df):
    return df.fillna(value={"PostCode": "UNKNOWN"}).fillna(value=0)

new_df = df.map_partitions(fillna)
# Since there could be an NA in the index clear the partition/division information
new_df.clear_divisions()
```

您不仅限于像本例中调用 pandas 内置函数。只要您的函数接受并返回 DataFrame，您几乎可以在 `map_partitions` 中实现任何想做的事情。

本章节中无法覆盖完整的 pandas API，但如果一个函数可以在逐行处理时不需要了解前后的行，则可能已经在 Dask DataFrames 中使用 `map_partitions` 实现。如果没有，您也可以使用 示例 9-14 中的模式自行实现。

当在 DataFrame 上使用 `map_partitions` 时，您可以更改每行的任何内容，包括其分区的键。如果您更改了分区键中的值，您必须使用 `clear_divisions` 清除生成的 DataFrame 上的分区信息，或者使用 `set_index` 指定正确的索引，关于这一点您将在下一节中了解更多。

###### 警告

不正确的分区信息可能导致不正确的结果，而不仅仅是异常，因为 Dask 可能会错过相关数据。

## 处理多个 DataFrames

pandas 和 Dask 有四个常见的函数用于合并 DataFrames。在根上是`concat`函数，它允许在任何轴上连接 DataFrames。在 Dask 中，连接 DataFrames 通常较慢，因为它涉及工作节点之间的通信。另外三个函数是`join`，`merge`和`append`，它们都在`concat`之上实现了常见情况的特殊情况，并且具有略有不同的性能考虑。在处理多个 DataFrames 时，良好的分区和划分选择对性能有很大影响。

Dask 的`join`和`merge`函数除了标准的 pandas 参数外，还有一个额外的可选参数。`npartitions`指定了目标输出分区的数量，但仅用于基于哈希的连接（您将在“多 DataFrame 内部”中了解到）。`join`和`merge`会根据需要自动重新分区输入的 DataFrames。这很好，因为您可能不知道分区情况，但由于重新分区可能很慢，明确使用较低级别的`concat`函数可以帮助及早发现性能问题。Dask 的 join 在进行左连接或外连接时一次只能接受两个以上的 DataFrames。

###### 提示

Dask 具有专门的逻辑来加速多 DataFrame 的连接，因此在大多数情况下，与`a.join(b).join(c).join(d).join(e)`相比，使用`a.join([b, c, d, e])`将更加有利。但是，如果您执行与小数据集的左连接，则第一种语法可能更高效。

当您通过行（类似于 SQL UNION）合并（通过`concat`）DataFrames 时，性能取决于要合并的 DataFrames 的分区是否有序。我们称一系列 DataFrames 的分区为*有序*，如果所有分区都是已知的，并且前一个 DataFrame 的最高分区低于下一个 DataFrame 的最低分区。如果任何输入具有未知分区，则 Dask 将生成一个没有已知分区的输出。具有所有已知分区时，Dask 将行基础的连接视为仅元数据更改，并且不会执行任何洗牌操作。这要求分区之间不存在重叠。此外，额外的`interleave_partitions`参数将行基础组合的连接类型更改为无输入分区限制，并将导致已知分区器。

Dask 的基于列的`concat`（类似于 SQL JOIN）在组合的 DataFrames 的分区/分割方面也有限制。Dask 的 concat 版本仅支持内部或完全外连接，不支持左连接或右连接。基于列的连接要求所有输入都有已知的分区器，并且结果是具有已知分区的 DataFrame。具有已知分区器对于后续连接非常有用。

###### 警告

在对具有未知分区的 DataFrame 按行操作时，不要使用 Dask 的 `concat`，因为它可能会返回不正确的结果。Dask 假设索引是对齐的，如果没有索引，则不会存在。

### 多 DataFrame 内部

Dask 使用四种技术——哈希、广播、分区和 `stack_partitions`——来合并 DataFrame，每种技术的性能差异很大。Dask 根据索引、分区和请求的连接类型（例如，outer/left/inner）选择合适的技术。三种基于列的连接技术是哈希连接、广播连接和分区连接。在执行基于行的组合（例如 `append`）时，Dask 使用一种特殊的技术称为 `stack_partitions`，速度非常快。重要的是，您了解每种技术的性能以及引起 Dask 选择哪种方法的条件。

*哈希* 连接是 Dask 在没有其他适合的连接技术时使用的默认技术。哈希连接会为所有输入 DataFrame 的数据洗牌，以在目标键上进行分区。哈希连接使用键的哈希值，导致生成的 DataFrame 没有任何特定顺序。因此，哈希连接的结果没有已知的分区。

*广播* 连接非常适合将大 DataFrame 与小 DataFrame 进行连接。在广播连接中，Dask 将较小的 DataFrame 分发给所有工作进程。这意味着较小的 DataFrame 必须能够放入内存中。为了告诉 Dask 一个 DataFrame 适合广播，确保它全部存储在一个分区中，例如调用 `repartition(npartitions=1)`。

*分区* 连接发生在沿着索引组合 DataFrame 时，其中所有 DataFrame 的分区都是已知的。由于输入分区已知，Dask 能够在 DataFrame 之间对齐分区，涉及的数据传输更少，因为每个输出分区具有比完整输入集更小的集合。

由于分区和广播连接速度更快，帮助 Dask 完成一些工作是值得的。例如，连接几个已知且对齐分区的 DataFrame，以及一个未对齐的 DataFrame，将导致昂贵的哈希连接。相反，尝试在剩余的 DataFrame 上设置索引并分区，或者先连接较便宜的 DataFrame，然后再执行昂贵的连接。

使用 `stack_partitions` 不同于所有其他选项，因为它不涉及任何数据的移动。相反，生成的 DataFrame 分区列表是输入 DataFrames 的上游分区的联合。Dask 在大多数基于行的组合中使用 `stack​_partiti⁠ons`，除非所有输入 DataFrame 的分区都已知且未正确排序，并且您请求 Dask `interleave_partitions`。当输入分区已知且正确排序时，`stack_partitions` 函数能够仅在其输出中提供已知分区。如果所有分区都已知但未正确排序，并且您设置了 `interleave_partitions`，Dask 将使用分区连接。虽然这种方法比较便宜，但并非免费，可能导致分区数量过多，需要重新分区。

### 缺失功能

并非所有的多 DataFrame 操作都已实现；`compare` 就是这样一个操作，这导致了关于 Dask DataFrames 限制的下一节。

## 什么不起作用

Dask 的 DataFrame 实现了大部分，但并非全部 pandas DataFrame API。由于开发时间的关系，某些 pandas API 在 Dask 中并未实现。其他部分则是为了避免暴露出意外缓慢的 API 而未使用。

有时 API 只是缺少一些小部分，因为 pandas 和 Dask 都在积极开发中。例如 `split` 函数就是一个例子。在本地 pandas 中，你可以调用 `split(expand=true)`，而不是进行 `split().explode`。其中一些可以是参与并[为 Dask 项目做贡献](https://oreil.ly/OHPqQ)的好地方，如果您感兴趣的话。

有些库的并行效果不如其他库好。在这些情况下，一种常见的方法是尝试对数据进行足够的筛选或聚合，以便可以在本地表示数据，然后将本地库应用于数据。例如，在绘图时，通常会预先聚合计数或进行随机抽样，然后绘制结果。

虽然大部分 pandas DataFrame API 都可以直接使用，但在切换到 Dask DataFrame 之前，重要的是确保您有良好的测试覆盖率，以捕捉它不适用的情况。

## 什么更慢

通常情况下，使用 Dask DataFrames 可以提高性能，但并非总是如此。一般来说，较小的数据集在本地 pandas 中的表现更好。正如讨论的那样，任何涉及洗牌的操作在分布式系统中通常比在本地系统中慢。迭代算法还可以产生大量操作的图形，这些操作在 Dask 中评估速度比传统的贪婪评估要慢。

一些问题通常不适合数据并行计算。例如，使用具有更多并行写入者的单个锁的数据存储写入时会增加锁争用，并可能比单线程写入更慢。在这些情况下，你可以有时重新分区数据或将单个分区写入以避免锁争用。

## 处理递归算法

Dask 的惰性评估，由其谱系图提供支持，通常是有益的，使其能够自动合并步骤。然而，当图变得太大时，Dask 可能会在管理上遇到困难，这通常表现为驱动程序或笔记本变慢，有时会出现内存不足的异常。幸运的是，你可以通过将 DataFrame 写出并重新读入来解决这个问题。一般来说，Parquet 是执行此操作的最佳格式，因为它占用空间小且自描述，因此不需要进行模式推断。

## 其他函数有何不同

由于性能原因，Dask DataFrames 的各个部分与本地 DataFrames 行为略有不同。

`reset_index`

索引将在每个分区上重新从零开始。

`kurtosis`

不会过滤掉非数字（NaN）值并使用 SciPy 默认值。

`concat`

不同于强制执行类别类型，每个类别类型都扩展为其与所有连接的类别的并集。

`sort_values`

Dask 仅支持单列排序。

连接

当同时连接两个以上的 DataFrames 时，连接类型必须是 outer 或 left。

###### 提示

如果你对深入了解 Dask 感兴趣，有几本专注于 Dask 的书籍正在积极开发中。本章中的大部分材料基于 [*Scaling Python with Dask*](https://oreil.ly/Fk0I6)。

## 类似 pandas 的 Modin DataFrames

[Modin](https://oreil.ly/KR1wT)，就像 Dask DataFrames 一样，旨在大部分替代 pandas DataFrames。Modin DataFrames 的整体性能与 Dask DataFrames 相似，但有几个注意事项。Modin 对内部的控制较少，这可能限制某些应用的性能。由于 Modin 和 Dask DataFrames 相似度足够高，我们在这里不会详细介绍，只是说如果 Dask 不能满足你的需求，Modin 是另一个选择。

###### 注意

*Modin* 是一个旨在通过自动分布计算加速 pandas 的新库，可以利用系统上所有可用的 CPU 核心。Modin 声称可以几乎线性加速系统上 pandas DataFrames 的计算，无论大小如何。

由于 Modin 在 Ray 上与 Dask DataFrames 如此相似，我们决定跳过重复 Dask 在 Ray 上的示例，因为它们在本质上并无大变化。

###### 警告

当你并排查看 Dask 和 Modin 的文档时，你可能会觉得 Dask 在其开发周期中较早。在我们看来，事实并非如此；相反，Dask 文档采用更为保守的方法来标记功能是否就绪。

## 使用 Spark 进行大数据处理

如果您正在使用现有的大数据基础设施（如 Apache Hive、Iceberg 或 HBase），Spark 是一个极好的选择。Spark 具有诸如过滤器推送等优化功能，可以显著提高性能。Spark 具有更传统的大数据 DataFrame 接口。

Spark 的强项在于其所属的数据生态系统。作为一个基于 Java 的工具，带有 Python API，Spark 与传统的大数据生态系统高度集成。Spark 支持最广泛的格式和文件系统，使其成为许多流水线初始阶段的优秀选择。

虽然 Spark 继续添加更多类似于 pandas 的功能，但其 DataFrame 最初是基于 SQL 设计的。您有几种选项可以了解 Spark，包括一些 O'Reilly 的书籍：[*Learning Spark*](https://oreil.ly/LearningSpark2) by Jules Damji，[*High Performance Spark*](https://oreil.ly/highperfSpark) by Holden and Rachel Warren，以及[*Spark: The Definitive Guide*](https://oreil.ly/sparkTDG) by Bill Chambers and Matei Zaharia。

###### 警告

与 Ray 不同，Spark 通常是惰性的，这意味着它不会在被强制之前评估数据。这可能会使调试变得具有挑战性，因为错误可能会出现在其根本原因几行之外。

## 使用本地工具

有些工具不太适合分布式操作。幸运的是，只要您的数据集足够小，您可以将其转换为各种本地进程格式。如果整个数据集可以放入内存中，`to_pandas`和`to_arrow`是将数据集转换为本地对象的最简单方法。对于较大的对象，每个分区可能适合内存，但整个数据集可能不适合，`iter_batches`将为您提供一个生成器/迭代器，以逐个分区消耗数据。`iter_batches`函数接受`batch_format`参数，在`pandas`和`pyarrow`之间进行切换。如果可能，`pyarrow`通常比`pandas`更高效。

# 使用内置 Ray Dataset 操作

除了允许您在各种工具之间传输数据外，Ray 还具有一些内置操作。Ray Datasets 并不试图匹配任何特定的现有 API，而是暴露基本的构建块，当现有库不满足您的需求时可以使用这些构建块。

Ray Datasets 支持基本的数据操作。Ray Datasets 并不旨在提供类似于 pandas 的 API；相反，它专注于提供基本的原语来构建。Dataset API 的功能受到启发，同时具有面向分区的函数。Ray 还最近添加了`groupBy`和聚合功能。

大多数数据集操作的核心构建块是 `map_batches`。默认情况下，`map_batches` 在构成数据集的块或批次上执行您提供的函数，并使用结果生成新数据集。`map_batches` 函数用于实现 `filter`、`flat_map` 和 `map`。通过查看将单词计数示例重写为直接使用 `map_batches` 的示例，您可以看到 `map_batches` 的灵活性，同时删除仅出现一次的单词，如 Example 9-15 所示。

##### Example 9-15\. 使用 `map_batches` 进行 Ray 单词计数的示例，详见[Ray word count with `map_batches`](https://oreil.ly/HP05n)

```py
def tokenize_batch(batch):
    nested_tokens = map(lambda s: s.split(" "), batch)
    # Flatten the result
    nr = []
    for r in nested_tokens:
        nr.extend(r)
    return nr

def pair_batch(batch):
    return list(map(lambda w: (w, 1), batch))

def filter_for_interesting(batch):
    return list(filter(lambda wc: wc[1] > 1, batch))

words = pages.map_batches(tokenize_batch).map_batches(pair_batch)
# The one part we can't rewrite with map_batches since it involves a shuffle
grouped_words = words.groupby(lambda wc: wc[0]) 
interesting_words = groupd_words.map_batches(filter_for_interesting)
```

`map_batches` 函数接受参数以定制其行为。对于有状态的操作，您可以将计算策略从默认的`tasks`更改为`actors`。前面的示例使用了默认格式，即 Ray 的内部格式，但您也可以将数据转换为`pandas`或`pyarrow`。您可以在 Example 9-16 中看到 Ray 将数据转换为 pandas 的示例。

##### Example 9-16\. 使用 Ray 的 `map_batches` 与 pandas 更新列的示例，详见[Ray `map_batches` with pandas to update a column](https://oreil.ly/HP05n)

```py
# Kind of hacky string munging to get a median-ish to weight our values.
def update_empsize_to_median(df):
    def to_median(value):
        if " to " in value:
            f , t = value.replace(",", "").split(" to ")
            return (int(f) + int(t)) / 2.0
        elif "Less than" in value:
            return 100
        else:
            return 10000
    df["EmployerSize"] = df["EmployerSize"].apply(to_median)
    return df

ds_with_median = ds.map_batches(update_empsize_to_median, batch_format="pandas")
```

###### 提示

您返回的结果必须是列表、`pandas` 或 `pyarrow`，并且不需要与接收的相同类型匹配。

Ray 数据集没有内置的方法来指定要安装的附加库。您可以使用 `map_batches` 和任务来完成此操作，如 Example 9-17 所示，它安装额外的库以解析 HTML。

##### Example 9-17\. 使用 Ray 的 `map_batches` 与额外库的示例，详见[Using Ray `map_batches` with extra libraries](https://oreil.ly/HP05n)

```py
def extract_text_for_batch(sites):
    text_futures = map(lambda s: extract_text.remote(s), sites)
    result = ray.get(list(text_futures))
    # ray.get returns None on an empty input, but map_batches requires lists
    if result is None:
        return []
    return result

def tokenize_batch(texts):
    token_futures = map(lambda s: tokenize.remote(s), texts)
    result = ray.get(list(token_futures))
    if result is None:
        return []
    # Flatten the result
    nr = []
    for r in result:
        nr.extend(r)
    return nr

# Exercise for the reader: generalize the preceding patterns - 
# note the flatten magic difference

urls = ray.data.from_items(["http://www.holdenkarau.com", "http://www.google.com"])

pages = urls.map(fetch)

page_text = pages.map_batches(extract_text_for_batch)
words = page_text.map_batches(tokenize_batch)
word_count = words.groupby(lambda x: x).count()
word_count.show()
```

对于需要洗牌的操作，Ray 拥有 `GroupedDataset`，其行为略有不同。与其余的 Datasets API 不同，Ray 中的 `groupby` 是惰性评估的。`groupby` 函数接受列名或函数，其中具有相同值的记录将被聚合在一起。一旦您有了 `GroupedDataset`，您就可以将多个聚合传递给 `aggregate` 函数。Ray 的 `AggregateFn` 类在概念上类似于 Dask 的 `Aggregation` 类，只是它是按行操作的。由于它是按行操作的，所以当发现新的键值时，您需要提供一个 `init` 函数。对于每个新元素，您提供 `accumulate` 而不是每个新块提供 `chunk`。您仍然提供一种组合聚合器的方法，称为 `merge` 而不是 `agg`，两者都有可选的 `finalize`。为了理解差异，我们将 Dask 加权平均示例改写为 Ray，如 Example 9-18 所示。

##### Example 9-18\. Ray 加权平均聚合的示例，详见[Ray weighted average aggregation](https://oreil.ly/HP05n)

```py
def init_func(key):
    # First elem is weighted total, second is weights
    return [0, 0]

def accumulate_func(accumulated, row):
    return [
        accumulated[0] + 
        (float(row["EmployerSize"]) * float(row["DiffMeanHourlyPercent"])),
        accumulated[1] + row["DiffMeanHourlyPercent"]]

def combine_aggs(agg1, agg2):
    return (agg1[0] + agg2[0], agg1[1] + agg2[1])

def finalize(agg):
    if agg[1] != 0:
        return agg[0] / agg[1]
    else:
        return 0

weighted_mean = ray.data.aggregate.AggregateFn(
    name='weighted_mean',
    init=init_func,
    merge=combine_aggs,
    accumulate_row=accumulate_func, # Used to be accumulate
    # There is a higher performance option called accumulate_block for vectorized op
    finalize=finalize)
aggregated = ds_with_median.groupby("PostCode").aggregate(weighted_mean)
```

###### 注意

使用 `None` 实现完整数据集聚合，因为所有记录都具有相同的键。

Ray 的并行控制不像 Dask 的索引或 Spark 的分区那样灵活。您可以控制目标分区的数量，但无法控制数据的分布方式。

###### 注意

Ray 目前没有利用已知分区概念以最小化洗牌操作。

# 实现 Ray 数据集

Ray 数据集是使用您在前几章中使用的工具构建的。Ray 将每个数据集分割成许多较小的组件。这些较小的组件在 Ray 代码内部被称为 *blocks* 和 *partitions*。每个分区包含一个 Arrow 数据集，表示整个 Ray 数据集的一个切片。由于 Arrow 不支持 Ray 的所有类型，如果有不支持的类型，每个分区还包含一个不支持类型的列表。

每个数据集内部的数据存储在标准的 Ray 对象存储中。由于 Ray 不能分割单个对象，每个分区都存储为一个单独的对象。这也意味着您可以将底层的 Ray 对象用作 Ray 远程函数和 actors 的参数。数据集包含对这些对象的引用以及模式信息。

###### 提示

由于数据集包含模式信息，加载数据集会阻塞在第一个分区上，以便确定模式信息。其余分区会急切加载，但像 Ray 的其他操作一样不会阻塞。

与 Ray 的其余部分保持一致，数据集是不可变的。当您想对数据集执行操作时，您会应用一个转换，比如 `filter`、`join` 或 `map`，Ray 返回一个包含结果的新数据集。

Ray 数据集可以使用任务（也称为远程函数）或者 actors 来进行转换处理。像 Modin 这样构建在 Ray 数据集之上的库依赖于使用 actors 进行处理，以便能够实现涉及状态的某些 ML 任务。

# 结论

Ray 在处理工具之间透明地处理数据移动方面表现出色，与传统技术相比，在工具之间的通信障碍要高得多。两个独立的框架，Modin 和 Dask，都在 Ray 数据集之上提供了类似于 pandas 的体验，这使得扩展现有的数据科学工作流程变得简单。在 Ray 数据集上的 Spark（称为 *RayDP*）为那些在具有现有大数据工具的组织中工作的人提供了一条简便的集成路径。

在本章中，您学会了如何使用 Ray 有效地处理数据，以支持您的机器学习和其他需求。在下一章中，您将学习如何使用 Ray 来支持机器学习。

¹ 这就像一家福特经销商建议购买福特车一样，所以接受这些建议时要持保留态度。

² 在 Dask 中，通过使用多线程可以避免原生代码中出现这个问题，但是具体细节超出了本书的范围。

³ 对精确分位数的备用算法依赖于更多的洗牌操作以减少空间开销。

⁴ 键偏斜可以使得已知的分区器无法执行此操作。
