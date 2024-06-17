# 附录 B. 补充材料

本附录包含一些与书中案例研究相关的额外代码。您可能会发现这些材料有助于全面理解示例。

# 使用 Asyncio 的餐具示例

“案例研究：机器人和餐具”分析了由多个线程修改全局“厨房”对象实例中餐具记录而引起的竞争条件错误。为了完整起见，这里是我们可能创建的解决方案的异步版本。

我想要强调关于在`asyncio`方法中并发性的 *可观察性* 的特定点，如示例 B-1 所示。

##### 示例 B-1\. 使用 asyncio 进行餐具管理

```py
import asyncio

class CoroBot():  ![1](img/1.png)
  def __init__(self):
    self.cutlery = Cutlery(knives=0, forks=0)
    self.tasks = asyncio.Queue()  ![2](img/2.png)

  async def manage_table(self):
    while True:
      task = await self.tasks.get()  ![3](img/3.png)
      if task == 'prepare table':
        kitchen.give(to=self.cutlery, knives=4, forks=4)
      elif task == 'clear table':
        self.cutlery.give(to=kitchen, knives=4, forks=4)
      elif task == 'shutdown':
        return

from attr import attrs, attrib

@attrs
class Cutlery:
    knives = attrib(default=0)
    forks = attrib(default=0)

    def give(self, to: 'Cutlery', knives=0, forks=0):
        self.change(-knives, -forks)
        to.change(knives, forks)

    def change(self, knives, forks):
            self.knives += knives
            self.forks += forks

kitchen = Cutlery(knives=100, forks=100)
bots = [CoroBot() for i in range(10)]

import sys
for b in bots:
    for i in range(int(sys.argv[1])):
        b.tasks.put_nowait('prepare table')
        b.tasks.put_nowait('clear table')
    b.tasks.put_nowait('shutdown')

print('Kitchen inventory before service:', kitchen)

loop = asyncio.get_event_loop()
tasks = []
for b in bots:
    t = loop.create_task(b.manage_table())
    tasks.append(t)

task_group = asyncio.gather(*tasks)
loop.run_until_complete(task_group)
print('Kitchen inventory after service:', kitchen)
```

