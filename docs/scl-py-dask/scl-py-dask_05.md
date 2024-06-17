# 第五章。Dask 的集合

到目前为止，您已经看到了 Dask 是如何构建的基础知识，以及 Dask 如何利用这些构建模块支持数据科学与数据帧。本章探讨了 Dask 的 bag 和 array 接口的地方——相对于数据帧而言，这些接口经常被忽视更合适。正如在“Hello Worlds”中提到的，Dask 袋实现了常见的函数式 API，而 Dask 数组实现了 NumPy 数组的一个子集。

###### 提示

理解分区对于理解集合非常重要。如果您跳过了“分区/分块集合”，现在是回头看一看的好时机。

# Dask 数组

Dask 数组实现了 NumPy ndarray 接口的一个子集，使它们非常适合用于将使用 NumPy 的代码移植到 Dask 上运行。你从上一章对数据帧的理解大部分都适用于 Dask 数组，以及你对 ndarrays 的理解。

## 常见用例

Dask 数组的一些常见用例包括：

+   大规模成像和天文数据

+   天气数据

+   多维数据

与 Dask 数据帧和 pandas 类似，如果在较小规模问题上不使用 nparray，那么 Dask 数组可能不是正确的解决方案。

## 不适用 Dask 数组的情况

如果你的数据适合在单台计算机的内存中，使用 Dask 数组不太可能比 nparrays 带来太多好处，特别是与像 Numba 这样的本地加速器相比，Numba 适用于使用和不使用图形处理单元（GPUs）的本地任务的向量化和并行化。你可以使用 Numba 与或不使用 Dask，并且我们将看看如何在第十章中进一步加速 Dask 数组使用 Numba。

Dask 数组与其本地对应物一样，要求数据都是相同类型的。这意味着它们不能用于半结构化或混合类型数据（例如字符串和整数）。

## 加载/保存

与 Dask 数据帧一样，加载和写入函数以 `to_` 或 `read_` 作为前缀开始。每种格式都有自己的配置，但一般来说，第一个位置参数是要读取数据的位置。位置可以是文件的通配符路径（例如 *s3://test-bucket/magic/**），文件列表或常规文件位置。

Dask 数组支持读取以下格式：

+   zarr

+   npy 堆栈（仅本地磁盘）

以及读取和写入：

+   hdf5

+   zarr

+   tiledb

+   npy 堆栈（仅本地磁盘）

另外，您可以将 Dask 数组转换为/从 Dask 袋和数据帧（如果类型兼容）。正如您可能已经注意到的那样，Dask 不支持从许多格式读取数组，这为使用袋提供了一个绝佳的机会（在下一节中介绍）。

## 有何缺失

虽然 Dask 数组实现了大量的 ndarray API，但并非完整集合。与 Dask 数据框一样，部分省略是有意的（例如，`sort`，大部分 `linalg` 等，这些操作会很慢），而其他部分则是因为还没有人有时间来实现它们。

## 特殊的 Dask 函数

与分布式数据框一样，Dask 数组的分区性质使得性能略有不同，因此有一些在 `numpy.linalg` 中找不到的独特 Dask 数组函数：

`map_overlap`

您可以将此用于数据的任何窗口视图，例如在 Dask 数据框上的 `map_overlap`。

`map_blocks`

这类似于 Dask 的 DataFrames `map_partitions`，您可以用它来实现尚未在标准 Dask 库中实现的尴尬并行操作，包括 NumPy 中的新元素级函数。

`topk`

这将返回数组的前 k 个元素，而不是完全排序它（后者要显著更昂贵）。¹

`compute_chunk_sizes`

Dask 需要知道块的大小来支持索引；如果一个数组具有未知的块大小，您可以调用此函数。

这些特殊函数在基础常规集合上不存在，因为它们在非并行/非分布式环境中无法提供相同的性能节省。

# Dask Bags

继续与 Python 内部数据结构类比，您可以将 bags 视为稍有不同的列表或集合。 Bags 类似于列表，但没有顺序的概念（因此没有索引操作）。或者，如果您将 bags 视为集合，则它们与集合的区别在于 bags 允许重复。 Dask 的 bags 对它们包含的内容没有太多限制，并且同样具有最小的 API。 实际上，示例 2-6 到 2-9 涵盖了 bags 的核心 API 大部分内容。

###### 提示

对于从 Apache Spark 转来的用户，Dask bags 与 Spark 的 RDDs 最为接近。

## 常见用例

当数据的结构未知或不一致时，bags 是一个很好的选择。 一些常见用例包括：

+   将一堆 `dask.delayed` 调用分组在一起，例如，用于加载混乱或非结构化（或不支持的）数据。

+   “清理”（或为其添加结构）非结构化数据（如 JSON）。

