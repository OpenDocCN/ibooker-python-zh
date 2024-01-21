# 附录 C：更换基础设施：使用 CSV 做所有事情

> 原文：[Appendix C: Swapping Out the Infrastructure: Do Everything with CSVs](https://www.cosmicpython.com/book/appendix_csvs.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)

本附录旨在简要说明 Repository、UnitOfWork 和 Service Layer 模式的好处。它旨在从第六章中延伸出来。

就在我们完成构建 Flask API 并准备发布时，业务部门来找我们，道歉地说他们还没有准备好使用我们的 API，并询问我们是否可以构建一个仅从几个 CSV 中读取批次和订单并输出第三个 CSV 的东西。

通常这是一种可能会让团队咒骂、唾弃并为他们的回忆做笔记的事情。但我们不会！哦不，我们已经确保我们的基础设施问题与我们的领域模型和服务层很好地解耦。切换到 CSV 将只是简单地编写一些新的`Repository`和`UnitOfWork`类，然后我们就能重用领域层和服务层的*所有*逻辑。

这是一个 E2E 测试，向您展示 CSV 的流入和流出：

*第一个 CSV 测试（`tests/e2e/test_csv.py`）*

```py
def test_cli_app_reads_csvs_with_batches_and_orders_and_outputs_allocations(
        make_csv
):
    sku1, sku2 = random_ref('s1'), random_ref('s2')
    batch1, batch2, batch3 = random_ref('b1'), random_ref('b2'), random_ref('b3')
    order_ref = random_ref('o')
    make_csv('batches.csv', [
        ['ref', 'sku', 'qty', 'eta'],
        [batch1, sku1, 100, ''],
        [batch2, sku2, 100, '2011-01-01'],
        [batch3, sku2, 100, '2011-01-02'],
    ])
    orders_csv = make_csv('orders.csv', [
        ['orderid', 'sku', 'qty'],
        [order_ref, sku1, 3],
        [order_ref, sku2, 12],
    ])

    run_cli_script(orders_csv.parent)

    expected_output_csv = orders_csv.parent / 'allocations.csv'
    with open(expected_output_csv) as f:
        rows = list(csv.reader(f))
    assert rows == [
        ['orderid', 'sku', 'qty', 'batchref'],
        [order_ref, sku1, '3', batch1],
        [order_ref, sku2, '12', batch2],
    ]
```

毫无思考地实现并不考虑存储库和所有那些花哨东西，你可能会从这样的东西开始：

*我们的 CSV 读取器/写入器的第一个版本（src/bin/allocate-from-csv）*

```py
#!/usr/bin/env python
import csv
import sys
from datetime import datetime
from pathlib import Path

from allocation import model

def load_batches(batches_path):
    batches = []
    with batches_path.open() as inf:
        reader = csv.DictReader(inf)
        for row in reader:
            if row['eta']:
                eta = datetime.strptime(row['eta'], '%Y-%m-%d').date()
            else:
                eta = None
            batches.append(model.Batch(
                ref=row['ref'],
                sku=row['sku'],
                qty=int(row['qty']),
                eta=eta
            ))
    return batches

def main(folder):
    batches_path = Path(folder) / 'batches.csv'
    orders_path = Path(folder) / 'orders.csv'
    allocations_path = Path(folder) / 'allocations.csv'

    batches = load_batches(batches_path)

    with orders_path.open() as inf, allocations_path.open('w') as outf:
        reader = csv.DictReader(inf)
        writer = csv.writer(outf)
        writer.writerow(['orderid', 'sku', 'batchref'])
        for row in reader:
            orderid, sku = row['orderid'], row['sku']
            qty = int(row['qty'])
            line = model.OrderLine(orderid, sku, qty)
            batchref = model.allocate(line, batches)
            writer.writerow([line.orderid, line.sku, batchref])

if __name__ == '__main__':
    main(sys.argv[1])
```

看起来还不错！而且我们正在重用我们的领域模型对象和领域服务。

但这不会起作用。现有的分配也需要成为我们永久 CSV 存储的一部分。我们可以编写第二个测试来迫使我们改进事情：

*另一个，带有现有分配（`tests/e2e/test_csv.py`）*

```py
def test_cli_app_also_reads_existing_allocations_and_can_append_to_them(
        make_csv
):
    sku = random_ref('s')
    batch1, batch2 = random_ref('b1'), random_ref('b2')
    old_order, new_order = random_ref('o1'), random_ref('o2')
    make_csv('batches.csv', [
        ['ref', 'sku', 'qty', 'eta'],
        [batch1, sku, 10, '2011-01-01'],
        [batch2, sku, 10, '2011-01-02'],
    ])
    make_csv('allocations.csv', [
        ['orderid', 'sku', 'qty', 'batchref'],
        [old_order, sku, 10, batch1],
    ])
    orders_csv = make_csv('orders.csv', [
        ['orderid', 'sku', 'qty'],
        [new_order, sku, 7],
    ])

    run_cli_script(orders_csv.parent)

    expected_output_csv = orders_csv.parent / 'allocations.csv'
    with open(expected_output_csv) as f:
        rows = list(csv.reader(f))
    assert rows == [
        ['orderid', 'sku', 'qty', 'batchref'],
        [old_order, sku, '10', batch1],
        [new_order, sku, '7', batch2],
    ]
```

我们可以继续对`load_batches`函数进行修改并添加额外的行，以及一种跟踪和保存新分配的方式，但我们已经有了一个可以做到这一点的模型！它叫做我们的 Repository 和 UnitOfWork 模式。

我们需要做的（“我们需要做的”）只是重新实现相同的抽象，但是以 CSV 作为基础，而不是数据库。正如您将看到的，这确实相对简单。

# 为 CSV 实现 Repository 和 UnitOfWork

一个基于 CSV 的存储库可能看起来像这样。它将磁盘上读取 CSV 的所有逻辑抽象出来，包括它必须读取*两个不同的 CSV*（一个用于批次，一个用于分配），并且它给我们提供了熟悉的`.list()` API，这提供了一个领域对象的内存集合的幻觉：

*使用 CSV 作为存储机制的存储库（`src/allocation/service_layer/csv_uow.py`）*

```py
class CsvRepository(repository.AbstractRepository):

    def __init__(self, folder):
        self._batches_path = Path(folder) / 'batches.csv'
        self._allocations_path = Path(folder) / 'allocations.csv'
        self._batches = {}  # type: Dict[str, model.Batch]
        self._load()

    def get(self, reference):
        return self._batches.get(reference)

    def add(self, batch):
        self._batches[batch.reference] = batch

    def _load(self):
        with self._batches_path.open() as f:
            reader = csv.DictReader(f)
            for row in reader:
                ref, sku = row['ref'], row['sku']
                qty = int(row['qty'])
                if row['eta']:
                    eta = datetime.strptime(row['eta'], '%Y-%m-%d').date()
                else:
                    eta = None
                self._batches[ref] = model.Batch(
                    ref=ref, sku=sku, qty=qty, eta=eta
                )
        if self._allocations_path.exists() is False:
            return
        with self._allocations_path.open() as f:
            reader = csv.DictReader(f)
            for row in reader:
                batchref, orderid, sku = row['batchref'], row['orderid'], row['sku']
                qty = int(row['qty'])
                line = model.OrderLine(orderid, sku, qty)
                batch = self._batches[batchref]
                batch._allocations.add(line)

    def list(self):
        return list(self._batches.values())
```

这是一个用于 CSV 的 UoW 会是什么样子的示例：

*用于 CSV 的 UoW：commit = csv.writer（`src/allocation/service_layer/csv_uow.py`）*

```py
class CsvUnitOfWork(unit_of_work.AbstractUnitOfWork):

    def __init__(self, folder):
        self.batches = CsvRepository(folder)

    def commit(self):
        with self.batches._allocations_path.open('w') as f:
            writer = csv.writer(f)
            writer.writerow(['orderid', 'sku', 'qty', 'batchref'])
            for batch in self.batches.list():
                for line in batch._allocations:
                    writer.writerow(
                        [line.orderid, line.sku, line.qty, batch.reference]
                    )

    def rollback(self):
        pass
```

一旦我们做到了，我们用于读取和写入批次和分配到 CSV 的 CLI 应用程序将被简化为它应该有的东西——一些用于读取订单行的代码，以及一些调用我们*现有*服务层的代码：

*用九行代码实现 CSV 的分配（src/bin/allocate-from-csv）*

```py
def main(folder):
    orders_path = Path(folder) / 'orders.csv'
    uow = csv_uow.CsvUnitOfWork(folder)
    with orders_path.open() as f:
        reader = csv.DictReader(f)
        for row in reader:
            orderid, sku = row['orderid'], row['sku']
            qty = int(row['qty'])
            services.allocate(orderid, sku, qty, uow)
```

哒哒！*现在你们是不是印象深刻了*？

Much love,

Bob and Harry
