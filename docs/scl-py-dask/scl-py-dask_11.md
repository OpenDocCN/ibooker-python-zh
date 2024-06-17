# 第十一章：用 Dask 进行机器学习

现在你已经了解了 Dask 的许多不同数据类型、计算模式、部署选项和库，我们准备开始机器学习。你会很快发现，使用 Dask 进行机器学习非常直观，因为它与许多其他流行的机器学习库运行在相同的 Python 环境中。Dask 的内置数据类型和分布式调度器大部分完成了繁重的工作，使得编写代码成为用户的愉快体验。¹

本章将主要使用 Dask-ML 库，这是 Dask 开源项目中得到强力支持的机器学习库，但我们也会突出显示其他库，如 XGBoost 和 scikit-learn。Dask-ML 库旨在在集群和本地运行。² Dask-ML 通过扩展许多常见的机器学习库提供了熟悉的接口。机器学习与我们迄今讨论的许多任务有所不同，因为它需要框架（这里是 Dask-ML）更密切地协调工作。在本章中，我们将展示如何在自己的程序中使用它，并提供一些技巧。

由于机器学习是如此广泛和多样化的学科，我们只能涵盖 Dask-ML 有用的一些情况。本章将讨论一些常见的工作模式，例如探索性数据分析、随机拆分、特征化、回归和深度学习推断，从实践者的角度来看，逐步了解 Dask。如果您没有看到您特定的库或用例被涵盖，仍然可能通过 Dask 加速，您应该查看[Dask-ML 的 API 指南](https://oreil.ly/eJGHU)。然而，机器学习并非 Dask 的主要关注点，因此您可能需要使用其他工具，如 Ray。

# 并行化机器学习

许多机器学习工作负载在两个维度上面临扩展挑战：模型大小和数据大小。训练具有大特征或组件的模型（例如许多深度学习模型）通常会变得计算受限，其中训练、预测和评估模型变得缓慢和难以管理。另一方面，许多机器学习模型，甚至看似简单的模型如回归模型，通常会在处理大量训练数据集时超出单台机器的限制，从而在扩展挑战中变得内存受限。

在内存受限的工作负载上，我们已经涵盖的 Dask 的高级集合（如 Dask 数组、DataFrame 和 bag）与 Dask-ML 库结合，提供了本地扩展能力。对于计算受限的工作负载，Dask 通过 Dask-ML 和 Dask-joblib 等集成实现了训练的并行化。在使用 scikit-learn 时，Dask 可以管理集群范围的工作分配，使用 Dask-joblib。你可能会注意到，每个工作流需要不同的库，这是因为每个机器学习工具都使用自己的并行化方法，而 Dask 扩展了这些方法。

您可以将 Dask 与许多流行的机器学习库一起使用，包括 scikit-learn 和 XGBoost。您可能已经熟悉您喜爱的机器学习库内部的单机并行处理。Dask 将这些单机框架（如 Dask-joblib）扩展到通过网络连接的多台机器上。

# 使用 Dask-ML 的时机

Dask 在具有有限分布式可变状态（如大型模型权重）的并行任务中表现突出。Dask 通常用于对机器学习模型进行推断/预测，这比训练要简单。另一方面，训练模型通常需要更多的工作器间通信，例如模型权重更新和重复循环，有时每个训练周期的计算量可能不同。您可以在这两种用例中都使用 Dask，但是对于训练的采用和工具支持并不如推断广泛。

Dask 与常见的数据准备工具（包括 pandas、NumPy、PyTorch 和 TensorFlow）的集成使得构建推断管道变得更加容易。在像 Spark 这样的基于 JVM 的工具中，使用这些库会增加更多的开销。

Dask 的另一个很好的用例是在训练之前进行特征工程和绘制大型数据集。Dask 的预处理函数通常使用与 scikit-learn 相同的签名和方式，同时跨多台机器分布工作。类似地，对于绘图和可视化，Dask 能够生成一个大型数据集的美观图表，超出了 matplotlib/seaborn 的常规限制。

对于更复杂的机器学习和深度学习工作，一些用户选择单独生成 PyTorch 或 TensorFlow 模型，然后使用生成的模型进行基于 Dask 的推断工作负载。这样可以使 Dask 端的工作负载具有令人尴尬的并行性。另外，一些用户选择使用延迟模式将训练数据写入 Dask DataFrame，然后将其馈送到 Keras 或 Torch 中。请注意，这样做需要一定的中等工作量。

如前面章节所述，Dask 项目仍处于早期阶段，其中一些库仍在进行中并带有免责声明。我们格外小心地验证了 Dask-ML 库中使用的大部分数值方法，以确保逻辑和数学是正确的，并按预期工作。然而，一些依赖库有警告称其尚未准备好供主流使用，特别是涉及 GPU 感知工作负载和大规模分布式工作负载时。我们预计随着社区的增长和用户贡献反馈，这些问题将得到解决。

# 开始使用 Dask-ML 和 XGBoost

Dask-ML 是 Dask 的官方支持的机器学习库。在这里，我们将介绍 Dask-ML API 提供的功能，以及它如何将 Dask、pandas 和 scikit-learn 集成到其功能中，以及 Dask 和其 scikit-learn 等效物之间的一些区别。此外，我们还将详细讲解几个 XGBoost 梯度提升集成案例。我们将主要使用之前用过的纽约市黄色出租车数据进行示例演示。您可以直接从[纽约市网站](https://oreil.ly/lbU5V)访问数据集。

## 特征工程

就像任何优秀的数据科学工作流一样，我们从清理、应用缩放器和转换开始。Dask-ML 提供了大部分来自 scikit-learn 的预处理 API 的即插即用替代品，包括 `StandardScaler`、`PolynomialFeatures`、`MinMaxScaler` 等。

您可以将多个列传递给转换器，每个列都将被归一化，最终生成一个延迟的 Dask DataFrame，您应该调用 `compute` 方法来执行计算。

在 示例 11-1 中，我们对行程距离（单位为英里）和总金额（单位为美元）进行了缩放，生成它们各自的缩放变量。这是我们在第四章中进行的探索性数据分析的延续。

##### 示例 11-1\. 使用 `StandardScaler` 对 Dask DataFrame 进行预处理

```py
from dask_ml.preprocessing import StandardScaler
import dask.array as da
import numpy as np

df = dd.read_parquet(url)
trip_dist_df = df[["trip_distance", "total_amount"]]
scaler = StandardScaler()

scaler.fit(trip_dist_df)
trip_dist_df_scaled = scaler.transform(trip_dist_df)
trip_dist_df_scaled.head()
```

对于分类变量，在 Dask-ML 中虽然有 `OneHotEncoder`，但其效率和一对一替代程度都不及其 scikit-learn 的等效物。因此，我们建议使用 `Categorizer` 对分类 dtype 进行编码。³

示例 11-2 展示了如何对特定列进行分类，同时保留现有的 DataFrame。我们选择 `payment_type`，它最初被编码为整数，但实际上是一个四类别分类变量。我们调用 Dask-ML 的 `Categorizer`，同时使用 pandas 的 `CategoricalDtype` 给出类型提示。虽然 Dask 具有类型推断能力（例如，它可以自动推断类型），但在程序中明确指定类型总是更好的做法。

##### 示例 11-2\. 使用 Dask-ML 对 Dask DataFrame 进行分类变量预处理

```py
from dask_ml.preprocessing import Categorizer
from pandas.api.types import CategoricalDtype

payment_type_amt_df = df[["payment_type", "total_amount"]]

cat = Categorizer(categories={"payment_type": CategoricalDtype([1, 2, 3, 4])})
categorized_df = cat.fit_transform(payment_type_amt_df)
categorized_df.dtypes
payment_type_amt_df.head()
```

或者，您可以选择使用 Dask DataFrame 的内置分类器。虽然 pandas 对 Object 和 String 作为分类数据类型是宽容的，但除非首先将这些列读取为分类变量，否则 Dask 将拒绝这些列。有两种方法可以做到这一点：在读取数据时将列声明为分类，使用`dtype={col: categorical}`，或在调用`get_dummies`之前转换，使用`df​.catego⁠rize(“col1”)`。这里的理由是 Dask 是惰性评估的，不能在没有看到完整唯一值列表的情况下创建列的虚拟变量。调用`.categorize()`很方便，并允许动态处理额外的类别，但请记住，它确实需要先扫描整个列以获取类别，然后再扫描转换列。因此，如果您已经知道类别并且它们不会改变，您应该直接调用`DummyEncoder`。

Example 11-3 一次对多列进行分类。在调用`execute`之前，没有任何实质性的东西被实现，因此你可以一次链式地连接许多这样的预处理步骤。

##### Example 11-3\. 使用 Dask DataFrame 内置的分类变量作为预处理

```py
train = train.categorize("VendorID")
train = train.categorize("passenger_count")
train = train.categorize("store_and_fwd_flag")

test = test.categorize("VendorID")
test = test.categorize("passenger_count")
test = test.categorize("store_and_fwd_flag")
```

`DummyEncoder`是 Dask-ML 中类似于 scikit-learn 的`OneHotEncoder`，它将变量转换为 uint8，即一个 8 位无符号整数，更节省内存。

再次，有一个 Dask DataFrame 函数可以给您类似的结果。Example 11-4 在分类列上演示了这一点，而 Example 11-5 则预处理了日期时间。日期时间可能会带来一些棘手的问题。在这种情况下，Python 原生反序列化日期时间。请记住，始终在转换日期时间之前检查并应用必要的转换。

##### Example 11-4\. 使用 Dask DataFrame 内置的虚拟变量作为哑变量的预处理

```py
from dask_ml.preprocessing import DummyEncoder

dummy = DummyEncoder()
dummified_df = dummy.fit_transform(categorized_df)
dummified_df.dtypes
dummified_df.head()
```

##### Example 11-5\. 使用 Dask DataFrame 内置的虚拟变量作为日期时间的预处理

```py
train['Hour'] = train['tpep_pickup_datetime'].dt.hour
test['Hour'] = test['tpep_pickup_datetime'].dt.hour

train['dayofweek'] = train['tpep_pickup_datetime'].dt.dayofweek
test['dayofweek'] = test['tpep_pickup_datetime'].dt.dayofweek

train = train.categorize("dayofweek")
test = test.categorize("dayofweek")

dom_train = dd.get_dummies(
    train,
    columns=['dayofweek'],
    prefix='dom',
    prefix_sep='_')
dom_test = dd.get_dummies(
    test,
    columns=['dayofweek'],
    prefix='dom',
    prefix_sep='_')

hour_train = dd.get_dummies(
    train,
    columns=['dayofweek'],
    prefix='h',
    prefix_sep='_')
hour_test = dd.get_dummies(
    test,
    columns=['dayofweek'],
    prefix='h',
    prefix_sep='_')

dow_train = dd.get_dummies(
    train,
    columns=['dayofweek'],
    prefix='dow',
    prefix_sep='_')
dow_test = dd.get_dummies(
    test,
    columns=['dayofweek'],
    prefix='dow',
    prefix_sep='_')
```

Dask-ML 的`train_test_split`方法比 Dask DataFrames 版本更灵活。两者都支持分区感知，我们使用它们而不是 scikit-learn 的等价物。scikit-learn 的`train_test_split`可以在此处调用，但它不具备分区感知性，可能导致大量数据在工作节点之间移动，而 Dask 的实现则会在每个分区上分割训练集和测试集，避免洗牌（参见 Example 11-6）。

##### Example 11-6\. Dask DataFrame 伪随机分割

```py
from dask_ml.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    df['trip_distance'], df['total_amount'])
```

每个分区块的随机分割的副作用是，整个 DataFrame 的随机行为不能保证是均匀的。如果你怀疑某些分区可能存在偏差，你应该计算、重新分配，然后洗牌分割。

## 模型选择和训练

很多 scikit-learn 的模型选择相关功能，包括交叉验证、超参数搜索、聚类、回归、填充以及评分方法，都已经作为一个可替换组件移植到 Dask 中。有几个显著的改进使得这些功能比简单的并行计算架构更高效，这些改进利用了 Dask 的任务图形视图。

大多数基于回归的模型已经为 Dask 实现，并且可以作为 scikit-learn 的替代品使用。⁴ 许多 scikit-learn 用户熟悉使用 `.reshape()` 对 pandas 进行操作，需要将 pandas DataFrame 转换为二维数组以便 scikit-learn 使用。对于一些 Dask-ML 的函数，你仍然需要调用 `ddf.to_dask_array()` 将 DataFrame 转换为数组以进行训练。最近，一些 Dask-ML 已经改进，可以直接在 Dask DataFrames 上工作，但并非所有库都支持。

示例 11-7 展示了使用 Dask-ML 进行直观的多变量线性回归。假设您希望在两个预测列和一个输出列上构建回归模型。您将应用 `.to_array()` 将数据类型转换为 Dask 数组，然后将它们传递给 Dask-ML 的 `LinearRegression` 实现。请注意，我们需要明确指定分块大小，这是因为 Dask-ML 在线性模型的底层实现中并未完全能够从前一步骤中推断出分块大小。我们还特意使用 scikit-learn 的评分库，而不是 Dask-ML。我们注意到 Dask-ML 在处理分块大小时存在实施问题。⁵ 幸运的是，在这一点上，这个计算是一个简化步骤，它可以在没有任何特定于 Dask 的逻辑的情况下工作。⁶

##### 示例 11-7\. 使用 Dask-ML 进行线性回归

```py
from dask_ml.linear_model import LinearRegression
from dask_ml.model_selection import train_test_split

regr_df = df[['trip_distance', 'total_amount']].dropna()
regr_X = regr_df[['trip_distance']]
regr_y = regr_df[['total_amount']]

X_train, X_test, y_train, y_test = train_test_split(
    regr_X, regr_y)

X_train = X_train.to_dask_array(lengths=[100]).compute()
X_test = X_test.to_dask_array(lengths=[100]).compute()
y_train = y_train.to_dask_array(lengths=[100]).compute()
y_test = y_test.to_dask_array(lengths=[100]).compute()

reg = LinearRegression()
reg.fit(X_train, y_train)
y_pred = reg.predict(X_test)

r2_score(y_test, y_pred)
```

请注意，scikit-learn 和 Dask-ML 模型的函数参数是相同的，但是目前不支持的一些功能。例如，`LogisticRegression` 在 Dask-ML 中是可用的，但是不支持多类别求解器，这意味着 Dask-ML 中尚未实现多类别求解器的确切等效。因此，如果您想要使用 multinomial loss solver newton-cg 或 newton-cholesky，可能不会起作用。对于大多数 `LogisticRegression` 的用途，缺省的 liblinear 求解器会起作用。在实践中，此限制仅适用于更为专业和高级的用例。

对于超参数搜索，Dask-ML 具有与 scikit-learn 类似的 `GridSearchCV` 用于参数值的详尽搜索，以及 `RandomizedSearchCV` 用于从列表中随机尝试超参数。如果数据和结果模型不需要太多的缩放，可以直接运行这些功能，与 scikit-learn 的变体类似。

交叉验证和超参数调优通常是一个昂贵的过程，即使在较小的数据集上也是如此，任何运行过 scikit-learn 交叉验证的人都会证明。Dask 用户通常处理足够大的数据集，使用详尽搜索算法是不可行的。作为替代方案，Dask-ML 实现了几种额外的自适应算法和基于超带的方法，这些方法通过稳健的数学基础更快地接近调整参数。⁷ `HyperbandSearchCV` 方法的作者要求引用使用。⁸

## 当没有 Dask-ML 等效时。

如果在 scikit-learn 或其他数据科学库中存在但不在 Dask-ML 中存在的函数，则可以编写所需代码的分布式版本。毕竟，Dask-ML 可以被视为 scikit-learn 的便利包装器。

Example 11-8 使用 scikit-learn 的学习函数 `SGDRegressor` 和 `LinearRegression`，并使用 `dask.delayed` 将延迟功能包装在该方法周围。您可以对您想要并行化的任何代码片段执行此操作。

##### Example 11-8\. 使用 Dask-ML 进行线性回归

```py
from sklearn.linear_model import LinearRegression as ScikitLinearRegression
from sklearn.linear_model import SGDRegressor as ScikitSGDRegressor

estimators = [ScikitLinearRegression(), ScikitSGDRegressor()]
run_tasks = [dask.delayed(estimator.fit)(X_train, y_train)
             for estimator in estimators]
run_tasks
```

## 使用 Dask 的 joblib。

或者，您可以使用 scikit-learn 与 joblib（参见 Example 11-9），这是一个可以将任何 Python 函数作为流水线步骤在单台机器上计算的包。Joblib 在许多不依赖于彼此的并行计算中表现良好。在这种情况下，拥有单台机器上的数百个核心将非常有帮助。虽然典型的笔记本电脑没有数百个核心，但使用它所拥有的四个核心仍然是有益的。使用 Dask 版本的 joblib，您可以使用来自多台机器的核心。这对于在单台机器上计算受限的 ML 工作负载是有效的。

##### Example 11-9\. 使用 joblib 进行计算并行化

```py
from dask.distributed import Client
from joblib import parallel_backend

client = Client('127.0.0.1:8786')

X, y = load_my_data()
net = get_that_net()

gs = GridSearchCV(
    net,
    param_grid={'lr': [0.01, 0.03]},
    scoring='accuracy',
)

XGBClassifier()

with parallel_backend('dask'):
    gs.fit(X, y)
print(gs.cv_results_)
```

## 使用 Dask 的 XGBoost。

XGBoost 是一个流行的 Python 梯度增强库，用于并行树增强。众所周知的梯度增强方法包括自举聚合（bagging）。各种梯度增强方法已在大型强子对撞机的高能物理数据分析中使用，用于训练深度神经网络以确认发现希格斯玻色子。梯度增强方法目前在地质或气候研究等科学领域中被广泛使用。考虑到其重要性，我们发现在 Dask-ML 上的 XGBoost 是一个良好实现的库，可以为用户准备就绪。

Dask-ML 具有内置支持以与 Dask 数组和 DataFrame 一起使用 XGBoost。通过在 Dask-ML 中使用 XGBClassifiers，您将在分布式模式下设置 XGBoost，它可以与您的 Dask 集群一起使用。在这种情况下，XGBoost 的主进程位于 Dask 调度程序中，XGBoost 的工作进程将位于 Dask 的工作进程上。数据分布使用 Dask DataFrame 处理，拆分为 pandas DataFrame，并在同一台机器上的 Dask 工作进程和 XGBoost 工作进程之间通信。

XGBoost 使用 `DMatrix`（数据矩阵）作为其标准数据格式。XGBoost 有内置的 Dask 兼容的 `DMatrix`，可以接受 Dask 数组和 Dask DataFrame。一旦设置了 Dask 环境，梯度提升器的使用就像您期望的那样。像往常一样指定学习率、线程和目标函数。示例 11-10 使用 Dask CUDA 集群，并运行标准的梯度提升器训练。

##### 示例 11-10\. 使用 Dask-ML 进行梯度提升树

```py
import xgboost as xgb
from dask_cuda import LocalCUDACluster
from dask.distributed import Client

n_workers = 4
cluster = LocalCUDACluster(n_workers)
client = Client(cluster)

dtrain = xgb.dask.DaskDMatrix(client, X_train, y_train)

booster = xgb.dask.train(
    client,
    {"booster": "gbtree", "verbosity": 2, "nthread": 4, "eta": 0.01, gamma=0,
     "max_depth": 5, "tree_method": "auto", "objective": "reg:squarederror"},
    dtrain,
    num_boost_round=4,
    evals=[(dtrain, "train")])
```

在 示例 11-11 中，我们进行了简单的训练并绘制了特征重要性图。注意，当我们定义 `DMatrix` 时，我们明确指定了标签，标签名称来自于 Dask DataFrame 到 `DMatrix`。

##### 示例 11-11\. 使用 XGBoost 库的 Dask-ML

```py
import xgboost as xgb

dtrain = xgb.DMatrix(X_train, label=y_train, feature_names=X_train.columns)
dvalid = xgb.DMatrix(X_test, label=y_test, feature_names=X_test.columns)
watchlist = [(dtrain, 'train'), (dvalid, 'valid')]
xgb_pars = {
    'min_child_weight': 1,
    'eta': 0.5,
    'colsample_bytree': 0.9,
    'max_depth': 6,
    'subsample': 0.9,
    'lambda': 1.,
    'nthread': -1,
    'booster': 'gbtree',
    'silent': 1,
    'eval_metric': 'rmse',
    'objective': 'reg:linear'}
model = xgb.train(xgb_pars, dtrain, 10, watchlist, early_stopping_rounds=2,
                  maximize=False, verbose_eval=1)
print('Modeling RMSLE %.5f' % model.best_score)

xgb.plot_importance(model, max_num_features=28, height=0.7)

pred = model.predict(dtest)
pred = np.exp(pred) - 1
```

将之前的示例结合起来，您现在可以编写一个函数，该函数可以适配模型、提供早停参数，并使用 Dask 进行 XGBoost 预测（见 示例 11-12）。这些将在您的主客户端代码中调用。

##### 示例 11-12\. 使用 Dask XGBoost 库进行梯度提升树训练和推断

```py
import xgboost as xgb
from dask_cuda import LocalCUDACluster
from dask.distributed import Client

n_workers = 4
cluster = LocalCUDACluster(n_workers)
client = Client(cluster)

def fit_model(client, X, y, X_valid, y_valid,
              early_stopping_rounds=5) -> xgb.Booster:
    Xy_valid = dxgb.DaskDMatrix(client, X_valid, y_valid)
    # train the model
    booster = xgb.dask.train(
        client,
        {"booster": "gbtree", "verbosity": 2, "nthread": 4, "eta": 0.01, gamma=0,
         "max_depth": 5, "tree_method": "gpu_hist", "objective": "reg:squarederror"},
        dtrain,
        num_boost_round=500,
        early_stopping_rounds=early_stopping_rounds,
        evals=[(dtrain, "train")])["booster"]
    return booster

def predict(client, model, X):
    predictions = xgb.predict(client, model, X)
    assert isinstance(predictions, dd.Series)
    return predictions
```

# 在 Dask-SQL 中使用 ML 模型

另一个较新的添加是 Dask-SQL 库，它提供了一个便捷的包装器，用于简化 ML 模型训练工作负载。 示例 11-13 加载与之前相同的 NYC 黄色出租车数据作为 Dask DataFrame，然后将视图注册到 Dask-SQL 上下文中。

##### 示例 11-13\. 将数据集注册到 Dask-SQL 中

```py
import dask.dataframe as dd
import dask.datasets
from dask_sql import Context

# read dataset
taxi_df = dd.read_csv('./data/taxi_train_subset.csv')
taxi_test = dd.read_csv('./data/taxi_test.csv')

# create a context to register tables
c = Context()
c.create_table("taxi_test", taxi_test)
c.create_table("taxicab", taxi_df)
```

Dask-SQL 实现了类似 BigQuery ML 的 ML SQL 语言，使您能够简单地定义模型，将训练数据定义为 SQL select 语句，然后在不同的 select 语句上运行推断。

您可以使用我们讨论过的大多数 ML 模型定义模型，并在后台运行 scikit-learn ML 模型。在 示例 11-14 中，我们使用 Dask-SQL 训练了之前训练过的 `LinearRegression` 模型。我们首先定义模型，告诉它使用 scikit-learn 的 `LinearRegression` 和目标列。然后，我们使用必要的列传递训练数据。您可以使用 `DESCRIBE` 语句检查训练的模型；然后您可以在 `FROM PREDICT` 语句中看到模型如何在另一个 SQL 定义的数据集上运行推断。

##### 示例 11-14\. 在 Dask-SQL 上定义、训练和预测线性回归模型

```py
import dask.dataframe as dd
import dask.datasets
from dask_sql import Context

c = Context()
# define model
c.sql(
    """
CREATE MODEL fare_linreg_model WITH (
 model_class = 'LinearRegression',
 wrap_predict = True,
 target_column = 'fare_amount'
) AS (
 SELECT passenger_count, fare_amount
 FROM taxicab
 LIMIT 1000
)
"""
)

# describe model
c.sql(
    """
DESCRIBE MODEL fare_linreg_model
 """
).compute()

# run inference
c.sql(
    """
SELECT
 *
FROM PREDICT(MODEL fare_linreg_model,
 SELECT * FROM taxi_test
)
 """
).compute()
```

同样地，如 示例 11-15 所示，您可以使用 Dask-ML 库运行分类模型，类似于我们之前讨论过的 XGBoost 模型。

##### 示例 11-15\. 在 Dask-SQL 上使用 XGBoost 定义、训练和预测分类器

```py
import dask.dataframe as dd
import dask.datasets
from dask_sql import Context

c = Context()
# define model
c.sql(
    """
CREATE MODEL classify_faretype WITH (
 model_class = 'XGBClassifier',
 target_column = 'fare_type'
) AS (
 SELECT airport_surcharge, passenger_count, fare_type
 FROM taxicab
 LIMIT 1000
)
"""
)

# describe model
c.sql(
    """
DESCRIBE MODEL classify_faretype
 """
).compute()

# run inference
c.sql(
    """
SELECT
 *
FROM PREDICT(MODEL classify_faretype,
 SELECT airport_surcharge, passenger_count, FROM taxi_test
)
 """
).compute()
```

# 推断和部署

无论您选择使用哪些库来训练和验证您的模型（可以使用一些 Dask-ML 库，或完全不使用 Dask 训练），在使用 Dask 进行模型推断部署时，需要考虑以下一些事项。

## 手动分发数据和模型

当将数据和预训练模型加载到 Dask 工作节点时，`dask.delayed` 是主要工具（参见 示例 11-16）。在分发数据时，您应选择使用 Dask 的集合：数组和 DataFrame。正如您从 第四章 中记得的那样，每个 Dask DataFrame 由一个 pandas DataFrame 组成。这非常有用，因为您可以编写一个方法，该方法接受每个较小的 DataFrame，并返回计算输出。还可以使用 Dask DataFrame 的 `map_partitions` 函数为每个分区提供自定义函数和任务。

如果您正在读取大型数据集，请记得使用延迟表示法，以延迟实体化并避免过早读取。

###### 小贴士

`map_partitions` 是一种逐行操作，旨在适合序列化代码并封送到工作节点。您可以定义一个处理推理的自定义类来调用，但需要调用静态方法，而不是依赖实例的方法。我们在 第四章 进一步讨论了这一点。

##### 示例 11-16\. 在 Dask 工作节点上加载大文件

```py
from skimage.io import imread
from skimage.io.collection import alphanumeric_key
from dask import delayed
import dask.array as da
import os

root, dirs, filenames = os.walk(dataset_dir)
# sample first file
imread(filenames[0])

@dask.delayed
def lazy_reader(file):
    return imread(file)

# we have a bunch of delayed readers attached to the files
lazy_arrays = [lazy_reader(file) for file in filenames]

# read individual files from reader into a dask array
# particularly useful if each image is a large file like DICOM radiographs
# mammography dicom tends to be extremely large
dask_arrays = [
    da.from_delayed(delayed_reader, shape=(4608, 5200,), dtype=np.float32)
    for delayed_reader in lazy_arrays
]
```

## 使用 Dask 进行大规模推理

当使用 Dask 进行规模推理时，您会将训练好的模型分发到每个工作节点，然后将 Dask 集合（DataFrame 或数组）分发到这些分区，以便一次处理集合的一部分，从而并行化工作流程。这种策略在简单的推理部署中效果良好。我们将讨论其中一种实现方式：手动定义工作流程，使用 `map_partitions`，然后用 PyTorch 或 Keras/TensorFlow 模型包装现有函数。对于基于 PyTorch 的模型，您可以使用 Skorch 将模型包装起来，从而使其能够与 Dask-ML API 一起使用。对于 TensorFlow 模型，您可以使用 SciKeras 创建一个与 scikit-learn 兼容的模型，这样就可以用于 Dask-ML。对于 PyTorch，SaturnCloud 的 dask-pytorch-ddp 库目前是最广泛使用的。至于 Keras 和 TensorFlow，请注意，虽然可以做到，但 TensorFlow 不喜欢一些线程被移动到其他工作节点。

部署推理最通用的方式是使用 Dask DataFrame 的 `map_partitions`（参见 示例 11-17）。您可以使用自定义推理函数，在每行上运行该函数，数据映射到每个工作节点的分区。

##### 示例 11-17\. 使用 Dask DataFrame 进行分布式推理

```py
import dask.dataframe as dd
import dask.bag as db

def rowwise_operation(row, arg *):
    # row-wise compute
    return result

def partition_operation(df):
    # partition wise logic
    result = df[col1].apply(rowwise_operation)
    return result

ddf = dd.read_csv(“metadata_of_files”)
results = ddf.map_partitions(partition_operation)
results.compute()

# An alternate way, but note the .apply() here becomes a pandas apply, not
# Dask .apply(), and you must define axis = 1
ddf.map_partitions(
    lambda partition: partition.apply(
        lambda row: rowwise_operation(row), axis=1), meta=(
            'ddf', object))
```

Dask 提供比其他可扩展库更多的灵活性，特别是在并行行为方面。在前面的示例中，我们定义了一个逐行工作的函数，然后将该函数提供给分区逻辑，每个分区在整个 DataFrame 上运行。我们可以将其作为样板来定义更精细的批处理函数（见示例 11-18）。请记住，在逐行函数中定义的行为应该没有副作用，即，应避免突变函数的输入，这是 Dask 分布式延迟计算的一般最佳实践。此外，正如前面示例中的注释所述，如果在分区式 lambda 内执行`.apply()`，这会调用 pandas 的`.apply()`。在 Pandas 中，`.apply()`默认为`axis = 0`，如果你想要其他方式，应记得指定`axis = 1`。

##### 示例 11-18\. 使用 Dask DataFrame 进行分布式推断

```py
def handle_batch(batch, conn, nlp_model):
    # run_inference_here.
    conn.commit()

def handle_partition(df):
    worker = get_worker()
    conn = connect_to_db()
    try:
        nlp_model = worker.roberta_model
    except BaseException:
        nlp_model = load_model()
        worker.nlp_model = nlp_model
    result, batch = [], []
    for _, row in part.iterrows():
        if len(batch) % batch_size == 0 and len(batch) > 0:
            batch_results = handle_batch(batch, conn, nlp_model)
            result.append(batch_results)
            batch = []
        batch.append((row.doc_id, row.sent_id, row.utterance))
    if len(batch) > 0:
        batch_results = handle_batch(batch, conn, nlp_model)
        result.append(batch_results)
    conn.close()
    return result

ddf = dd.read_csv("metadata.csv”)
results = ddf.map_partitions(handle_partition)
results.compute()
```

# 结论

在本章中，您已经学习了如何使用 Dask 的构建模块来编写数据科学和 ML 工作流程，将核心 Dask 库与您可能熟悉的其他 ML 库结合起来，以实现您所需的任务。您还学习了如何使用 Dask 来扩展计算和内存密集型 ML 工作负载。

Dask-ML 几乎提供了与 scikit-learn 功能相当的库，通常使用 Dask 带来的任务和数据并行意识调用 scikit-learn。Dask-ML 由社区积极开发，并将进一步增加用例和示例。查阅 Dask 文档以获取最新更新。

此外，您已经学会了如何通过使用 joblib 进行计算密集型工作负载的并行化 ML 训练方法，并使用批处理操作处理数据密集型工作负载，以便自己编写任何定制实现。

最后，你已经学习了 Dask-SQL 的用例及其 SQL ML 语句，在模型创建、超参数调整和推断中提供高级抽象。

由于 ML 可能需要大量计算和内存，因此在正确配置的集群上部署您的 ML 工作并密切监视进度和输出非常重要。我们将在下一章中介绍部署、分析和故障排除。

¹ 如果你认为编写数据工程代码是“有趣”的人。

² 这对于非批量推断尤为重要，可以很大程度上提高使用相同代码的便利性。

³ 由于性能原因，在撰写本文时，Dask 的`OneHotEncoder`调用了 pandas 的`get_dummies`方法，这比 scikit-learn 的`OneHotEncoder`实现较慢。另一方面，`Categorizer`使用了 Dask DataFrame 的聚合方法，以高效地扫描类别。

⁴ Dask-ML 中的大多数线性模型使用了为 Dask 实现的广义线性模型库的基本实现。我们已经验证了代码的数学正确性，但这个库的作者尚未认可其在主流应用中的使用。

⁵ Dask-ML 版本 2023.3.24；部分广义线性模型依赖于 dask-glm 0.1.0。

⁶ 因为这是一个简单的归约操作，我们不需要保留之前步骤中的分块。

⁷ Dask-ML 的官方文档提供了有关实现的自适应和近似交叉验证方法以及使用案例的更多信息。

⁸ 他们在文档中指出，如果使用此方法，应引用以下论文：S. Sievert, T. Augspurger, 和 M. Rocklin, “Better and Faster Hyperparameter Optimization with Dask,” *Proceedings of the 18th Python in Science Conference* (2019), *[`doi.org/10.25080/Majora-7ddc1dd1-011`](https://doi.org/10.25080/Majora-7ddc1dd1-011)*.