![1](img/#co_supplementary_material_CO1-1)

现在我们有了 CoroBot，而不是 ThreadBot。此代码示例仅使用一个线程，并且该线程将管理所有 10 个独立的 CoroBot 对象实例——每个对象实例对应餐厅中的一个桌子。

![2](img/#co_supplementary_material_CO1-2)

我们使用的是支持`asyncio`的队列，而不是`queue.Queue`。

![3](img/#co_supplementary_material_CO1-3)

这是重点：只有在出现`await`关键字的地方，执行才会在不同的 CoroBot 实例之间切换。在函数的其余部分不可能进行上下文切换，这就是为什么在修改厨房餐具库存时不会出现竞争条件的原因。

存在`await`关键字使得上下文切换 *可观察*。这使得在并发应用程序中推理潜在的竞争条件变得更加容易。无论分配了多少任务，该代码版本始终通过测试：

```py
$ python cutlery_test_corobot.py 100000
Kitchen inventory before service: Cutlery(knives=100, forks=100)
Kitchen inventory after service: Cutlery(knives=100, forks=100)

```

这真的一点也不令人印象深刻：这是完全可以预测的结果，因为代码中显然没有竞争条件。这 *恰恰* 是重点所在。

# 新闻网站爬虫的补充材料

“案例研究：新闻爬取”中展示的 *index.html* 文件是运行代码所需的。在示例 B-2 中，我们可以看到如何创建这个文件。

##### 示例 B-2\. web 爬虫案例研究所需的 index.html 文件

```py
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>The News</title>
    <style>
        .wrapper {
            display: grid;
            grid-template-columns: 300px 300px 300px;
            grid-gap: 10px;
            width: 920px;
            margin: 0 auto;
        }

        .box {
            border-radius: 40px;
            padding: 20px;
            border: 1px solid slategray;
        }

        .cnn {
            background-color: #cef;
        }

        .aljazeera {
            background-color: #fea;
        }

        h1 {
            text-align: center;
            font-size: 60pt;
        }

        a {
            color: black;
            text-decoration: none;
        }
        span {
            text-align: center;
            font-size: 15pt;
            color: black;
        }
    </style>
</head>
<body>
<h1>The News</h1>
<div class="wrapper">
    $body
</div>
</body>
</html>
```

这只是一个非常基本的模板，具有基本的样式。

# ZeroMQ 案例研究的补充材料

在“案例研究：应用程序性能监控”中，我提到您需要提供用于显示指标图表的 HTML 文件。这个文件，*charts.html*，在示例 B-3 中提供。您应该从[Smoothie Charts](http://smoothiecharts.org)或其中一个 CDN 获取*smoothie.min.js*的 URL，并将该 URL 用作`src`属性。

##### 示例 B-3\. charts.html

```py
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Server Performance</title>
    <script src="smoothie.min.js"></script>
    <script type="text/javascript">
        function createTimeline() {
            var cpu = {};  ![1](img/1.png)
            var mem = {};

            var chart_props = {
                responsive: true,
                enableDpiScaling: false,
                millisPerPixel:100,
                grid: {
                    millisPerLine: 4000,
                    fillStyle: '#ffffff',
                    strokeStyle: 'rgba(0,0,0,0.08)',
                    verticalSections: 10
                },
                labels:{fillStyle:'#000000',fontSize:18},
                timestampFormatter:SmoothieChart.timeFormatter,
                maxValue: 100,
                minValue: 0
            };

            var cpu_chart = new SmoothieChart(chart_props);  ![2](img/2.png)
            var mem_chart = new SmoothieChart(chart_props);

            function add_timeseries(obj, chart, color) {  ![3](img/3.png)
                obj[color] = new TimeSeries();
                chart.addTimeSeries(obj[color], {
                    strokeStyle: color,
                    lineWidth: 4
                })
            }

            var evtSource = new EventSource("/feed");  ![4](img/4.png)
            evtSource.onmessage = function(e) {
                var obj = JSON.parse(e.data);  ![5](img/5.png)
                if (!(obj.color in cpu)) {
                    add_timeseries(cpu, cpu_chart, obj.color);
                }
                if (!(obj.color in mem)) {
                    add_timeseries(mem, mem_chart, obj.color);
                }
                cpu[obj.color].append(
                    Date.parse(obj.timestamp), obj.cpu);  ![6](img/6.png)
                mem[obj.color].append(
                    Date.parse(obj.timestamp), obj.mem);
            };

            cpu_chart.streamTo(
                document.getElementById("cpu_chart"), 1000
            );
            mem_chart.streamTo(
                document.getElementById("mem_chart"), 1000
            );
        }
    </script>
    <style>
        h1 {
            text-align: center;
            font-family: sans-serif;
        }
    </style>
</head>
<body onload="createTimeline()">
    <h1>CPU (%)</h1>
    <canvas id="cpu_chart" style="width:100%; height:300px">
    </canvas>
    <hr>
    <h1>Memory usage (MB)</h1>
    <canvas id="mem_chart" style="width:100%; height:300px">
    </canvas>
```

![1](img/#co_supplementary_material_CO2-1)

`cpu` 和 `mem` 各自是一个将颜色映射到 `TimeSeries()` 实例的映射。

![2](img/#co_supplementary_material_CO2-2)

为 CPU 创建一个图表实例，并为内存使用创建另一个图表实例。

![3](img/#co_supplementary_material_CO2-3)

我们在`EventSource()`实例的`onmessage`事件中创建一个`TimeSeries()`实例。这意味着任何新数据的到来（例如不同颜色名称）都将为其自动创建一个新的时间序列。`add_timeseries()`函数创建`TimeSeries()`实例并将其添加到给定的图表实例中。

![4](img/#co_supplementary_material_CO2-4)

在*/feed* URL 上创建一个新的`EventSource()`实例。浏览器将连接到我们服务器上的此端点（*metric_server.py*）。请注意，如果连接丢失，浏览器将自动尝试重新连接。虽然经常被忽视，但在许多情况下，服务器发送事件的简单性使其优于 WebSocket。

![5](img/#co_supplementary_material_CO2-5)

每当服务器发送数据时，`onmessage`事件将触发。此处的数据被解析为 JSON。

![6](img/#co_supplementary_material_CO2-6)

请回想一下，`cpu`标识符是颜色到`TimeSeries()`实例的映射。在这里，我们获取该时间序列并向其附加数据。我们还获取时间戳并解析它，以获取图表所需的正确格式。

# asyncpg 案例研究的数据库触发器处理

在“案例研究：缓存失效”中，由于篇幅限制，省略了所需的一个 Python 源文件。该文件在示例 B-4 中展示。

##### 示例 B-4\. triggers.py

```py
# triggers.py
from asyncpg.connection import Connection  ![1](img/1.png)

async def create_notify_trigger(  ![2](img/2.png)
        conn: Connection,
        trigger_name: str = 'table_update_notify',
        channel: str = 'table_change') -> None:
    await conn.execute(
        'CREATE EXTENSION IF NOT EXISTS hstore')  ![3](img/3.png)
    await conn.execute(
            SQL_CREATE_TRIGGER.format(
                trigger_name=trigger_name,
                channel=channel))  ![4](img/4.png)

async def add_table_triggers(  ![5](img/5.png)
        conn: Connection,
        table: str,
        trigger_name: str = 'table_update_notify',
        schema: str = 'public') -> None:
    templates = (SQL_TABLE_INSERT, SQL_TABLE_UPDATE,
                 SQL_TABLE_DELETE)  ![6](img/6.png)
    for template in templates:
        await conn.execute(
            template.format(
                table=table,
                trigger_name=trigger_name,
                schema=schema))  ![7](img/7.png)

SQL_CREATE_TRIGGER = """\ CREATE OR REPLACE FUNCTION {trigger_name}()
 RETURNS trigger AS $$
DECLARE
 id integer; -- or uuid
 data json;
BEGIN
 data = json 'null';
 IF TG_OP = 'INSERT' THEN
 id = NEW.id;
 data = row_to_json(NEW);
 ELSIF TG_OP = 'UPDATE' THEN
 id = NEW.id;
 data = json_build_object(
      'old', row_to_json(OLD),
      'new', row_to_json(NEW),
      'diff', hstore_to_json(hstore(NEW) - hstore(OLD))
 );
 ELSE
 id = OLD.id;
 data = row_to_json(OLD);
 END IF;
 PERFORM
 pg_notify(
      '{channel}',
 json_build_object(
        'table', TG_TABLE_NAME,
        'id', id,
        'type', TG_OP,
        'data', data
 )::text
 );
 RETURN NEW;
END;
$$ LANGUAGE plpgsql;
"""  ![8](img/8.png)

SQL_TABLE_UPDATE = """\ DROP TRIGGER IF EXISTS
  {table}_notify_update ON {schema}.{table};
CREATE TRIGGER {table}_notify_update
 AFTER UPDATE ON {schema}.{table}
 FOR EACH ROW
 EXECUTE PROCEDURE {trigger_name}();
"""  ![9](img/9.png)

SQL_TABLE_INSERT = """\ DROP TRIGGER IF EXISTS
  {table}_notify_insert ON {schema}.{table};
CREATE TRIGGER {table}_notify_insert
 AFTER INSERT ON {schema}.{table}
 FOR EACH ROW
 EXECUTE PROCEDURE {trigger_name}();
"""

SQL_TABLE_DELETE = """\ DROP TRIGGER IF EXISTS
  {table}_notify_delete ON {schema}.{table};
CREATE TRIGGER {table}_notify_delete
 AFTER DELETE ON {schema}.{table}
 FOR EACH ROW
 EXECUTE PROCEDURE {trigger_name}();
"""
```

![1](img/#co_supplementary_material_CO3-1)

尽管此导入仅用于允许在类型注释中使用`Connection`，但这些函数需要`asyncpg`。

![2](img/#co_supplementary_material_CO3-2)

`create_notify_trigger()`协程函数将在数据库中创建触发器函数本身。触发器函数将包含将发送更新的频道名称。函数本身的代码位于`SQL_CREATE_TRIGGER`标识符中，并设置为格式化字符串。

![3](img/#co_supplementary_material_CO3-3)

请回想一下案例研究示例，更新通知包括“diff”部分，显示旧数据与新数据之间的差异。我们使用 PostgreSQL 的`hstore`功能来计算该差异。它提供了接近集合语义的功能。`hstore`扩展默认未启用，因此我们在此启用它。

![4](img/#co_supplementary_material_CO3-4)

模板中将替换所需的触发器名称和通道，然后执行。

![5](img/#co_supplementary_material_CO3-5)

第二个函数，`add_table_triggers()`，将触发器函数连接到插入、更新和删除等表事件上。

![6](img/#co_supplementary_material_CO3-6)

每种方法都有三个格式字符串。

![7](img/#co_supplementary_material_CO3-7)

所需变量被替换进模板，然后执行。

![8](img/#co_supplementary_material_CO3-8)

这个 SQL 代码花费了比预期更长的时间才能准确无误！这个 PostgreSQL 过程被用于处理插入、更新和删除事件；判断使用 `TG_OP` 变量。如果操作是 `INSERT`，那么 `NEW` 将被定义（而 `OLD` 将不会）。对于 `DELETE`，`OLD` 将被定义但 `NEW` 不会。对于 `UPDATE`，两者都被定义，这允许我们计算差异。我们还利用 PostgreSQL 的内置 JSON 支持，使用 `row_to_json()` 和 `hstore_to_json()` 函数：这些意味着我们的回调处理程序将接收到有效的 JSON。

最后，调用 `pg_notify()` 函数实际发送事件。*所有订阅者* 在 `{channel}` 上都将收到通知。

![9](img/#co_supplementary_material_CO3-9)

这是标准的触发器代码：它设置一个触发器，以便在特定事件发生时调用特定过程 `{trigger_name}()`，如 `INSERT` 或 `UPDATE`。

可以围绕从 PostgreSQL 接收到的通知构建许多有用的应用程序。

# Sanic 示例的补充材料：aelapsed 和 aprofiler

Sanic 案例研究（参见 *asyncpg* 案例研究）包括用于打印函数执行时间的实用装饰器。这些显示在 示例 B-5 中。

##### 示例 B-5\. perf.py

```py
# perf.py
import logging
from time import perf_counter
from inspect import iscoroutinefunction

logger = logging.getLogger('perf')

def aelapsed(corofn, caption=''):  ![1](img/1.png)
    async def wrapper(*args, **kwargs):
        t0 = perf_counter()
        result = await corofn(*args, **kwargs)
        delta = (perf_counter() - t0) * 1e3
        logger.info(
            f'{caption} Elapsed: {delta:.2f} ms')
        return result
    return wrapper

def aprofiler(cls, bases, members):  ![2](img/2.png)
    for k, v in members.items():
        if iscoroutinefunction(v):
            members[k] = aelapsed(v, k)
    return type.__new__(type, cls, bases, members)
```

![1](img/#co_supplementary_material_CO4-1)

`aelapsed()` 装饰器将记录执行包装的协程所花费的时间。

![2](img/#co_supplementary_material_CO4-2)

`aprofiler()` 元类将确保类中的每个协程函数都被 `aelapsed()` 装饰器包装。
