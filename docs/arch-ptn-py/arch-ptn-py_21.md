# 附录 D：使用 Django 的存储库和工作单元模式

> 原文：[Appendix D: Repository and Unit of Work Patterns with Django](https://www.cosmicpython.com/book/appendix_django.html)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)

假设你想使用 Django 而不是 SQLAlchemy 和 Flask。事情可能会是什么样子？首先要选择安装的位置。我们将其放在与我们的主要分配代码相邻的一个单独的包中：

```py
├── src
│   ├── allocation
│   │   ├── __init__.py
│   │   ├── adapters
│   │   │   ├── __init__.py
...
│   ├── djangoproject
│   │   ├── alloc
│   │   │   ├── __init__.py
│   │   │   ├── apps.py
│   │   │   ├── migrations
│   │   │   │   ├── 0001_initial.py
│   │   │   │   └── __init__.py
│   │   │   ├── models.py
│   │   │   └── views.py
│   │   ├── django_project
│   │   │   ├── __init__.py
│   │   │   ├── settings.py
│   │   │   ├── urls.py
│   │   │   └── wsgi.py
│   │   └── manage.py
│   └── setup.py
└── tests
    ├── conftest.py
    ├── e2e
    │   └── test_api.py
    ├── integration
    │   ├── test_repository.py
...
```

###### 提示

这个附录的代码在 GitHub 的 appendix_django 分支中[GitHub](https://oreil.ly/A-I76)：

```py
git clone https://github.com/cosmicpython/code.git
cd code
git checkout appendix_django
```

# 使用 Django 的存储库模式

我们使用了一个名为[`pytest-django`](https://github.com/pytest-dev/pytest-django)的插件来帮助管理测试数据库。

重写第一个存储库测试是一个最小的改变 - 只是用 Django ORM/QuerySet 语言重写了一些原始 SQL：

*第一个存储库测试适应（`tests/integration/test_repository.py`）*

```py
from djangoproject.alloc import models as django_models

@pytest.mark.django_db
def test_repository_can_save_a_batch():
    batch = model.Batch("batch1", "RUSTY-SOAPDISH", 100, eta=date(2011, 12, 25))

    repo = repository.DjangoRepository()
    repo.add(batch)

    [saved_batch] = django_models.Batch.objects.all()
    assert saved_batch.reference == batch.reference
    assert saved_batch.sku == batch.sku
    assert saved_batch.qty == batch._purchased_quantity
    assert saved_batch.eta == batch.eta
```

第二个测试涉及的内容更多，因为它有分配，但它仍然由熟悉的 Django 代码组成：

*第二个存储库测试更复杂（`tests/integration/test_repository.py`）*

```py
@pytest.mark.django_db
def test_repository_can_retrieve_a_batch_with_allocations():
    sku = "PONY-STATUE"
    d_line = django_models.OrderLine.objects.create(orderid="order1", sku=sku, qty=12)
    d_b1 = django_models.Batch.objects.create(
    reference="batch1", sku=sku, qty=100, eta=None
)
    d_b2 = django_models.Batch.objects.create(
    reference="batch2", sku=sku, qty=100, eta=None
)
    django_models.Allocation.objects.create(line=d_line, batch=d_batch1)

    repo = repository.DjangoRepository()
    retrieved = repo.get("batch1")

    expected = model.Batch("batch1", sku, 100, eta=None)
    assert retrieved == expected  # Batch.__eq__ only compares reference
    assert retrieved.sku == expected.sku
    assert retrieved._purchased_quantity == expected._purchased_quantity
    assert retrieved._allocations == {
        model.OrderLine("order1", sku, 12),
    }
```

这是实际存储库的最终外观：

*一个 Django 存储库（`src/allocation/adapters/repository.py`）*

```py
class DjangoRepository(AbstractRepository):

    def add(self, batch):
        super().add(batch)
        self.update(batch)

    def update(self, batch):
        django_models.Batch.update_from_domain(batch)

    def _get(self, reference):
        return django_models.Batch.objects.filter(
            reference=reference
        ).first().to_domain()

    def list(self):
        return [b.to_domain() for b in django_models.Batch.objects.all()]
```

你可以看到，实现依赖于 Django 模型具有一些自定义方法，用于转换到我们的领域模型和从领域模型转换。¹

## Django ORM 类上的自定义方法，用于转换到/从我们的领域模型

这些自定义方法看起来像这样：

*Django ORM 与领域模型转换的自定义方法（`src/djangoproject/alloc/models.py`）*

```py
from django.db import models
from allocation.domain import model as domain_model


class Batch(models.Model):
    reference = models.CharField(max_length=255)
    sku = models.CharField(max_length=255)
    qty = models.IntegerField()
    eta = models.DateField(blank=True, null=True)

    @staticmethod
    def update_from_domain(batch: domain_model.Batch):
        try:
            b = Batch.objects.get(reference=batch.reference)  #(1)
        except Batch.DoesNotExist:
            b = Batch(reference=batch.reference)  #(1)
        b.sku = batch.sku
        b.qty = batch._purchased_quantity
        b.eta = batch.eta  #(2)
        b.save()
        b.allocation_set.set(
            Allocation.from_domain(l, b)  #(3)
            for l in batch._allocations
        )

    def to_domain(self) -> domain_model.Batch:
        b = domain_model.Batch(
            ref=self.reference, sku=self.sku, qty=self.qty, eta=self.eta
        )
        b._allocations = set(
            a.line.to_domain()
            for a in self.allocation_set.all()
        )
        return b


class OrderLine(models.Model):
    #...
```

①

对于值对象，`objects.get_or_create` 可以工作，但对于实体，你可能需要一个显式的 try-get/except 来处理 upsert。²

②

我们在这里展示了最复杂的例子。如果你决定这样做，请注意会有样板代码！幸运的是，它并不是非常复杂的样板代码。

③

关系也需要一些谨慎的自定义处理。

###### 注意

就像在第二章中一样，我们使用了依赖反转。ORM（Django）依赖于模型，而不是相反。

# 使用 Django 的工作单元模式

测试并没有太大改变：

*适应的 UoW 测试（`tests/integration/test_uow.py`）*

```py
def insert_batch(ref, sku, qty, eta):  #(1)
    django_models.Batch.objects.create(reference=ref, sku=sku, qty=qty, eta=eta)


def get_allocated_batch_ref(orderid, sku):  #(1)
    return django_models.Allocation.objects.get(
        line__orderid=orderid, line__sku=sku
    ).batch.reference


@pytest.mark.django_db(transaction=True)
def test_uow_can_retrieve_a_batch_and_allocate_to_it():
    insert_batch("batch1", "HIPSTER-WORKBENCH", 100, None)

    uow = unit_of_work.DjangoUnitOfWork()
    with uow:
        batch = uow.batches.get(reference="batch1")
        line = model.OrderLine("o1", "HIPSTER-WORKBENCH", 10)
        batch.allocate(line)
        uow.commit()

    batchref = get_allocated_batch_ref("o1", "HIPSTER-WORKBENCH")
    assert batchref == "batch1"


@pytest.mark.django_db(transaction=True)  #(2)
def test_rolls_back_uncommitted_work_by_default():
    ...

@pytest.mark.django_db(transaction=True)  #(2)
def test_rolls_back_on_error():
    ...
```

①

因为在这些测试中有一些小的辅助函数，所以实际的测试主体基本上与 SQLAlchemy 时一样。

②

`pytest-django` `mark.django_db(transaction=True)` 是必须的，用于测试我们的自定义事务/回滚行为。

实现相当简单，尽管我尝试了几次才找到 Django 事务魔法的调用：

*适用于 Django 的 UoW（`src/allocation/service_layer/unit_of_work.py`）*

```py
class DjangoUnitOfWork(AbstractUnitOfWork):
    def __enter__(self):
        self.batches = repository.DjangoRepository()
        transaction.set_autocommit(False)  #(1)
        return super().__enter__()

    def __exit__(self, *args):
        super().__exit__(*args)
        transaction.set_autocommit(True)

    def commit(self):
        for batch in self.batches.seen:  #(3)
            self.batches.update(batch)  #(3)
        transaction.commit()  #(2)

    def rollback(self):
        transaction.rollback()  #(2)
```

①

`set_autocommit(False)` 是告诉 Django 停止自动立即提交每个 ORM 操作，并开始一个事务的最佳方法。

②

然后我们使用显式回滚和提交。

③

一个困难：因为与 SQLAlchemy 不同，我们不是在领域模型实例本身上进行检测，`commit()` 命令需要显式地通过每个存储库触及的所有对象，并手动将它们更新回 ORM。

# API：Django 视图是适配器

Django 的*views.py*文件最终几乎与旧的*flask_app.py*相同，因为我们的架构意味着它是围绕我们的服务层（顺便说一句，服务层没有改变）的一个非常薄的包装器：

*Flask app → Django views (src/djangoproject/alloc/views.py)*

```py
os.environ['DJANGO_SETTINGS_MODULE'] = 'djangoproject.django_project.settings'
django.setup()

@csrf_exempt
def add_batch(request):
    data = json.loads(request.body)
    eta = data['eta']
    if eta is not None:
        eta = datetime.fromisoformat(eta).date()
    services.add_batch(
        data['ref'], data['sku'], data['qty'], eta,
        unit_of_work.DjangoUnitOfWork(),
    )
    return HttpResponse('OK', status=201)

@csrf_exempt
def allocate(request):
    data = json.loads(request.body)
    try:
        batchref = services.allocate(
            data['orderid'],
            data['sku'],
            data['qty'],
            unit_of_work.DjangoUnitOfWork(),
        )
    except (model.OutOfStock, services.InvalidSku) as e:
        return JsonResponse({'message': str(e)}, status=400)

    return JsonResponse({'batchref': batchref}, status=201)
```

# 为什么这么难？

好吧，它可以工作，但感觉比 Flask/SQLAlchemy 要费力。为什么呢？

在低级别上的主要原因是因为 Django 的 ORM 工作方式不同。我们没有 SQLAlchemy 经典映射器的等价物，因此我们的`ActiveRecord`和领域模型不能是同一个对象。相反，我们必须在存储库后面构建一个手动翻译层。这是更多的工作（尽管一旦完成，持续的维护负担不应该太高）。

由于 Django 与数据库紧密耦合，您必须使用诸如`pytest-django`之类的辅助工具，并从代码的第一行开始仔细考虑测试数据库的使用方式，这是我们在纯领域模型开始时不必考虑的。

但在更高的层面上，Django 之所以如此出色的原因是，它的设计围绕着使构建具有最少样板的 CRUD 应用程序变得容易的最佳点。但我们的整本书的主要内容是关于当您的应用程序不再是一个简单的 CRUD 应用程序时该怎么办。

在那一点上，Django 开始妨碍而不是帮助。像 Django 管理这样的东西，在您开始时非常棒，但如果您的应用程序的整个目的是围绕状态更改的工作流程构建一套复杂的规则和建模，那么它们就会变得非常危险。Django 管理绕过了所有这些。

# 如果您已经有 Django，该怎么办

那么，如果您想将本书中的一些模式应用于 Django 应用程序，您应该怎么做呢？我们建议如下：

+   存储库和工作单元模式将需要相当多的工作。它们在短期内将为您带来的主要好处是更快的单元测试，因此请评估在您的情况下是否值得这种好处。在长期内，它们将使您的应用程序与 Django 和数据库解耦，因此，如果您预计希望迁移到其中任何一个，存储库和 UoW 是一个好主意。

+   如果您在*views.py*中看到很多重复，可能会对服务层模式感兴趣。这是一种很好的方式，可以让您将用例与 Web 端点分开思考。

+   您仍然可以在 Django 模型中进行 DDD 和领域建模，尽管它们与数据库紧密耦合；您可能会因迁移而放慢速度，但这不应该是致命的。因此，只要您的应用程序不太复杂，测试速度不太慢，您可能会从*fat models*方法中获益：尽可能将大部分逻辑下推到模型中，并应用实体、值对象和聚合等模式。但是，请参见以下警告。

话虽如此，[Django 社区中的一些人](https://oreil.ly/Nbpjj)发现，*fat models*方法本身也会遇到可扩展性问题，特别是在管理应用程序之间的相互依赖方面。在这些情况下，将业务逻辑或领域层提取出来，放置在视图和表单以及*models.py*之间，可以尽可能地保持其最小化。

# 途中的步骤

假设您正在开发一个 Django 项目，您不确定是否会变得足够复杂，以至于需要我们推荐的模式，但您仍然希望采取一些步骤，以使您的生活在中期更轻松，并且如果以后要迁移到我们的一些模式中，也更轻松。考虑以下内容：

+   我们听到的一个建议是从第一天开始在每个 Django 应用程序中放置一个*logic.py*。这为您提供了一个放置业务逻辑的地方，并使您的表单、视图和模型免于业务逻辑。这可以成为迈向完全解耦的领域模型和/或服务层的垫脚石。

+   业务逻辑层可能开始使用 Django 模型对象，只有在以后才会完全脱离框架，并在纯 Python 数据结构上工作。

+   对于读取方面，您可以通过将读取放入一个地方来获得 CQRS 的一些好处，避免在各个地方散布 ORM 调用。

+   在为读取和领域逻辑分离模块时，值得脱离 Django 应用程序层次结构。业务问题将贯穿其中。

###### 注意

我们想要向 David Seddon 和 Ashia Zawaduk 致敬，因为他们讨论了附录中的一些想法。他们尽力阻止我们在我们没有足够个人经验的话题上说出任何愚蠢的话，但他们可能失败了。

有关处理现有应用程序的更多想法和实际经验，请参阅附录。

DRY-Python 项目的人们构建了一个名为 mappers 的工具，看起来可能有助于最小化这种事情的样板文件。

`@mr-bo-jangles`建议您可以使用`update_or_create`，但这超出了我们的 Django-fu。
