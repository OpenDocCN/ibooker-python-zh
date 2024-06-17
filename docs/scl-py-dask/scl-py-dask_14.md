# 附录 B. 可扩展数据框架：比较和一些历史

Dask 的分布式类似于 pandas 的 DataFrame，在我们看来是其关键特性之一。存在各种方法提供可扩展的类似 DataFrame 的功能。使得 Dask 的 DataFrame 脱颖而出的一个重要因素是对 pandas API 的高度支持，其他项目正在迅速赶上。本附录比较了一些不同的当前和历史数据框架库。

要理解这些差异，我们将看几个关键因素，其中一些与我们在第八章中建议的技术类似。首先是 API 的外观，以及使用 pandas 的现有技能和代码可以转移多少。然后我们将看看有多少工作被强制在单个线程、驱动程序/主节点上进行，然后在单个工作节点上进行。

可扩展数据框架并不一定意味着分布式，尽管分布式扩展通常允许处理比单机选项更大的数据集更经济实惠，并且在真正大规模的情况下，这是唯一的实际选择。

# 工具

许多工具中常见的一个依赖是它们建立在 ASF Arrow 之上。虽然 Arrow 是一个很棒的项目，我们希望看到它持续被采纳，但它在[类型差异](https://oreil.ly/VPyAL)方面有些差异，特别是在可空性方面。¹ 这些差异意味着大多数使用 Arrow 构建的系统共享一些共同的限制。

开放多处理（OpenMP）和开放消息传递接口（OpenMPI）是许多这些工具依赖的另外两个常见依赖项。尽管它们有类似的缩写，你通常会看到它们被称为，但它们采用了根本不同的并行化方法。OpenMP 是一个专注于共享内存的单机工具（可能存在非均匀访问）。OpenMPI 支持多台机器，而不是共享内存，使用消息传递（在概念上类似于 Dask 的 Actor 系统）进行并行化。

## 仅限单机

单机可扩展数据框架专注于并行化计算或允许数据不同时驻留在内存中（例如，一些可以驻留在磁盘上）。在某种程度上，这种“数据可以驻留在磁盘上”的方法可以通过操作系统级别的交换文件来解决，但实际上，让库在元素的智能页面进出中进行智能分页也具有其优点。

### Pandas

在讨论缩放 DataFrame 的部分中提到 pandas 可能看起来有些愚蠢，但记住我们比较的基准是什么是有用的。总体而言，Pandas 是单线程的，要求所有数据都适合单台机器的内存。可以使用各种技巧来处理 pandas 中更大的数据集，如创建大交换文件或逐个处理较小的块。需要注意的是，许多这些技术都已纳入用于扩展 pandas 的工具中，因此如果您需要这样做，现在可能是开始探索扩展选项的时候了。另一方面，如果在 pandas 中一切正常运行，通过使用 pandas 本身可以获得 100% 的 pandas API 兼容性，这是其他选项无法保证的。另外，[pandas 是直接要求，而不是可扩展 pandas 工具之一](https://oreil.ly/IzYDb)。

### H2O 的 DataTable

DataTable 是一个类似于 DataFrame 的单机尝试，旨在扩展处理能力达到 100 GB（尽管项目作者将其描述为“大数据”，我们认为它更接近中等规模数据）。尽管是为 Python 设计的，DataTable 并没有简单复制 pandas 的 API，而是致力于继承很多 R 的 `data.table` API。这使得它对于来自 R 的团队来说可能是一个很好的选择，但对于专注于 pandas 的用户来说可能不太吸引人。DataTable 也是一个单公司开源项目，存放在 H2O 的 GitHub 上，而不是在某个基金会或自己的平台上。在撰写本文时，它的[开发活动相对集中](https://oreil.ly/8vgA5)。它有积极的持续集成（在 PR 进来时运行），我们认为这表明它是高质量的软件。DataTable 可以使用 OpenMP 在单台机器上并行计算，但不要求使用 OpenMP。

### Polars

Polars 是另一个单机可扩展的 DataFrame，但它采用的方法是在 Rust 中编写其核心功能，而不是 C/C++ 或 Fortran。与许多分布式 DataFrame 工具类似，Polars 使用 ASF 的 Arrow 项目来存储 DataFrame。同样，Polars 使用惰性评估来管道化操作，并在内部分区/分块 DataFrame，因此（大部分时间）只需在任一时间内内存中保留数据的子集。Polars 在所有单机可扩展 DataFrame 中拥有[最大的开发者社区](https://oreil.ly/zxoFJ)。Polars 在其主页上链接到基准测试，显示其比许多分布式工具快得多，但仅当将分布式工具约束为单机时才有意义，这是不太可能的。它通过使用单台机器中的所有核心来实现其并行性。Polars 拥有[详尽的文档](https://oreil.ly/QW5s2)，并且还有一个明确的章节，介绍从常规 pandas 迁移时可以期待的内容。它不仅具有持续集成，而且还将基准测试集成为每个 PR 的一部分，并针对多个版本的 Python 和环境进行测试。

## 分布式

扩展 DataFrame 的大多数工具都具有分布式的特性，因为在单个机器上的所有花哨技巧只能带来有限的效果。

### ASF Spark DataFrame

Spark 最初以所谓的弹性分布式数据集（RDD）起步，然后迅速添加了更类似于 DataFrame 的 API，称为 DataFrames。这引起了很多兴奋，但许多人误解它是指“类似于 pandas”，而 Spark 的（最初的）DataFrames 更类似于“类似于 SQL 的”DataFrames。Spark 主要用 Scala 和 Java 编写，两者都运行在 Java 虚拟机（JVM）上。虽然 Spark 有 Python API，但它涉及 JVM 和 Python 之间大量数据传输，这可能很慢，并且可能增加内存需求。Spark DataFrames 在 ASF Arrow 之前创建，因此具有其自己的内存存储格式，但后来添加了对 Arrow 在 JVM 和 Python 之间通信的支持。

要调试 PySpark 错误尤其困难，因为一旦出错，你会得到一个 Java 异常和一个 Python 异常。

### SparklingPandas

由于 Holden 共同编写了 SparklingPandas，我们可以自信地说不要使用这个库，而不必担心会有人不高兴。SparklingPandas 建立在 ASF Spark 的 RDD 和 DataFrame API 之上，以提供更类似于 Python 的 API，但由于其标志是一只熊猫在便签纸上吃竹子，你可以看到我们并没有完全成功。SparklingPandas 确实表明通过重用 pandas 的部分内容可以提供类似 pandas 的体验。

对于尴尬并行类型的操作，通过使用 `map` 将 pandas API 的每个函数添加到每个 DataFrame 上，Python 代码的委托非常快速。一些操作，如 dtypes，仅在第一个 DataFrame 上评估。分组和窗口操作则更为复杂。

由于最初的合著者有其他重点领域的日常工作，项目未能超越概念验证阶段。

### Spark Koalas / Spark pandas DataFrames

Koalas 项目最初源自 Databricks，并已整合到 Spark 3.2 中。Koalas 采用类似的分块 pandas DataFrames 方法，但这些 DataFrames 表示为 Spark DataFrames 而不是 Arrow DataFrames。像大多数系统一样，DataFrames 被延迟评估以允许流水线处理。Arrow 用于将数据传输到 JVM 并从中传输数据，因此您仍然具有 Arrow 的所有类型限制。这个项目受益于成为一个庞大社区的一部分，并与传统的大数据堆栈大部分互通。这源自于作为 JVM 和 Hadoop 生态系统的一部分，但这也会带来性能上的一些不利影响。目前，在 JVM 和 Python 之间移动数据会增加开销，而且总体上，Spark 专注于支持更重的任务。

在 Spark Koalas / Spark pandas DataFrames 上的分组操作尚不支持部分聚合。这意味着一个键的所有数据必须适合一个节点。

### Cylon

Cylon 的主页非常专注于基准测试，但它选择的基准测试（将 Cylon 与 Spark 在单机上进行比较）很容易达到，因为 Spark 是设计用于分布式使用而不是单机使用。Cylon 使用 PyArrow 进行存储，并使用 OpenMPI 管理其任务并行性。Cylon 还有一个名为 GCylon 的 GPU 后端。PyClon 的文档还有很大的改进空间，并且当前的 API 文档链接已经失效。

Cylon 社区似乎每年有约 30 条消息，试图找到任何使用 DataFrame 库的开源用户 [没有结果](https://oreil.ly/uroxr)。[贡献者文件](https://oreil.ly/dWC16) 和 LinkedIn 显示大多数贡献者都来自同一所大学。

该项目遵循几个软件工程的最佳实践，如启用 CI。尽管如此，相对较小（明显活跃）的社区和缺乏清晰的文档意味着，在我们看来，依赖 Cylon 可能比其他选项更复杂。

### Ibis

[Ibis 项目](https://oreil.ly/9OL2f) 承诺“结合 Python 分析的灵活性和现代 SQL 的规模与性能”。它将你的代码编译成类似 pandas 的 SQL 代码（尽可能），这非常方便，因为许多大数据系统（如 Hive、Spark、BigQuery 等）支持 SQL，而且 SQL 是目前大多数数据库的事实标准查询语言。不幸的是，SQL 的实现并不统一，因此在不同后端引擎之间移动可能会导致故障，但 Ibis 在 [跟踪哪些 API 适用于哪些后端引擎](https://oreil.ly/g2E_W) 方面做得很好。当然，这种设计限制了你可以在 SQL 中表达的表达式类型。

### Modin

与 Ibis 类似，Modin 与许多其他工具略有不同，它具有多个分布式后端，包括 Ray、Dask 和 OpenMPI。Modin 的宣称目标是处理从 1 MB 到 1+ TB 的数据，这是一个广泛的范围。[Modin 的主页](https://modin.org) 还声称可以“通过更改一行代码扩展您的 pandas 工作流”，虽然这种说法有吸引力，但在我们看来，它对 API 兼容性和利用并行和分布式系统所需的知识要求做出了过多的承诺。³ 在我们看来，Modin 很令人兴奋，因为每个分布式计算引擎都有自己重新实现 pandas API 的需求看起来很愚蠢。Modin 有一个非常活跃的开发者社区，核心开发者来自多个公司和背景。另一方面，我们认为当前的文档并没有很好地帮助用户理解 Modin 的局限性。幸运的是，您对 Dask DataFrames 的大部分直觉在 Modin 中仍然适用。我们认为 Modin 对需要在不同计算引擎之间移动的个人用户来说是理想选择。

###### 警告

与其他系统不同，Modin 被积极评估，这意味着它不能利用自动流水线处理您的计算。

### Vanilla Dask DataFrame

我们在这里有偏见，但我们认为 Dask 的 DataFrame 库在平衡易于入门和明确其限制方面做得非常好。Dask 的 DataFrames 拥有来自多家不同公司的大量贡献者。Dask DataFrames 还具有相对高水平的并行性，包括对分组操作的支持，在许多其他系统中找不到。

### cuDF

cuDF 扩展了 Dask DataFrame，以支持 GPU。然而，它主要是一个单一公司项目，来自 NVIDIA。这是有道理的，因为 NVIDIA 希望卖更多的 GPU，但这也意味着它不太可能很快为 AMD GPU 添加支持。如果 NVIDIA 继续认为为数据分析销售更多 GPU 是最佳选择的话，该项目可能会得到维护，并保持类似 pandas 的接口。

cuDF 不仅具有 CI，而且具有区域责任的强大代码审查文化。

# 结论

在理想的世界中，会有一个明确的赢家，但正如你所见，不同的可扩展 DataFrame 库为不同目的提供服务，除了那些已经被放弃的，所有都有潜在的用途。我们认为所有这些库都有其位置，取决于您的确切需求。

¹ Arrow 允许所有数据类型为 null。Pandas 不允许整数列包含 null。当将 Arrow 文件读取为 pandas 时，如果一个整数列不包含 null，它将被读取为整数在 pandas DataFrame 中，但如果在运行时遇到 null，则整个列将被读取为浮点数。

² 除了我们自己之外，如果你正在阅读这篇文章，你可能已经帮助 Holden 买了一杯咖啡，那就足够了。:)

³ 例如，看看关于 groupBy + apply 的限制混乱，除了 [GitHub 问题](https://oreil.ly/rIeam) 外，没有其他文档。 
