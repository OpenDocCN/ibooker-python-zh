# 第十五章：文件

# 预览

除了处理 API 请求和传统的内容（如 HTML）之外，Web 服务器还需要处理双向文件传输。非常大的文件可能需要以不占用太多系统内存的*块*的形式传输。您还可以通过 `StaticFiles` 提供对文件目录（以及任意深度的子目录）的访问。

# 多部分支持

要处理大文件，FastAPI 的上传和下载功能需要这些额外的模块：

[Python-Multipart](https://oreil.ly/FUBk7)

`pip install python-multipart`

[aio-files](https://oreil.ly/OZYYR)

`pip install aiofiles`

# 文件上传

FastAPI 的目标是 API 开发，本书中大多数示例都使用了 JSON 请求和响应。但在下一章中，您将看到处理方式不同的表单。本章涵盖了文件，它在某些方面被视为表单处理。

FastAPI 提供了两种文件上传技术：`File()` 和 `UploadFile`。

## File()

`File()` 用作直接文件上传的类型。您的路径函数可以是同步 (`def`) 或异步 (`async def`) 的，但是异步版本更好，因为它在文件上传时不会占用您的 Web 服务器。

FastAPI 将以块的形式拉取文件并将其重新组装到内存中，因此 `File()` 应仅用于相对较小的文件。FastAPI 不假定输入为 JSON，而是将文件编码为表单元素。

让我们编写请求文件并进行测试的代码。您可以选择您的机器上的任何文件进行测试，或者从网站（如 [Fastest Fish](https://oreil.ly/EnlH-)）下载一个文件进行测试。我从那里抓取了一个 1 KB 的文件并将其保存到本地为 *1KB.bin*。

在 示例 15-1 中，将以下行添加到你的 *main.py* 的顶部。

##### 示例 15-1\. 使用 FastAPI 处理小文件上传

```py
from fastapi import File

@app.post("/small")
async def upload_small_file(small_file: bytes = File()) -> str:
    return f"file size: {len(small_file)}"
```

在 Uvicorn 重新启动后，尝试在 示例 15-2 中进行 HTTPie 测试。

##### 示例 15-2\. 使用 HTTPie 上传小文件

```py
$ http -f -b POST http://localhost:8000/small small_file@1KB.bin
"file size: 1000"
```

关于这个测试的一些注意事项：

+   你需要包括 `-f`（或 `--form`），因为文件是以表单形式而不是 JSON 文本上传的。

+   `small_file@1KB.bin`:

    `small_file`

    匹配 FastAPI 路径函数中的变量名 `small_file`，参见 示例 15-1

    `@`

    HTTPie 的快捷方式来制作表单

    `1KB.bin`

    正在上传的文件

示例 15-3 是一个等价的编程测试。

##### 示例 15-3\. 使用 Requests 上传小文件

```py
$ python
>>> import requests
>>> url = "http://localhost:8000/small"
>>> files = {'small_file': open('1KB.bin', 'rb')}
>>> resp = requests.post(url, files=files)
>>> print(resp.json())
file size: 1000
```

## UploadFile

对于大文件，最好使用 `UploadFile`。这将在服务器的磁盘上创建一个 Python `SpooledTemporaryFile` 对象，而不是在内存中。这是一个类似于 Python 文件的对象，支持 `read()`、`write()` 和 `seek()` 方法。示例 15-4 展示了这一点，并且还使用了 `async def` 而不是 `def`，以避免在上传文件片段时阻塞 Web 服务器。

##### 示例 15-4\. 使用 FastAPI 上传大文件（添加到 *main.py*）

```py
from fastapi import UploadFile

@app.post("/big")
async def upload_big_file(big_file: UploadFile) -> str:
    return f"file size: {big_file.size}, name: {big_file.filename}"
```

###### 注意

`File()` 创建了一个 `bytes` 对象并需要括号。`UploadFile` 是一个不同类的对象。

如果 Uvicorn 的起动电机还没磨损，现在是测试时间了。这次，示例 15-5 到 15-6 使用了一个来自 Fastest Fish 的 1 GB 文件 (*1GB.bin*)。

##### 示例 15-5\. 使用 HTTPie 测试大文件上传

```py
$ http -f -b POST http://localhost:8000/big big_file@1GB.bin
"file size: 1000000000, name: 1GB.bin"
```

##### 示例 15-6\. 使用 Requests 测试大文件上传

```py
>>> import requests
>>> url = "http://localhost:8000/big"
>>> files = {'big_file': open('1GB.bin', 'rb')}
>>> resp = requests.post(url, files=files)
>>> print(resp.json())
file size: 1000000000, name: 1GB.bin
```

# 下载文件

不幸的是，重力不会使文件下载更快。相反，我们将使用类似于上传方法的等效方法。

## FileResponse

首先，在 示例 15-7 中是一次性的版本，`FileResponse`。

##### 示例 15-7\. 使用 `FileResponse` 下载小文件（添加到 main.py）

```py
from fastapi.responses import FileResponse

@app.get("/small/{name}")
async def download_small_file(name):
    return FileResponse(name)
```

这里大概有个测试。首先，将文件 *1KB.bin* 放到与 *main.py* 相同的目录下。然后，运行 示例 15-8。

##### 示例 15-8\. 使用 HTTPie 下载小文件

```py
$ http -b http://localhost:8000/small/1KB.bin

-----------------------------------------
| NOTE: binary data not shown in terminal |
-----------------------------------------
```

如果你不信任那个抑制消息，示例 15-9 将输出传递到像 `wc` 这样的实用程序，以确保你收到了 1,000 字节。

##### 示例 15-9\. 使用 HTTPie 下载小文件，并计算字节数

```py
$ http -b http://localhost:8000/small/1KB.bin | wc -c
    1000
```

## StreamingResponse

类似于 `FileUpload`，最好使用 `StreamingResponse` 下载大文件，它会以块返回文件。示例 15-10 展示了这一点，使用了一个 `async def` 路径函数，以避免在不使用 CPU 时阻塞。我暂时跳过错误检查；如果文件 `path` 不存在，`open()` 调用会引发异常。

##### 示例 15-10\. 使用 `StreamingResponse` 返回大文件（添加到 main.py）

```py
from pathlib import path
from typing import Generator
from fastapi.responses import StreamingResponse

def gen_file(path: str) -> Generator:
    with open(file=path, mode="rb") as file:
        yield file.read()

@app.get("/download_big/{name}")
async def download_big_file(name:str):
    gen_expr = gen_file(file_path=path)
    response = StreamingResponse(
        content=gen_expr,
        status_code=200,
    )
    return response
```

`gen_expr` 是由 *generator function* `gen_file()` 返回的 *generator expression*。`StreamingResponse` 使用它作为其可迭代的 `content` 参数，因此可以分块下载文件。

示例 15-11 是配套的测试。（这首先需要将文件 *1GB.bin* 与 *main.py* 放在一起，并且可能需要一点时间。）

##### 示例 15-11\. 使用 HTTPie 下载大文件

```py
$ http -b http://localhost:8000/big/1GB.bin | wc -c
 1000000000
```

# 提供静态文件

传统的 Web 服务器可以像处理普通文件系统一样处理服务器文件。FastAPI 允许您使用 `StaticFiles` 实现这一点。

对于这个示例，让我们创建一个（无聊的）免费文件的目录供用户下载：

+   在与 *main.py* 同级的目录下创建一个名为 *static* 的目录。（这个名字可以随意取；我只是称之为 *static* 是为了帮助记住我为什么创建它。）

+   在其中放置一个名为 *abc.txt* 的文本文件，其文本内容为 `abc :)`。

示例 15-12 将为 */static* 开头的任何 URL （你也可以在这里使用任何文本字符串）提供来自 *static* 目录的文件。

##### 示例 15-12\. 使用 `StaticFiles` 服务于目录中的所有内容（添加到 main.py）

```py
from pathlib import Path
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

# Directory containing main.py:
top = Path(__file__).resolve.parent

app.mount("/static",
    StaticFiles(directory=f"{top}/static", html=True),
    name="free")
```

那个 `top` 计算确保你将 `static` 放在 *main.py* 旁边。`__file__` 变量是此文件（*main.py*）的完整路径名。

示例 15-13 是手动测试 示例 15-12 的一种方式。

##### 示例 15-13\. 获取一个静态文件

```py
$ http -b localhost:8000/static/abc.txt
abc :)
```

关于您传递给`StaticFiles()`的`html=True`参数呢？这使它更像是传统服务器，如果该目录中存在*index.html*文件，则会返回该文件，但您在 URL 中并没有显式要求*index.html*。因此，让我们在*static*目录中创建一个包含内容为`Oh. Hi!`的*index.html*文件，然后使用示例 15-14 进行测试。

##### 示例 15-14\. 从/static 获取一个 index.html 文件

```py
$ http -b localhost:8000/static/
Oh. Hi!
```

您可以拥有任意数量的文件（及其子目录及文件等）。在*static*下创建一个名为*xyz*的子目录，并放入两个文件：

*xyx.txt*

包含文本`xyz :(`。

*index.html*

包含文本`How did you find me?`

我不会在这里包含示例。请自行尝试，希望您能有更多的命名想象力。

# 回顾

本章展示了如何上传和下载文件——无论是小文件，大文件，甚至是巨大文件。另外，您还学会了如何以怀旧（非 API）的 Web 风格从目录中提供*静态文件*。
