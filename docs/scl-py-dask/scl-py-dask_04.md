# 第四章 Dask DataFrame

虽然 Pandas DataFrame 非常流行，但随着数据规模的增长，它们很快会遇到内存限制，因为它们将整个数据存储在内存中。Pandas DataFrame 具有强大的 API，用于各种数据操作，并且经常是许多分析和机器学习项目的起点。虽然 Pandas 本身没有内置机器学习功能，但数据科学家们经常在新项目的探索阶段的数据和特征准备中使用它。因此，将 Pandas DataFrame 扩展到能够处理大型数据集对许多数据科学家至关重要。大多数数据科学家已经熟悉 Pandas 库，而 Dask 的 DataFrame 实现了大部分 Pandas API，并且增加了扩展能力。

Dask 是最早实现可用子集的 Pandas API 之一，但其他项目如 Spark 已经添加了它们自己的方法。本章假定您已经对 Pandas DataFrame API 有很好的理解；如果没有，您应该查看[*Python for Data Analysis*](https://learning.oreilly.com/library/view/python-for-data/9781098104023)。

由于鸭子类型，你经常可以在只做少量更改的情况下使用 Dask DataFrame 替代 Pandas DataFrame。然而，这种方法可能会有性能缺陷，并且一些功能是不存在的。这些缺点来自于 Dask 的分布式并行性质，它为某些类型的操作增加了通信成本。在本章中，您将学习如何最小化这些性能缺陷，并解决任何缺失功能。

Dask DataFrame 要求您的数据和计算与 Pandas DataFrame 非常匹配。Dask 有用于非结构化数据的 bags，用于数组结构化数据的 arrays，用于任意函数的 Dask 延迟接口，以及用于有状态操作的 actors。如果即使在小规模下您都不考虑使用 Pandas 解决您的问题，那么 Dask DataFrame 可能不是正确的解决方案。

# Dask DataFrame 的构建方式

Dask DataFrame 是基于 Pandas DataFrame 构建的。每个分区都存储为一个 Pandas DataFrame。¹ 使用 Pandas DataFrame 作为分区简化了许多 API 的实现。特别是对于基于行的操作，Dask 会将函数调用传递给每个 Pandas DataFrame。

大多数分布式组件都是基于三个核心构建模块`map_partitions`、`reduction`和`rolling`来构建的。通常情况下，你不需要直接调用这些函数；而是使用更高级的 API。但是理解这些函数以及它们的工作原理对于理解 Dask 的工作方式非常重要。`shuffle`是重新组织数据的分布式 DataFrame 的关键构建块。与其他构建模块不同的是，你可能更频繁地直接使用它，因为 Dask 无法隐藏分区。

# 加载和写入

数据分析只有在能够访问到数据时才有价值，我们的见解只有在产生行动时才有帮助。由于我们的数据并非全部都在 Dask 中，因此从世界其他地方读取和写入数据至关重要。到目前为止，本书中的示例主要使用了本地集合，但您有更多选择。

Dask 支持读取和写入许多标准文件格式和文件系统。这些格式包括 CSV、HDF、定宽、Parquet 和 ORC。Dask 支持许多标准的分布式文件系统，从 HDFS 到 S3，以及从常规文件系统读取。

对于 Dask 最重要的是，分布式文件系统允许多台计算机读取和写入相同的文件集。分布式文件系统通常在多台计算机上存储数据，这允许存储比单台计算机更多的数据。通常情况下，分布式文件系统也具有容错性（通过复制来实现）。分布式文件系统可能与您习惯的工作方式有重要的性能差异，因此重要的是查看您正在使用的文件系统的用户文档。需要关注的一些内容包括块大小（通常不希望写入比这些更小的文件，因为其余部分是浪费空间）、延迟和一致性保证。

###### 提示

在 Dask 中从常规本地文件读取可能会很复杂，因为文件需要存在于所有工作节点上。如果文件仅存在于主节点上，请考虑将其复制到像 S3 或 NFS 这样的分布式文件系统，或者在本地加载并使用 Dask 的 `client.scatter` 函数来分发数据（如果数据足够小）。足够小的文件*可能*表明你还不需要使用 Dask，除非对其进行处理很复杂或很慢。

## 格式

Dask 的 DataFrame 加载和写入函数以 `to_` 或 `read_` 作为前缀。每种格式都有自己的配置，但通常第一个位置参数是要读取的数据的位置。位置可以是文件的通配符路径（例如 *s3://test-bucket/magic/**）、文件列表或常规文件位置。

###### 注意

通配符路径仅适用于支持目录列表的文件系统。例如，它们在 HTTP 上不起作用。

在加载数据时，正确设置分区数量将加快所有操作的速度。有时无法以正确的分区数加载数据，在这种情况下，您可以在加载后重新分区数据。正如讨论的那样，更多的分区允许更多的并行处理，但也带来非零的开销。不同的格式有略微不同的控制方式。HDF 使用 `chunksize`，表示每个分区的行数。Parquet 也使用 `split_row_groups`，它接受一个整数，表示期望从 Parquet 文件中逻辑分区的划分，并且 Dask 将整个数据集分割成这些块，或更少。如果未指定，默认行为是每个分区对应一个 Parquet 文件。基于文本的格式（CSV、固定宽度等）使用 `blocksize` 参数，其含义与 Parquet 的 `chunksize` 相同，但最大值为 64 MB。您可以通过加载数据集并查看任务和分区数量随着较小的目标大小增加来验证这一点，就像 示例 4-1 中所示。

##### 示例 4-1\. 使用 1 KB 块加载 CSV 的 Dask DataFrame

```py
many_chunks = dd.read_csv(url, blocksize="1kb")
many_chunks.index
```

加载 CSV 和 JSON 文件可能比 Parquet 更复杂，而其他自描述数据类型没有编码任何模式信息。Dask DataFrame 需要知道不同列的类型，以正确地序列化数据。默认情况下，Dask 将自动查看前几条记录并猜测每列的数据类型。这个过程称为模式推断，但它可能相当慢。

不幸的是，模式推断并不总是有效。例如，如果尝试从 *https​://gender-pay-gap​.ser⁠vice.gov.uk/viewing/download-data/2021* 加载英国性别工资差距数据时，如同 示例 4-2 中所示，将会出现 “在 `pd.read​_csv`/`pd.read_table` 中找到的不匹配的数据类型” 的错误。当 Dask 的列类型推断错误时，您可以通过指定 `dtype` 参数（每列）来覆盖它，就像 示例 4-3 中所示。

##### 示例 4-2\. 使用完全依赖推断加载 CSV 的 Dask DataFrame

```py
df = dd.read_csv(
    "https://gender-pay-gap.service.gov.uk/viewing/download-data/2021")
```

##### 示例 4-3\. 使用指定数据类型加载 CSV 的 Dask DataFrame

```py
df = dd.read_csv(
    "https://gender-pay-gap.service.gov.uk/viewing/download-data/2021",
    dtype={'CompanyNumber': 'str', 'DiffMeanHourlyPercent': 'float64'})
```

###### 注意

在理论上，通过使用 `sample` 参数并指定更多字节，可以让 Dask 采样更多记录，但目前这并不能解决问题。当前的采样代码并没有严格遵守请求的字节数量。

即使模式推断没有返回错误，完全依赖它也有许多缺点。模式推断涉及对数据的抽样，因此其结果既是概率性的又很慢。在可以的情况下，应使用自描述格式或避免模式推断；这样可以提高数据加载速度并增强可靠性。您可能会遇到的一些常见自描述格式包括 Parquet、Avro 和 ORC。

读取和写入新文件格式是一项繁重的工作，特别是如果没有现成的 Python 库。如果有现成的库，您可能会发现将原始数据读入一个包并使用`map`函数解析它会更容易，我们将在下一章进一步探讨这一点。

###### 小贴士

Dask 在加载时不会检测排序数据。相反，如果您有预排序数据，在设置索引时添加`sorted=true`参数可以利用您已经排序的数据，这是您将在下一节中学习的步骤。但是，如果在数据未排序时指定此选项，则可能会导致数据静默损坏。

您还可以将 Dask 连接到数据库或微服务。关系型数据库是一种很棒的工具，通常在简单读写方面表现出色。通常，关系型数据库支持分布式部署，其中数据分割在多个节点上，这在处理大型数据集时经常使用。关系型数据库通常非常擅长处理大规模的事务，但在同一节点上运行分析功能可能会遇到问题。Dask 可用于有效地读取和计算 SQL 数据库中的数据。

您可以使用 Dask 的内置支持通过 SQLAlchemy 加载 SQL 数据库。为了让 Dask 在多台机器上拆分查询，您需要给它一个索引键。通常，SQL 数据库会有一个主键或数字索引键，您可以用于此目的（例如，`read_sql_table("customers", index_col="customer_id")`）。示例在示例 4-4 中展示了这一点。

##### 示例 4-4\. 使用 Dask DataFrame 从 SQL 读取和写入数据

```py
from sqlite3 import connect
from sqlalchemy import sql
import dask.dataframe as dd

#sqlite connection
db_conn = "sqlite://fake_school.sql"
db = connect(db_conn)

col_student_num = sql.column("student_number")
col_grade = sql.column("grade")
tbl_transcript = sql.table("transcripts")

select_statement = sql.select([col_student_num,
                              col_grade]
                              ).select_from(tbl_transcript)

#read from sql db
ddf = dd.read_sql_query(select_stmt,
                        npartitions=4,
                        index_col=col_student_num,
                        con=db_conn)

#alternatively, read whole table
ddf = dd.read_sql_table("transcripts",
                        db_conn,
                        index_col="student_number",
                        npartitions=4
                        )

#do_some_ETL...

#save to db
ddf.to_sql("transcript_analytics",
           uri=db_conn,
           if_exists='replace',
           schema=None,
           index=False
           )
```

更高级的与数据库或微服务的连接最好使用包接口并编写自定义加载代码，关于这一点您将在下一章中学到更多。

## 文件系统

加载数据可能是大量工作和瓶颈，因此 Dask 像大多数其他任务一样进行分布式处理。如果使用 Dask 分布式，每个工作节点必须能够访问文件以并行加载。与将文件复制到每个工作节点不同，网络文件系统允许每个人访问文件。Dask 的文件访问层使用 FSSPEC 库（来自 intake 项目）来访问不同的文件系统。由于 FSSPEC 支持一系列文件系统，因此它不会为每个支持的文件系统安装要求。使用示例 4-5 中的代码查看支持的文件系统及需要额外包的文件系统。

##### 示例 4-5\. 获取 FSSPEC 支持的文件系统列表

```py
from fsspec.registry import known_implementations
known_implementations
```

许多文件系统都需要某种配置，无论是端点还是凭证。通常新的文件系统，比如 MinIO，提供与 S3 兼容的 API，但超载端点并需要额外的配置才能正常运行。使用 Dask，您可以通过 `storage​_options` 参数来指定读写函数的配置参数。每个人的配置可能会有所不同。² Dask 将使用您的 `storage_options` 字典作为底层 FSSPEC 实现的关键字参数。例如，我对 MinIO 的 `storage_options` 如 示例 4-6 所示。

##### 示例 4-6\. 配置 Dask 以连接到 MinIO

```py
minio_storage_options = {
    "key": "YOURACCESSKEY",
    "secret": "YOURSECRETKEY",
    "client_kwargs": {
        "endpoint_url": "http://minio-1602984784.minio.svc.cluster.local:9000",
        "region_name": 'us-east-1'
    },
    "config_kwargs": {"s3": {"signature_version": 's3v4'}},
}
```

# 索引

在 DataFrame 中进行索引是 pandas 的强大功能之一，但在进入像 Dask 这样的分布式系统时，会有一些限制。由于 Dask 不跟踪每个分区的大小，不支持按行进行位置索引。您可以对列使用位置索引，以及对列或行使用标签索引。

索引经常用于过滤数据，仅保留您需要的组件。我们通过查看仅显示所有疫苗接种状态的人的案例率来处理旧金山 COVID-19 数据，如 示例 4-7 所示。

##### 示例 4-7\. Dask DataFrame 索引

```py
mini_sf_covid_df = (sf_covid_df
                    [sf_covid_df['vaccination_status'] == 'All']
                    [['specimen_collection_date', 'new_cases']])
```

如果您真的需要按行进行位置索引，请通过计算每个分区的大小并使用它来选择所需的分区子集来实现。这非常低效，因此 Dask 避免直接实现它；在执行此操作之前，请做出明智的选择。

# 洗牌

正如前一章所述，洗牌是昂贵的。导致洗牌昂贵的主要原因是在进程之间移动数据时的序列化开销，以及与从内存读取数据相比，网络的相对慢速。这些成本会随着被洗牌的数据量增加而增加，因此 Dask 有一些技术来减少被洗牌的数据量。这些技术取决于特定的数据属性或正在执行的操作。

## 滚动窗口和 map_overlap

触发洗牌的一种情况是滚动窗口，在分区的边缘，您的函数需要其邻居的一些记录。Dask DataFrame 具有特殊的 `map_overlap` 函数，您可以在其中指定一个*后视*窗口（也称为*向前*窗口）和一个*前视*窗口（也称为*向后*窗口）来传输行数（可以是整数或时间差）。利用此功能的最简单示例是滚动平均，如 示例 4-8 所示。

##### 示例 4-8\. Dask DataFrame 滚动平均

```py
def process_overlap_window(df):
    return df.rolling('5D').mean()

rolling_avg = partitioned_df.map_overlap(
    process_overlap_window,
    pd.Timedelta('5D'),
    0)
```

使用 `map_overlap` 允许 Dask 仅传输所需的数据。为使此实现正常工作，您的最小分区大小必须大于最大窗口。

###### 警告

Dask 的滚动窗口不会跨多个分区。如果你的 DataFrame 被分区，以至于向后或向前查看大于相邻分区的长度，结果将失败或不正确。Dask 对时间增量向后查看进行验证，但对向前查看或整数向后查看不执行此类检查。

解决 Dask 单分区向前/向后查看的有效但昂贵的技术是`repartition`你的 Dask DataFrames。

## 聚合

聚合是另一种特殊情况，可以减少需要通过网络传输的数据量。聚合是将记录组合的函数。如果你来自 map/reduce 或 Spark 背景，`reduceByKey`是经典的聚合函数。聚合可以是“按键”或全局的跨整个 DataFrame。

要按键聚合，首先需要使用表示键的列调用`groupby`，或用于聚合的键函数。例如，调用`df.groupby("PostCode")`按邮政编码对 DataFrame 进行分组，或调用`df.groupby(["PostCode", "SicCodes"])`使用多列进行分组。在功能上，许多与 pandas 相同的聚合函数可用，但 Dask 中的聚合性能与本地 pandas DataFrames 有很大不同。

###### 提示

如果按分区键聚合，Dask 可以在不需要洗牌的情况下计算聚合结果。

加快聚合的第一种方法是减少正在进行聚合的列，因为处理速度最快的数据是没有数据。最后，如果可能的话，同时进行多次聚合减少了需要洗牌同样数据的次数。因此，如果需要计算平均值和最大值，应同时计算两者（见示例 4-9）。

##### 示例 4-9\. Dask DataFrame 最大值和平均值

```py
dask.compute(
    raw_grouped[["new_cases"]].max(),
    raw_grouped[["new_cases"]].mean())
```

对于像 Dask 这样的分布式系统，如果可以部分评估然后合并聚合结果，你可以在洗牌之前组合一些记录。并非所有部分聚合都是相同的。部分聚合的关键在于，与原始的多个值相比，在合并相同键的值时数据量有所减少。

最有效的聚合需要亚线性数量的空间，不管记录的数量如何。其中一些，如 sum、count、first、minimum、maximum、mean 和 standard deviation，可以占用恒定空间。更复杂的任务，如分位数和不同计数，也有亚线性的近似选项。这些近似选项非常好用，因为精确答案可能需要存储的线性增长。³

有些聚合函数在增长上不是亚线性的，但往往或可能增长不是太快。计数不同值属于此类，但如果所有值都是唯一的，则没有节省空间。

要利用高效的聚合功能，您需要使用来自 Dask 的内置聚合，或者使用 Dask 的聚合类编写自己的聚合方法。在可以的情况下，使用内置聚合。内置聚合不仅需要更少的工作量，而且通常更快。并非所有的 pandas 聚合在 Dask 中都直接支持，因此有时你唯一的选择是编写自己的聚合。

如果选择编写自己的聚合，需要定义三个函数：`chunk`用于处理每个组-分区/块，`agg`用于在分区之间组合`chunk`的结果，以及（可选的）`finalize`用于获取`agg`的结果并生成最终值。

理解如何使用部分聚合的最快方法是查看一个使用所有三个函数的示例。在示例 4-10 中使用加权平均值可以帮助你思考每个函数所需的内容。第一个函数需要计算加权值和权重。`agg`函数通过对元组的每一部分进行求和来结合这些值。最后，`finalize`函数通过权重将总和除以。

##### 示例 4-10\. Dask 自定义聚合

```py
# Write a custom weighted mean, we get either a DataFrameGroupBy
# with multiple columns or SeriesGroupBy for each chunk
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

aggregated = (df_diff_with_emp_size.groupby("PostCode")
              ["EmployerSize", "DiffMeanHourlyPercent"].agg(weighted_mean))
```

在某些情况下，例如纯粹的求和，您不需要在`agg`的输出上进行任何后处理，因此可以跳过`finalize`函数。

并非所有的聚合都必须按键进行；您还可以跨所有行计算聚合。然而，Dask 的自定义聚合接口仅在按键操作时才暴露出来。

Dask 的内置完整 DataFrame 聚合使用一个称为`apply_contact_apply`的低级接口进行部分聚合。与学习两种不同的部分聚合 API 相比，我们更喜欢通过提供一个常量分组函数来进行静态的`groupby`。这样，我们只需了解一个聚合的接口。您可以使用此方法在 DataFrame 中查找聚合 COVID-19 数字，如示例 4-11 所示。

##### 示例 4-11\. 跨整个 DataFrame 进行聚合

```py
raw_grouped = sf_covid_df.groupby(lambda x: 0)
```

当存在内置聚合时，它很可能比我们编写的任何内容都要好。有时，部分聚合是部分实现的，例如 Dask 的 HyperLogLog：它仅适用于完整的 DataFrames。您通常可以通过复制`chunk`函数，使用`agg`的`combine`参数以及`finalize`的`aggregate`参数来转换简单的聚合。这通过在示例 4-12 中移植 Dask 的 HyperLogLog 实现来展示。

##### 示例 4-12\. 使用`dd.Aggregation`包装 Dask 的 HyperLogLog

```py
# Wrap Dask's hyperloglog in dd.Aggregation

from dask.dataframe import hyperloglog

approx_unique = dd.Aggregation(
    name='approx_unique',
    chunk=hyperloglog.compute_hll_array,
    agg=hyperloglog.reduce_state,
    finalize=hyperloglog.estimate_count)

aggregated = (df_diff_with_emp_size.groupby("PostCode")
              ["EmployerSize", "DiffMeanHourlyPercent"].agg(weighted_mean))
```

缓慢/低效的聚合操作（或者那些很可能导致内存不足异常的操作）使用与被聚合的记录数量成比例的存储空间。这些缓慢的操作包括制作列表和简单计算精确分位数。⁴ 在这些缓慢的聚合操作中，使用 Dask 的聚合类与 `apply` API 没有任何优势，后者可能更简单。例如，如果您只想要一个按邮政编码分组的雇主 ID 列表，而不必编写三个函数，可以使用像 `df.groupby("PostCode")["EmployerId"].apply(lambda g: list(g))` 这样的一行代码。Dask 将 `apply` 函数实现为一个完全的洗牌，这在下一节中有详细介绍。

###### 警告

Dask 在使用 `apply` 函数时无法应用部分聚合。

## 完全洗牌和分区

如果 Dask 内部的操作比在本地 DataFrame 中的预期要慢，可能是因为它需要进行完全洗牌。例如，排序就是一个例子，因为在分布式系统中排序通常需要进行洗牌，所以它本质上是昂贵的。在 Dask 中，有时完全洗牌是无法避免的。与完全洗牌本身慢速的相反，您可以使用它们来加速将来在相同分组键上进行的操作。正如在聚合部分提到的那样，触发完全洗牌的一种方式是在不对齐分区的情况下使用 `apply` 方法。

### 分区

在重新分区数据时，您最常使用完全洗牌。在处理聚合、滚动窗口或查找/索引时，拥有正确的分区非常重要。正如在滚动窗口部分讨论的那样，Dask 不能做超过一个分区的向前或向后查找，因此需要正确的分区才能获得正确的结果。对于大多数其他操作，错误的分区将减慢作业速度。

Dask 有三种主要方法来控制 DataFrame 的分区：`set_index`、`repartition` 和 `shuffle`（参见表 4-1）。当将分区更改为新的键/索引时，使用 `set_index`。`repartition` 保持相同的键/索引，但更改了分割。`repartition` 和 `set_index` 使用类似的参数，`repartition` 不需要索引键名称。一般来说，如果不更改用于索引的列，应该使用 `repartition`。`shuffle` 稍有不同，因为它不会产生类似于 `groupby` 可以利用的已知分区方案。

表 4-1\. 控制分区的函数

| 方法 | 更改索引键 | 设置分区数 | 导致已知分区方案 | 理想使用情况 |
| --- | --- | --- | --- | --- |
| `set_index` | 是 | 是 | 是 | 更改索引键 |
| `repartition` | 否 | 是 | 是 | 增加/减少分区数 |
| `shuffle` | 否 | 是 | 否 | 键的分布倾斜^(a) |
| ^(a) 为分布哈希键，可以帮助随机分布倾斜数据 *如果* 键是唯一的（但是集中的）。 |

为了为 DataFrame 获取正确的分区，第一步是决定是否需要索引。索引在按索引值过滤数据、索引、分组以及几乎任何其他按键操作时都非常有用。其中一种按键操作是 `groupby`，其中被分组的列可以是一个很好的键候选。如果您在列上使用滚动窗口，该列必须是键，这使得选择键相对容易。一旦确定了索引，您可以使用索引列名称调用 `set_index`（例如，`set_index("PostCode")`）。这通常会导致 shuffle，因此现在是调整分区大小的好时机。

###### 提示

如果您不确定当前用于分区的键是什么，可以检查 `index` 属性以查看分区键。

选择了键之后，下一个问题是如何设置分区大小。通常适用于这里的建议是 “分区/块集合”：尝试保持足够的分区以使每台机器保持忙碌，但请记住大约在 100 MB 到 1 GB 的一般甜点。如果给定目标分区数，Dask 通常会计算出相当均匀的分割。⁵ 幸运的是，`set_index` 也将接受 `npartitions`。要通过邮政编码重新分区数据，使用 10 个分区，您可以添加 `set_index("PostCode", npartitions=10)`；否则，Dask 将默认使用输入分区数。

如果您计划使用滚动窗口，您可能需要确保每个分区覆盖了正确大小的键范围。作为 `set_index` 的一部分，您需要计算自己的分区来确保每个分区具有正确范围的记录。分区被指定为列表，从第一个分区的最小值到最后一个分区的最大值。在构建由 Pandas DataFrame 组成的 Dask DataFrame 的分区 `[0, 100) [100, 200), [200, 300), [300, 500)`，您可以编写 `df.set_index("NumEmployees", divisions=[0, 100, 200, 300, 500])`。类似地，为了支持从 COVID-19 疫情开始到今天最多七天的滚动窗口的日期范围，请参见 Example 4-13。

##### 示例 4-13\. 使用 `set_index` 的 Dask DataFrame 滚动窗口

```py
divisions = pd.date_range(
    start="2021-01-01",
    end=datetime.today(),
    freq='7D').tolist()
partitioned_df_as_part_of_set_index = mini_sf_covid_df.set_index(
    'specimen_collection_date', divisions=divisions)
```

###### 警告

Dask，包括用于滚动时间窗口，假设您的分区索引是单调递增的。⁶

到目前为止，你必须指定分区的数量或具体的分割点，但你可能想知道 Dask 是否可以自己找出这些。幸运的是，Dask 的 repartition 函数有能力为给定的目标大小选择分割点，就像在 Example 4-14 中展示的那样。然而，这样做会有一个不可忽视的成本，因为 Dask 必须评估 DataFrame 以及重新分区本身。

##### Example 4-14\. Dask DataFrame 自动分区

```py
reparted = indexed.repartition(partition_size="20kb")
```

###### 警告

截至本文撰写时，Dask 的`set_index`有一个类似的`partition_size`参数，但仅适用于减少分区的数量。

正如你在本章开头看到的，当写入一个 DataFrame 时，每个分区都有其自己的文件，但有时这会导致文件过大或过小。有些工具只能接受一个文件作为输入，因此你需要将所有内容重新分区为单个分区。其他时候，数据存储系统被优化为特定的文件大小，例如 HDFS 的默认块大小为 128 MB。好消息是，诸如`repartition`和`set_index`的技术已经为你解决了这些问题。

# 尴尬的并行操作

Dask 的`map_partitions`函数将一个函数应用于底层 pandas DataFrame 的每个分区，结果也是一个 pandas DataFrame。使用`map_partitions`实现的函数是尴尬的并行，因为它们不需要任何数据的跨 worker 传输。⁷ Dask 实现了`map`与`map_partitions`，以及许多逐行操作。如果你想使用一个你找不到的逐行操作，你可以自己实现，就像在 Example 4-15 中展示的那样。

##### Example 4-15\. Dask DataFrame `fillna`

```py
def fillna(df):
    return df.fillna(value={"PostCode": "UNKNOWN"}).fillna(value=0)

new_df = df.map_partitions(fillna)
# Since there could be an NA in the index clear the partition / division
# information
new_df.clear_divisions()
```

你并不局限于调用 pandas 内置函数。只要你的函数接受并返回一个 DataFrame，你几乎可以在`map_partitions`中做任何你想做的事情。

完整的 pandas API 在本章中太长无法涵盖，但如果一个函数可以在不知道前后行的情况下逐行操作，那么它可能已经在 Dask DataFrame 中使用`map_partitions`实现了。

当在一个 DataFrame 上使用`map_partitions`时，你可以改变每行的任何内容，包括它分区的键。如果你改变了分区键中的值，你*必须*用`clear_divisions()`清除结果 DataFrame 上的分区信息，*或者*用`set_index`指定正确的索引，这个你将在下一节学到更多。

###### 警告

不正确的分区信息可能导致不正确的结果，而不仅仅是异常，因为 Dask 可能会错过相关数据。

# 处理多个 DataFrame

Pandas 和 Dask 有四个常用的用于组合 DataFrame 的函数。在根上是 `concat` 函数，它允许您在任何轴上连接 DataFrames。由于涉及到跨 worker 的通信，Dask 中的 DataFrame 连接通常较慢。另外三个函数是 `join`、`merge` 和 `append`，它们都在 `concat` 的基础上针对常见情况实现了特殊处理，并具有略微不同的性能考虑。在处理多个 DataFrame 时，通过良好的分区和键选择，尤其是分区数量，可以显著提升性能。

Dask 的 `join` 和 `merge` 函数接受大多数标准的 pandas 参数，还有一个额外的可选参数 `npartitions`。 `npartitions` 指定了目标输出分区的数量，但仅在哈希连接中使用（您将在 “多 DataFrame 内部” 中了解到）。这两个函数会根据需要自动重新分区您的输入 DataFrames。这非常好，因为您可能不了解分区情况，但由于重新分区可能很慢，当您不希望进行任何分区更改时，明确使用较低级别的 `concat` 函数可以帮助及早发现性能问题。Dask 的 `join` 在进行*left*或*outer*连接类型时可以一次处理多个 DataFrame。

###### 提示

Dask 具有特殊逻辑，可加速多个 DataFrame 的连接操作，因此在大多数情况下，您可以通过执行 `a.join([b, c, d, e])` 而不是 `a.join(b).join(c).join(d).join(e)` 获得更好的性能。但是，如果您正在执行与小数据集的左连接，则第一种语法可能更有效。

当您按行合并或 `concat` DataFrames（类似于 SQL UNION）时，性能取决于被合并 DataFrames 的分区是否 *well ordered*。如果一系列 DataFrame 的分区是良好排序的，那么所有分区都是已知的，并且前一个 DataFrame 的最高分区低于下一个 DataFrame 的最低分区，则我们称这些 DataFrame 的分区是良好排序的。如果任何输入具有未知分区，Dask 将产生一个没有已知分区的输出。对于所有已知分区，Dask 将行合并视为仅元数据的更改，并且不会执行任何数据重排。这要求分区之间没有重叠。还有一个额外的 `interleave_partitions` 参数，它将行合并的连接类型更改为无输入分区限制的连接类型，并导致已知的分区结果。具有已知分区的 Dask DataFrame 可以通过键支持更快的查找和操作。

Dask 的基于列的`concat`（类似于 SQL JOIN）在合并的 DataFrame 的分区/分片上也有限制。Dask 的`concat`仅支持内连接或全外连接，不支持左连接或右连接。基于列的连接要求所有输入具有已知的分区器，并且结果是具有已知分区的 DataFrame。拥有已知的分区器对后续的连接非常有用。

###### 警告

在处理具有未知分区的 DataFrame 时，不要使用 Dask 的`concat`，因为它可能会返回不正确的结果。⁸

## 多 DataFrame 内部

Dask 使用四种技术——哈希、广播、分区和`stack_partitions`——来组合 DataFrame，每种技术的性能差异很大。这四个函数与您从中选择的连接函数并不一一对应。相反，Dask 根据索引、分区和请求的连接类型（例如外部/左/内部）选择技术。三种基于列的连接技术是哈希连接、广播连接和分区连接。在进行基于行的组合（例如`append`）时，Dask 具有一种称为`stack_partitions`的特殊技术，速度特别快。重要的是，您理解每种技术的性能以及 Dask 选择每种方法的条件：

哈希连接

当没有其他适合的连接技术时，Dask 默认使用哈希连接。哈希连接会对所有输入的 DataFrame 进行数据分区，以目标键为基准。它使用键的哈希值，导致结果 DataFrame 没有特定的顺序。因此，哈希连接的结果没有任何已知的分区。

广播连接

适用于将大型 DataFrame 与小型 DataFrame 连接。在广播连接中，Dask 获取较小的 DataFrame 并将其分发到所有工作节点。这意味着较小的 DataFrame 必须能够适应内存。要告诉 Dask 一个 DataFrame 适合广播，请确保它全部存储在一个分区中，例如通过调用`repartition(npartitions=1)`。

分区连接

在沿索引组合 DataFrame 时发生，其中所有 DataFrame 的分区/分片都是已知的。由于输入的分区是已知的，Dask 能够在 DataFrame 之间对齐分区，涉及的数据传输更少，因为每个输出分区都包含不到完整输入集的数据。

由于分区连接和广播连接速度更快，为帮助 Dask 做一些工作是值得的。例如，将几个具有已知和对齐分区/分片的 DataFrame 连接到一个未对齐的 DataFrame 会导致昂贵的哈希连接。相反，尝试设置剩余 DataFrame 的索引和分区，或者先连接较便宜的 DataFrame，然后再执行昂贵的连接。

第四种技术 `stack_partitions` 与其他选项不同，因为它不涉及任何数据的移动。相反，生成的 DataFrame 分区列表是输入 DataFrames 的上游分区的并集。Dask 在大多数基于行的组合中使用 `stack_partitions` 技术，除非所有输入 DataFrame 的分区都已知，它们没有被很好地排序，并且你要求 Dask `interleave_partitions`。在输出中，`stack_partitions` 技术只有在输入分区已知且有序时才能提供已知分区。如果所有分区都已知但排序不好，并且你设置了 `interleave​_parti⁠tions`，Dask 将使用分区连接。虽然这种方法相对廉价，但并非免费，而且可能导致分区数量过多，需要重新分区。

## 缺失功能

并非所有的多数据框操作都已实现，比如 `compare`，这将我们引入关于 Dask DataFrames 限制的下一节。

# 不起作用的功能

Dask 的 DataFrame 实现了大部分但并非全部的 pandas DataFrame API。由于开发时间的原因，Dask 中未实现部分 pandas API。其他部分则未使用，以避免暴露可能意外缓慢的 API。

有时候 API 只是缺少一些小部分，因为 pandas 和 Dask 都在积极开发中。一个例子是来自 Example 2-10 的 `split` 函数。在本地 pandas 中，你可以调用 `split(expand=true)` 而不是 `split().explode()`。如果你有兴趣，这些缺失部分可以是你参与并 [贡献到 Dask 项目](https://oreil.ly/Txd_R) 的绝佳机会。

有些库并不像其他那样有效地并行化。在这些情况下，一种常见的方法是尝试将数据筛选或聚合到足够可以在本地表示的程度，然后再将本地库应用到数据上。例如，在绘图时，通常会预先聚合计数或取随机样本并绘制结果。

虽然大部分 pandas DataFrame API 可以正常工作，但在你切换到 Dask DataFrame 之前，确保有充分的测试覆盖来捕捉它不适用的情况是非常重要的。

# 速度慢

通常情况下，使用 Dask DataFrames 会提高性能，但并非总是如此。一般来说，较小的数据集在本地 pandas 中表现更好。正如讨论的那样，在涉及到洗牌的任何情况下，分布式系统中通常比本地系统慢。迭代算法也可能产生大量的操作图，这在 Dask 中与传统的贪婪评估相比，评估速度较慢。

某些问题通常不适合数据并行计算。例如，写入带有单个锁的数据存储，其具有更多并行写入者，将增加锁竞争并可能比单个线程进行写入更慢。在这些情况下，有时可以重新分区数据或写入单个分区以避免锁竞争。

# 处理递归算法

Dask 的惰性评估，由其谱系图支持，通常是有益的，允许它自动组合步骤。然而，当图变得过大时，Dask 可能会难以管理，通常表现为驱动进程或笔记本运行缓慢，有时会出现内存不足的异常。幸运的是，您可以通过将 DataFrame 写出并重新读取来解决这个问题。一般来说，Parquet 是这样做的最佳格式，因为它在空间上高效且自我描述，因此无需进行模式推断。

# 重新计算的数据

惰性评估的另一个挑战是如果您想多次重用一个元素。例如，假设您想加载几个 DataFrame，然后计算多个信息片段。您可以要求 Dask 通过运行 `client.persist(collection)` 将集合（包括 DataFrame、Series 等）保存在内存中。并非所有重新计算的数据都需要避免；例如，如果加载 DataFrame 很快，不持久化它们可能是可以接受的。

###### 警告

与 Apache Spark 明显不同，像 Dask 的其他函数一样，`persist()` 不会修改 DataFrame — 如果您在其上调用函数，数据仍然会重新计算。

# 其他函数的不同之处

由于性能原因，Dask DataFrame 的各个部分行为可能与本地 DataFrame 稍有不同：

`reset_index`

每个分区的索引将在零点重新开始。

`kurtosis`

此函数不会过滤掉 NaN，并使用 SciPy 的默认值。

`concat`

不同于强制转换类别类型，每个类别类型都会扩展到与其连接的所有类别的并集。

`sort_values`

Dask 仅支持单列排序。

连接多个 DataFrame

当同时连接两个以上的 DataFrame 时，连接类型必须是 outer 或 left。

在将代码移植到使用 Dask DataFrame 时，您应特别注意任何时候使用这些函数，因为它们可能不会完全在您预期的轴上工作。首先进行小范围测试，并测试数字的正确性，因为问题通常很难追踪。

在将现有的 pandas 代码移植到 Dask 时，请考虑使用本地单机版本生成测试数据集，以便与结果进行比较，以确保所有更改都是有意的。

# 使用 Dask DataFrame 进行数据科学：将其放在一起

Dask DataFrame 已经被证明是用于大数据的流行框架，因此我们希望强调一个常见的用例和考虑因素。在这里，我们使用一个经典的数据科学挑战数据集，即纽约市黄色出租车，并介绍一个数据工程师处理此数据集可能考虑的内容。在涵盖机器学习工作负载的后续章节中，我们将使用许多 DataFrame 工具来构建。

## 决定使用 Dask

正如前面讨论的，Dask 在数据并行任务中表现出色。一个特别适合的数据集是可能已经以列格式，如 Parquet 格式，可用的数据集。我们还评估数据存储在哪里，例如在 S3 或其他远程存储选项中。许多数据科学家和工程师可能会有一个不能在单台机器上容纳或由于合规性约束而无法在本地存储的数据集。Dask 的设计非常适合这些用例。

我们的 NYC 出租车数据符合所有这些标准：数据以 Parquet 格式由纽约市存储在 S3 中，并且它可以轻松地进行横向和纵向扩展，因为它按日期进行了分区。此外，我们评估数据已经结构化，因此我们可以使用 Dask DataFrame。由于 Dask DataFrames 和 pandas DataFrames 相似，我们还可以使用许多现有的 pandas 工作流。我们可以对其中一些样本进行采样，在较小的开发环境中进行探索性数据分析，然后使用相同的代码扩展到完整数据集。请注意，在示例 4-16 中，我们使用行组来指定分块行为。

##### 示例 4-16\. 使用 Dask DataFrame 加载多个 Parquet 文件

```py
filename = './nyc_taxi/*.parquet'
df_x = dd.read_parquet(
    filename,
    split_row_groups=2
)
```

## 使用 Dask 进行探索性数据分析

数据科学的第一步通常包括探索性数据分析（EDA），或者了解数据集并绘制其形状。在这里，我们使用 Dask DataFrames 来走过这个过程，并检查由于 pandas DataFrame 和 Dask DataFrame 之间微妙差异而引起的常见故障排除问题。

## 加载数据

第一次将数据加载到您的开发环境中时，您可能会遇到块大小问题或模式问题。虽然 Dask 尝试推断两者，但有时会失败。块大小问题通常会在您对微不足道的代码调用`.compute()`时出现，看到一个工作线程达到内存限制。在这种情况下，需要进行一些手动工作来确定正确的块大小。模式问题将显示为读取数据时的错误或警告，或者稍后以微妙的方式显示，例如不匹配的 float32 和 float64。如果您已经了解模式，建议在读取时通过指定 dtype 来强制执行。

在进一步探索数据集时，您可能会遇到默认以您不喜欢的格式打印的数据，例如科学计数法。这可以通过 pandas 而不是 Dask 本身来控制。Dask 隐式调用 pandas，因此您希望使用 pandas 显式设置您喜欢的格式。

数据的汇总统计工作类似于 pandas 的`.describe()`，还可以指定百分位数或`.quantile()`。请注意，如果运行多个这样的计算，请链式调用它们，这样可以节省计算时间。在 示例 4-17 中展示了如何使用 Dask DataFrame 的 `describe` 方法。

##### 示例 4-17\. 使用漂亮格式描述百分位数的 Dask DataFrame

```py
import pandas as pd

pd.set_option('display.float_format', lambda x: '%.5f' % x)
df.describe(percentiles=[.25, .5, .75]).compute()
```

## 绘制数据

绘制数据通常是了解数据集的重要步骤。绘制大数据是一个棘手的问题。作为数据工程师，我们经常通过首先使用较小的采样数据集来解决这个问题。为此，Dask 可以与 Python 绘图库（如 matplotlib 或 seaborn）一起使用，就像 pandas 一样。Dask DataFrame 的优势在于，现在我们可以绘制整个数据集（如果需要的话）。我们可以使用绘图框架以及 Dask 来绘制整个数据集。在这里，Dask 进行筛选、分布式工作节点的聚合，然后收集到一个非分布式库（如 matplotlib）来渲染的工作节点。Dask DataFrame 的绘图示例显示在 示例 4-18 中。

##### 示例 4-18\. Dask DataFrame 绘制行程距离

```py
import matplotlib.pyplot as plt
import seaborn as sns 
import numpy as np

get_ipython().run_line_magic('matplotlib', 'inline')
sns.set(style="white", palette="muted", color_codes=True)
f, axes = plt.subplots(1, 1, figsize=(11, 7), sharex=True)
sns.despine(left=True)
sns.distplot(
    np.log(
        df['trip_distance'].values +
        1),
    axlabel='Log(trip_distance)',
    label='log(trip_distance)',
    bins=50,
    color="r")
plt.setp(axes, yticks=[])
plt.tight_layout()
plt.show()
```

###### 小贴士

请注意，如果你习惯于 NumPy 的逻辑，绘制时需要考虑到 Dask DataFrame 层。例如，NumPy 用户会熟悉 `df[col].values` 语法用于定义绘图变量。在 Dask 中，`.values` 执行的操作不同；我们传递的是 `df[col]`。

## 检查数据

Pandas DataFrame 用户熟悉`.loc()`和`.iloc()`用于检查特定行或列的数据。这种逻辑转换到 Dask DataFrame，但是`.iloc()`的行为有重要的区别。

充分大的 Dask DataFrame 将包含多个 pandas DataFrame。这改变了我们应该如何思考编号和索引的方式。例如，对于 Dask，像`.iloc()`（通过索引访问位置的方法）不会完全像 pandas 那样工作，因为每个较小的 DataFrame 都有自己的`.iloc()`值，并且 Dask 不会跟踪每个较小 DataFrame 的大小。换句话说，全局索引值对于 Dask 来说很难确定，因为 Dask 将不得不逐个计算每个 DataFrame 才能获得索引。用户应该检查他们的 DataFrame 上的`.iloc()`并确保索引返回正确的值。

###### 小贴士

请注意，调用像`.reset_index()`这样的方法可能会重置每个较小的 DataFrame 中的索引，当用户调用`.iloc()`时可能返回多个值。

# 结论

在本章中，你已经了解到了如何理解 Dask 中哪些操作比你预期的更慢。你还学到了一些处理 pandas DataFrames 和 Dask DataFrames 性能差异的技术。通过理解 Dask DataFrames 性能可能不符合需求的情况，你也了解到了哪些问题不适合使用 Dask。为了能够综合这些内容，你还了解了 Dask DataFrame 的 IO 选项。从这里开始，你将继续学习有关 Dask 的其他集合，然后再进一步了解如何超越集合。 

在本章中，你已经了解到可能导致 Dask DataFrames 行为与预期不同或更慢的原因。对于 Dask DataFrames 实现方式的相同理解可以帮助你确定分布式 DataFrames 是否适合你的问题。你还看到了如何将超过单台机器处理能力的数据集导入和导出 Dask 的 DataFrames。

¹ 参见“分区/分块集合”进行分区的复习。

² [FSSPEC 文档](https://oreil.ly/ZfcRv)包含配置每个后端的具体信息。

³ 这可能导致在执行聚合时出现内存不足异常。存储空间的线性增长要求（在一个常数因子内）所有数据都必须能够适应单个进程，这限制了 Dask 的有效性。

⁴ 准确分位数的备选算法依赖更多洗牌操作来减少空间开销。

⁵ 键偏斜可能会使已知分区器无法处理。

⁶ 严格递增且无重复值（例如，1、4、7 是单调递增的，但 1、4、4、7 不是）。

⁷ [尴尬并行问题](https://oreil.ly/30938)是指分布式计算和通信的开销很低的问题。

^   ⁸ 当不存在索引时，Dask 假定索引是对齐的。