+   在固定范围内并行化一组任务，例如，如果您想调用 API 100 次，但不关心细节。

+   总括来说：如果数据不适合任何其他集合类型，bags 是您的朋友。

我们认为 Dask bags 最常见的用例是加载混乱数据或 Dask 没有内置支持的数据。

## 加载和保存 Dask Bags

Dask Bag 内置了用于文本文件的读取器，使用 `read_text`，以及 Avro 文件的读取器，使用 `read_avro`。同样，您也可以将 Dask Bag 写入文本文件和 Avro 文件，尽管结果必须可序列化。当 Dask 的内置工具无法满足读取数据需求时，通常会使用 Bags，因此接下来的部分将深入讲解如何超越这两种内置格式。

## 使用 Dask Bag 加载混乱数据

通常，在加载混乱数据时的目标是将其转换为结构化格式以便进一步处理，或者至少提取您感兴趣的组件。虽然您的数据格式可能略有不同，但本节将介绍如何加载一些混乱的 JSON 数据，并提取一些相关字段。不用担心——我们会指出不同格式或来源可能需要不同技术的地方。

对于混乱的文本数据（在 JSON 中很常见），您可以通过使用 bags 的 `read_text` 函数节省一些时间。`read_text` 函数默认按行分割记录；然而，许多格式不能通过行来处理。为了获取每个完整文件作为一个整体记录而不是被分割开，您可以将 `linedelimiter` 参数设置为找不到的值。通常 REST API 会返回结果作为一个子组件，因此在 示例 5-1 中，我们加载 [美国食品药品管理局（FDA）召回数据集](https://oreil.ly/3Xmd_) 并将其剥离到我们关心的部分。FDA 召回数据集是 JSON 数据中经常遇到的嵌套数据集的一个精彩现实世界示例，直接在 DataFrame 中处理这类数据集可能会很困难。

##### 示例 5-1\. 预处理 JSON

```py
def make_url(idx):
    page_size = 100
    start = idx * page_size
    u = f"https://api.fda.gov/food/enforcement.json?limit={page_size}&skip={start}"
    return u

urls = list(map(make_url, range(0, 10)))
# Since they are multi-line json we can't use the default \n line delim
raw_json = bag.read_text(urls, linedelimiter="NODELIM")

def clean_records(raw_records):
    import json
    # We don't need the meta field just the results field
    return json.loads(raw_records)["results"]

cleaned_records = raw_json.map(clean_records).flatten()
# And now we can convert it to a DataFrame
df = bag.Bag.to_dataframe(cleaned_records)
```

如果您需要从不受支持的源（如自定义存储系统）或二进制格式（如协议缓冲区或灵活图像传输系统）加载数据，您需要使用较低级别的 API。对于仍存储在像 S3 这样的 FSSPEC 支持的文件系统中的二进制文件，您可以尝试 示例 5-2 中的模式。

##### 示例 5-2\. 从 FSSPEC 支持的文件系统加载 PDF

```py
def discover_files(path: str):
    (fs, fspath) = fsspec.core.url_to_fs(path)
    return (fs, fs.expand_path(fspath, recursive="true"))

def load_file(fs, file):
    """Load (and initially process) the data."""
    from PyPDF2 import PdfReader
    try:
        file_contents = fs.open(file)
        pdf = PdfReader(file_contents)
        return (file, pdf.pages[0].extract_text())
    except Exception as e:
        return (file, e)

def load_data(path: str):
    (fs, files) = discover_files(path)
    bag_filenames = bag.from_sequence(files)
    contents = bag_filenames.map(lambda f: load_file(fs, f))
    return contents
```

如果您没有使用 FSSPEC 支持的文件系统，您仍然可以按照 示例 5-3 中所示的方式加载数据。

##### 示例 5-3\. 使用纯定制函数加载数据

```py
def special_load_function(x):
    ## Do your special loading logic in this function, like reading a database
    return ["Timbit", "Is", "Awesome"][0: x % 4]

partitions = bag.from_sequence(range(20), npartitions=5)
raw_data = partitions.map(special_load_function).flatten()
```

###### 注意

以这种方式加载数据要求每个文件能够适合一个 worker/executor。如果不符合该条件，情况会变得更加复杂。实现可分割的数据读取器超出了本书的范围，但您可以查看 Dask 的内部 IO 库（文本是最简单的）以获取一些灵感。

有时候，对于嵌套的目录结构，创建文件列表可能需要很长时间。在这种情况下，将文件列表并行化是值得的。有许多不同的技术可以并行化文件列表，但为了简单起见，我们展示了在 示例 5-4 中递归并行列出的方式。

##### 示例 5-4\. 并行列出文件（递归）

```py
def parallel_recursive_list(path: str, fs=None) -> List[str]:
    print(f"Listing {path}")
    if fs is None:
        (fs, path) = fsspec.core.url_to_fs(path)
    info = []
    infos = fs.ls(path, detail=True)
    # Above could throw PermissionError, but if we can't list the dir it's
    # probably wrong so let it bubble up
    files = []
    dirs = []
    for i in infos:
        if i["type"] == "directory":
            # You can speed this up by using futures; covered in Chapter 6
            dir_list = dask.delayed(parallel_recursive_list)(i["name"], fs=fs)
            dirs += dir_list
        else:
            files.append(i["name"])
    for sub_files in dask.compute(dirs):
        files.extend(sub_files)
    return files
```

###### 提示

你并不总是需要自己进行目录列表。检查一下是否有元数据存储（例如 Hive 或 Iceberg）可能会有所帮助，它可以提供文件列表，而不需要进行所有这些慢速 API 调用。

这种方法有一些缺点：即所有的文件名都回到一个单一的点——但这很少是个问题。然而，如果你的文件列表甚至只是太大而无法在内存中容纳，你可能会尝试使用递归算法来进行目录发现，然后采用迭代算法来列出文件，保持文件名在袋子里。² 代码会变得稍微复杂一些，如示例 5-5 所示，所以这种最后的方法很少被使用。

##### 示例 5-5\. 并行列出文件而不收集到驱动程序

```py
def parallel_list_directories_recursive(path: str, fs=None) -> List[str]:
    """
 Recursively find all the sub-directories.
 """
    if fs is None:
        (fs, path) = fsspec.core.url_to_fs(path)
    info = []
    # Ideally, we could filter for directories here, but fsspec lacks that (for
    # now)
    infos = fs.ls(path, detail=True)
    # Above could throw PermissionError, but if we can't list the dir, it's
    # probably wrong, so let it bubble up
    dirs = []
    result = []
    for i in infos:
        if i["type"] == "directory":
            # You can speed this up by using futures; covered in Chapter 6
            result.append(i["name"])
            dir_list = dask.delayed(
                parallel_list_directories_recursive)(i["name"], fs=fs)
            dirs += dir_list
    for sub_dirs in dask.compute(dirs):
        result.extend(sub_dirs)
    return result

def list_files(path: str, fs=None) -> List[str]:
    """List files at a given depth with no recursion."""
    if fs is None:
        (fs, path) = fsspec.core.url_to_fs(path)
    info = []
    # Ideally, we could filter for directories here, but fsspec lacks that (for
    # now)
    return map(lambda i: i["name"], filter(
        lambda i: i["type"] == "directory", fs.ls(path, detail=True)))

def parallel_list_large(path: str, npartitions=None, fs=None) -> bag:
    """
 Find all of the files (potentially too large to fit on the head node).
 """
    directories = parallel_list_directories_recursive(path, fs=fs)
    dir_bag = dask.bag.from_sequence(directories, npartitions=npartitions)
    return dir_bag.map(lambda dir: list_files(dir, fs=fs)).flatten()
```

一个完全迭代的 FSSPEC 算法不会比天真的列表快，因为 FSSPEC 不支持仅查询目录。

## 限制

Dask bags 不太适合大多数的缩减或洗牌操作，因为它们的核心 `reduction` 函数将结果缩减到一个分区，需要所有的数据都能适应单台机器。你可以合理地使用纯粹的常量空间的聚合，例如平均值、最小值和最大值。然而，大多数情况下，你会发现自己尝试对数据进行聚合，你应该考虑将你的 bag 转换为 DataFrame，使用 `bag.Bag.to_dataframe`。

###### 提示

所有三种 Dask 数据类型（bag、array 和 DataFrame）都有被转换为其他数据类型的方法。然而，有些转换需要特别注意。例如，当将 Dask DataFrame 转换为 Dask array 时，生成的数组将具有 `NaN`，如果你查看它生成的形状。这是因为 Dask DataFrame 不会跟踪每个 DataFrame 块中的行数。

# 结论

虽然 Dask DataFrames 得到了最多的使用，但 Dask arrays 和 bags 也有它们的用处。你可以使用 Dask arrays 来加速和并行化大型多维数组处理。Dask bags 允许你处理不太适合 DataFrame 的数据，比如 PDF 或多维嵌套数据。这些集合比 Dask DataFrames 得到的关注和积极的开发要少得多，但可能仍然在你的工作流程中有它们的位置。在下一章中，你将看到如何向你的 Dask 程序中添加状态，包括对 Dask 集合的操作。

¹ [`topk`](https://oreil.ly/vUjgv) 提取每个分区的前 k 个元素，然后只需要将 k 个元素从每个分区洗牌出来。

² 迭代算法涉及使用 *while* 或 *for* 这样的结构，而不是对同一函数的递归调用。
