# 第六章 分布式训练与 Ray Train

理查德·利奥

在之前的章节中，您已经学习了如何使用 Ray 构建和扩展强化学习应用程序，以及如何为这些应用程序优化超参数。正如我们在第一章中所指出的，Ray 还配备了 Ray Train 库，提供了广泛的机器学习训练集成，并允许它们无缝扩展。

我们将从提供关于为什么可能需要扩展机器学习训练的背景开始。然后我们将介绍您使用 Ray Train 所需了解的一些关键概念。最后，我们将介绍 Ray Train 提供的一些更高级的功能。

一如既往，你可以使用[本章的笔记本](https://github.com/maxpumperla/learning_ray/blob/main/notebooks/ch_06_train.ipynb)进行跟随。

# 分布式模型训练的基础

机器学习通常需要大量的重型计算。根据你正在训练的模型类型，无论是梯度提升树还是神经网络，你可能会面临几个训练机器学习模型的常见问题，这促使你调查分布式训练解决方案：

1.  完成训练所需的时间太长。

1.  数据的大小太大，无法放入一台机器中。

1.  模型本身太大，无法适应单台机器。

对于第一种情况，通过提高数据处理的吞吐量可以加速训练。一些机器学习算法，比如神经网络，可以并行计算部分内容以加快训练¹。

在第二种情况下，你选择的算法可能要求将数据集中的所有可用数据放入内存，但给定的单节点内存可能不足。在这种情况下，你需要将数据分布到多个节点上，并以分布式的方式进行训练。另一方面，有时你的算法可能不需要数据分布，但如果你一开始就使用分布式数据库系统，你仍然希望使用能够利用分布式数据的训练框架。

在第三种情况下，当您的模型无法适应单个机器时，您可能需要将模型分割为多个部分，分布在多台机器上。将模型分割到多台机器上的方法称为模型并行。要遇到这个问题，你首先需要一个足够大以至于无法放入单个机器的模型。通常情况下，像 Google 或 Facebook 这样的大公司倾向于需要模型并行，并依赖内部解决方案来处理分布式训练。

相比之下，前两个问题通常在机器学习从业者旅程的早期阶段就会出现。我们刚刚概述的这些问题的解决方案属于数据并行训练的范畴。你不是将模型分割到多台机器上，而是依赖分布式数据来加速训练。

特别是对于第一个问题，如果您能加速训练过程，希望能够在准确性减少或没有减少的情况下尽可能做到成本效益，为什么不去做呢？如果您有分布式数据，无论是由于算法的必要性还是数据存储方式的原因，您都需要一个训练解决方案来处理它。正如您将看到的，Ray Train 是专为高效的数据并行训练而构建的。

# Ray Train 简介

Ray Train 是一个用于 Ray 上的分布式训练库。它提供了关键工具，用于训练工作流的不同部分，从特征处理到可伸缩训练，再到与 ML 跟踪工具的集成和模型的导出机制。

在典型的 ML 训练流水线中，您将使用 Ray Train 的以下关键组件：

预处理器

Ray Train 提供了几个常见的预处理对象和工具，以将数据集对象处理成可消耗的特征供训练器使用。

训练器

Ray Train 拥有几个训练器类，使得分布式训练成为可能。训练器是围绕第三方训练框架（如 XGBoost）提供的包装类，与核心 Ray actors（用于分布）集成，还有 Tune 和数据集的整合。

模型

每个训练器可以生成一个模型。该模型可以用于服务。

让我们看看如何通过计算第一个 Ray Train 示例来将这些概念付诸实践。

## 创建一个端到端的 Ray Train 示例

在下面的示例中，我们演示了如何使用 Ray Train 加载、处理和训练机器学习模型的能力。

对于此示例，我们将使用 scikit-learn 的 `datasets` 包中的 `load_breast_cancer` 函数来使用一个简单的数据集²。我们首先将数据加载到 Pandas DataFrame 中，然后将其转换为所谓的 Ray `Dataset`。第七章完全专注于 Ray Data 库，我们在此仅用它来说明 Ray Train 的 API。

##### 示例 6-1。

```py
from ray.data import from_pandas
import sklearn.datasets

data_raw = sklearn.datasets.load_breast_cancer(as_frame=True)  ![1](img/1.png)

dataset_df = data_raw["data"]
predict_ds = from_pandas(dataset_df)  ![2](img/2.png)

dataset_df["target"] = data_raw["target"]
dataset = from_pandas(dataset_df)
```

![1](img/#co_distributed_training_with_ray_train_CO1-1)

将乳腺癌数据加载到 Pandas DataFrame 中。

![2](img/#co_distributed_training_with_ray_train_CO1-2)

从 DataFrame 创建一个 Ray Dataset。

接下来，让我们指定一个预处理函数。在这种情况下，我们将使用三个关键的预处理器：`Scaler`、`Repartitioner` 和 `Chain` 对象，将前两者链在一起。

##### 示例 6-2。

```py
preprocessor = Chain(  ![1](img/1.png)
    Scaler(["worst radius", "worst area"]),  ![2](img/2.png)
    Repartitioner(num_partitions=2)  ![3](img/3.png)
)
```

![1](img/#co_distributed_training_with_ray_train_CO2-1)

创建一个预处理的 `Chain`。

![2](img/#co_distributed_training_with_ray_train_CO2-2)

缩放两个特定的数据列。

![3](img/#co_distributed_training_with_ray_train_CO2-3)

将数据分区为两个分区。

我们进行分布式训练的入口是训练器对象。针对不同的框架有特定的训练器，并且每个训练器都配置了一些特定于框架的参数。

举个例子，让我们看看`XGBoostTrainer`类，它实现了[XGBoost](https://xgboost.readthedocs.io/en/stable/)的分布式训练。

##### 示例 6-3。

```py
trainer = XGBoostTrainer(
    scaling_config={  ![1](img/1.png)
        "num_actors": 2,
        "gpus_per_actor": 0,
        "cpus_per_actor": 2,
    },
    label="target",  ![2](img/2.png)
    params={  ![3](img/3.png)
        "tree_method": "approx",
        "objective": "binary:logistic",
        "eval_metric": ["logloss", "error"],
    },
)

result = trainer.fit(dataset=dataset, preprocessor=preprocessor)  ![4](img/4.png)

print(result)
```

![1](img/#co_distributed_training_with_ray_train_CO3-1)

指定缩放配置。

![2](img/#co_distributed_training_with_ray_train_CO3-2)

设置标签列。

![3](img/#co_distributed_training_with_ray_train_CO3-3)

指定 XGBoost 特定参数。

![4](img/#co_distributed_training_with_ray_train_CO3-4)

通过调用`fit`来训练模型。

# Ray Train 中的预处理器

预处理器是处理数据预处理的核心类。每个预处理器具有以下 API：

| transform | 用于处理并应用数据集的处理转换。 |
| --- | --- |
| fit | 用于计算和存储有关预处理器数据集的聚合状态。返回 self 以便进行链式调用。 |
| fit_transform | 用于执行需要聚合状态的转换的语法糖。可能在特定预处理器的实现级别进行优化。 |
| transform_batch | 用于对批处理进行相同的预测转换。 |

目前，Ray Train 提供以下编码器

| FunctionTransformer | 自定义转换器 |
| --- | --- |
| Pipeline | 顺序预处理 |
| StandardScaler | 标准化 |
| MinMaxScaler | 标准化 |
| OrdinalEncoder | 编码分类特征 |
| OneHotEncoder | 编码分类特征 |
| SimpleImputer | 缺失值填充 |
| LabelEncoder | 标签编码 |

您经常希望确保在训练时间和服务时间可以使用相同的数据预处理操作。

## 预处理器的使用

您可以通过将它们传递给训练器来使用这些预处理器。Ray Train 将负责以分布式方式应用预处理器到数据集中。

##### 示例 6-4。

```py
result = trainer.fit(dataset=dataset, preprocessor=preprocessor)
```

## 预处理器的序列化

现在，一些预处理操作符，如独热编码器，在训练时很容易运行并传递到服务。然而，像标准化这样的其他操作符有点棘手，因为您不希望在服务时间执行大数据处理（例如查找特定列的平均值）。

Ray Train 的一个很好的特点是它们是可序列化的。这使得您可以通过序列化这些运算符来轻松实现从训练到服务的一致性。

##### 示例 6-5。

```py
pickle.dumps(prep)  ![1](img/1.png)
```

![1](img/#co_distributed_training_with_ray_train_CO4-1)

我们可以序列化和保存预处理器。

# Ray Train 中的训练器

训练器是特定框架的类，以分布式方式运行模型训练。所有训练器共享一个公共接口：

| fit(self) | 使用给定数据集来拟合这个训练器。 |
| --- | --- |
| get_checkpoints(self) | 返回最近模型检查点列表。 |
| as_trainable(self) | 获取此对象的 Tune 可训练类包装器。 |

Ray Train 支持各种不同的训练器，涵盖多种框架，包括 XGBoost、LightGBM、Pytorch、HuggingFace、Tensorflow、Horovod、Scikit-learn、RLlib 等。

接下来，我们将深入了解两类特定的训练器：梯度提升树框架训练器和深度学习框架训练器。

## 梯度提升树的分布式训练

Ray Train 提供了 LightGBM 和 XGBoost 的训练器。

XGBoost 是一个经过优化的分布式梯度提升库，旨在高效、灵活和可移植。它在梯度提升框架下实现了机器学习算法。XGBoost 提供了一种并行树提升（也称为 GBDT、GBM），以快速和准确的方式解决许多数据科学问题。

LightGBM 是基于树的学习算法的梯度提升框架。与 XGBoost 相比，它是一个相对较新的框架，但在学术界和生产环境中迅速流行起来。

通过利用 Ray Train 的 XGBoost 或 LightGBM 训练器，您可以在多台机器上使用大型数据集训练 XGBoost Booster。

## 深度学习的分布式训练

Ray Train 提供了深度学习训练器，例如支持 Tensorflow、Horovod 和 Pytorch 等框架。

与梯度提升树训练器不同，这些深度学习框架通常会给用户更多控制权。例如，Pytorch 提供了一组原语，用户可以用来构建他们的训练循环。

因此，深度学习训练器 API 允许用户传入一个训练函数，并提供回调函数，用于报告指标和检查点。让我们看一个例子，Pytorch 训练脚本。

下面，我们构建一个标准的训练函数：

##### 示例 6-6。

```py
import torch
import torch.nn as nn

num_samples = 20
input_size = 10
layer_size = 15
output_size = 5

class NeuralNetwork(nn.Module):
    def __init__(self):
        super(NeuralNetwork, self).__init__()
        self.layer1 = nn.Linear(input_size, layer_size)
        self.relu = nn.ReLU()
        self.layer2 = nn.Linear(layer_size, output_size)

    def forward(self, input_data):
        return self.layer2(self.relu(self.layer1(input_data)))

input = torch.randn(num_samples, input_size)  ![1](img/1.png)
labels = torch.randn(num_samples, output_size)

def train_func():  ![2](img/2.png)
    num_epochs = 3
    model = NeuralNetwork()
    loss_fn = nn.MSELoss()
    optimizer = optim.SGD(model.parameters(), lr=0.1)
    for epoch in range(num_epochs):
        output = model(input)
        loss = loss_fn(output, labels)
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
```

![1](img/#co_distributed_training_with_ray_train_CO5-1)

在此示例中，我们使用随机生成的数据集。

![2](img/#co_distributed_training_with_ray_train_CO5-2)

定义一个训练函数。

我们构建了一个 Pytorch 训练脚本，其中创建了一个小型神经网络，并使用均方误差（`MSELoss`）目标来优化模型。这里模型的输入是随机噪声，但您可以想象它是从一个 Torch 数据集中生成的。

现在，Ray Train 将为您处理两个关键事项。

1.  建立一个协调进程间通信的后端。

1.  多个并行进程的实例化。

所以，简而言之，您只需要对您的代码做一行更改：

##### 示例 6-7。

```py
import ray.train.torch

def train_func_distributed():
    num_epochs = 3
    model = NeuralNetwork()
    model = train.torch.prepare_model(model)  ![1](img/1.png)
    loss_fn = nn.MSELoss()
    optimizer = optim.SGD(model.parameters(), lr=0.1)
    for epoch in range(num_epochs):
        output = model(input)
        loss = loss_fn(output, labels)
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
```

![1](img/#co_distributed_training_with_ray_train_CO6-1)

为分布式训练准备模型。

然后，您可以将其接入 Ray Train：

##### 示例 6-8。

```py
from ray.train import Trainer

trainer = Trainer(backend="torch", num_workers=4, use_gpu=False)  ![1](img/1.png)
trainer.start()
results = trainer.run(train_func_distributed)
trainer.shutdown()
```

![1](img/#co_distributed_training_with_ray_train_CO7-1)

创建一个训练器。对于 GPU 训练，将 `use_gpu` 设置为 True。

这段代码可以在单台机器或分布式集群上运行。

## 使用 Ray Train 训练器扩展训练规模

Ray Train 的总体理念是，用户不需要考虑如何并行化他们的代码。

使用 Ray Train Trainers，您可以指定一个`scaling_config`，允许您扩展您的训练而无需编写分布式逻辑。`scaling_config`允许您声明性地指定 Trainer 使用的*计算资源*。

特别是，您可以通过提供工作者数量和每个工作者应使用的设备类型来指定 Trainer 应该使用的并行度：

##### 示例 6-9。

```py
scaling_config = {"num_workers": 10, "use_gpu": True}

trainer = ray.train.integrations.XGBoostTrainer(
    scaling_config=scaling_config,
    # ...
)
```

注意，缩放配置参数取决于 Trainer 类型。

这种规范的好处在于，您不需要考虑底层硬件。特别是，您可以指定使用数百个工作者，Ray Train 将自动利用 Ray 集群中的所有节点：

##### 示例 6-10。

```py
# Connect to a large Ray cluster
ray.init(address="auto")

scaling_config = {"num_workers": 200, "use_gpu": True}

trainer = ray.train.integrations.XGBoostTrainer(
    scaling_config=scaling_config,
    # ...
)
```

## 将数据连接到分布式训练

Ray Train 提供了训练大数据集的实用工具。

与同样的理念相同，用户不需要考虑如何并行化他们的代码，您只需简单地将大数据集“连接”到 Ray Train，而不必考虑如何摄入和将数据提供给不同的并行工作者。

在这里，我们从随机数据创建了一个数据集。但是，您可以使用其他数据 API 来读取大量数据（使用`read_parquet`，从[Parquet 格式](https://parquet.apache.org/)读取数据）。

##### 示例 6-11。

```py
from typing import Dict
import torch
import torch.nn as nn

import ray
import ray.train as train
from ray.train import Trainer

def get_datasets(a=5, b=10, size=1000, split=0.8):

    def get_dataset(a, b, size):
        items = [i / size for i in range(size)]
        dataset = ray.data.from_items([{"x": x, "y": a * x + b} for x in items])
        return dataset

    dataset = get_dataset(a, b, size)
    split_index = int(dataset.count() * split)
    train_dataset, validation_dataset = dataset.random_shuffle().split_at_indices(
        [split_index]
    )
    train_dataset_pipeline = train_dataset.repeat().random_shuffle_each_window()
    validation_dataset_pipeline = validation_dataset.repeat()
    datasets = {
        "train": train_dataset_pipeline,
        "validation": validation_dataset_pipeline,
    }
    return datasets
```

然后可以指定一个训练函数来访问这些数据点。注意，这里使用了特定的`get_dataset_shard`函数。在幕后，Ray Train 会自动对提供的数据集进行分片，以便各个工作节点可以同时训练数据的不同子集。这样可以避免在同一个 epoch 内对重复数据进行训练。`get_dataset_shard`函数将从数据源传递数据的子集给每个单独的并行训练工作者。

接下来，我们对每个 shard 调用`iter_epochs`和`to_torch`。`iter_epochs`将生成一个迭代器。此迭代器将生成 Dataset 对象，每个对象将拥有整个 epoch 的 1 个 shard（命名为`train_dataset_iterator`和`validation_dataset_iterator`）。

`to_torch`将把 Dataset 对象转换为 Pytorch 迭代器。还有一个等价的`to_tf`函数，将其转换为 Tensorflow Data 迭代器。

当 epoch 结束时，Pytorch 迭代器会引发`StopIteration`，然后`train_dataset_iterator`将再次查询新的 shard 和新的 epoch。

##### 示例 6-12。

```py
def train_func(config):
    batch_size = config["batch_size"]
    # hidden_size = config["hidden_size"]
    # lr = config.get("lr", 1e-2)
    epochs = config.get("epochs", 3)

    train_dataset_pipeline_shard = train.get_dataset_shard("train")
    validation_dataset_pipeline_shard = train.get_dataset_shard("validation")
    train_dataset_iterator = train_dataset_pipeline_shard.iter_epochs()
    validation_dataset_iterator = validation_dataset_pipeline_shard.iter_epochs()

    for _ in range(epochs):
        train_dataset = next(train_dataset_iterator)
        validation_dataset = next(validation_dataset_iterator)
        train_torch_dataset = train_dataset.to_torch(
            label_column="y",
            feature_columns=["x"],
            label_column_dtype=torch.float,
            feature_column_dtypes=torch.float,
            batch_size=batch_size,
        )
        validation_torch_dataset = validation_dataset.to_torch(
            label_column="y",
            feature_columns=["x"],
            label_column_dtype=torch.float,
            feature_column_dtypes=torch.float,
            batch_size=batch_size,
        )
        # ... training

    return results
```

您可以通过以下方式使用 Trainer 将所有内容整合起来：

##### 示例 6-13。

```py
num_workers = 4
use_gpu = False

datasets = get_datasets()
trainer = Trainer("torch", num_workers=num_workers, use_gpu=use_gpu)
config = {"lr": 1e-2, "hidden_size": 1, "batch_size": 4, "epochs": 3}
trainer.start()
results = trainer.run(
   train_func,
   config,
   dataset=datasets,
   callbacks=[JsonLoggerCallback(), TBXLoggerCallback()],
)
trainer.shutdown()
print(results)
```

# Ray Train 特性

## 检查点

Ray Train 将生成**模型检查点**以检查训练的中间状态。这些模型检查点提供了经过训练的模型和适配的预处理器，可用于下游应用，如服务和推断。

##### 示例 6-14。

```py
result = trainer.fit()
model: ray.train.Model = result.checkpoint.load_model()
```

模型检查点的目标是抽象出模型和预处理器的实际物理表示。因此，您应该能够从云存储位置生成检查点，并将其转换为内存表示或磁盘表示，反之亦然。

##### 示例 6-15\.

```py
chkpt = Checkpoint.from_directory(dir)
chkpt.to_bytes() -> bytes
```

## 回调函数

您可能希望将您的训练代码与您喜爱的实验管理框架插入。Ray Train 提供了一个接口来获取中间结果和回调函数来处理或记录您的中间结果（传递给 `train.report(...)` 的值）。

Ray Train 包含了流行跟踪框架的内置回调，或者您可以通过 TrainingCallback 接口实现自己的回调函数。可用的回调函数包括：

+   Json 日志记录

+   Tensorboard 日志记录

+   MLflow 日志记录

+   Torch Profiler

##### 示例 6-16\.

```py
# Run the training function, logging all the intermediate results
# to MLflow and Tensorboard.

result = trainer.run(
    train_func,
    callbacks=[
        MLflowLoggerCallback(experiment_name="train_experiment"),
        TBXLoggerCallback(),
    ],
)
```

## 与 Ray Tune 集成

Ray Train 与 Ray Tune 集成，使您可以仅用几行代码进行超参数优化。Tune 将根据每个超参数配置创建一个试验。在每个试验中，将初始化一个新的 Trainer 并使用其生成的配置运行训练函数。

##### 示例 6-17\.

```py
from ray import tune

fail_after_finished = True
prep_v1 = preprocessor
prep_v2 = preprocessor

param_space = {
    "scaling_config": {
        "num_actors": tune.grid_search([2, 4]),
        "cpus_per_actor": 2,
        "gpus_per_actor": 0,
    },
    "preprocessor": tune.grid_search([prep_v1, prep_v2]),
    # "datasets": {
    #     "train_dataset": tune.grid_search([dataset_v1, dataset_v2]),
    # },
    "params": {
        "objective": "binary:logistic",
        "tree_method": "approx",
        "eval_metric": ["logloss", "error"],
        "eta": tune.loguniform(1e-4, 1e-1),
        "subsample": tune.uniform(0.5, 1.0),
        "max_depth": tune.randint(1, 9),
    },
}
if fail_after_finished > 0:
    callbacks = [StopperCallback(fail_after_finished=fail_after_finished)]
else:
    callbacks = None
tuner = tune.Tuner(
    XGBoostTrainer(
        run_config={"max_actor_restarts": 1},
        scaling_config=None,
        resume_from_checkpoint=None,
        label="target",
    ),
    run_config={},
    param_space=param_space,
    name="tuner_resume",
    callbacks=callbacks,
)
results = tuner.fit(datasets={"train_dataset": dataset})
print(results.results)
```

与其他分布式超参数调整解决方案相比，Ray Tune 和 Ray Train 具有一些独特的特性：

+   能够将数据集和预处理器指定为参数

+   容错性

+   能够在训练时调整工作人数。

## 导出模型

在使用 Ray Train 训练模型后，您可能希望将其导出到 Ray Serve 或模型注册表。

为此，您可以使用 load_model API 获取模型：

##### 示例 6-18\.

```py
result = trainer.fit(dataset=dataset, preprocessor=preprocessor)
print(result)

this_checkpoint = result.checkpoint
this_model = this_checkpoint.load_model()
predicted = this_model.predict(predict_ds)
print(predicted.to_pandas())
```

## 一些注意事项

特别是，请记住，标准神经网络训练通过在数据集的不同数据批次（通常称为小批量梯度下降）之间进行迭代。

为了加快速度，您可以并行计算每个小批量更新的梯度。这意味着批次应该在多台机器上分割。

如果保持批处理大小不变，则随着工作人数增加，系统利用率和效率会降低。

为了补偿，实践者通常增加每批数据的数量。

因此，通过数据的单次遍历时间（一个 epoch）应该理想地减少，因为总批次数量减少。

¹ 这特别适用于神经网络中的梯度计算。

² Ray 能处理比那更大的数据集。在 第七章 中，我们将更详细地看一下 Ray 数据库，了解如何处理大数据集。
