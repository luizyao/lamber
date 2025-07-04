# Lamber
Lamber 收集 [pytest](https://github.com/pytest-dev/pytest) 执行数据（[models](./src/lamber/models.py)），提供消费接口（[hooks](./src/lamber/hooks.py)），并允许用户集成自定义数据。

## 安装
```bash
pip install lamber
```

### 依赖
* python >= 3.10
* pytest >= 8.3.5

## 基于 Sqlite3 的测试报告
测试报告是一种典型的媒介，可以更加直观的展示 Lamber 的功能。Lamber 支持将测试执行数据保存到 Sqlite3 数据库文件中，通过 `--lamber-sqlite-dir` 命令行选项指定数据库文件的**目录**，数据库文件名为 `lamber.db`。

[lamber-report](https://github.com/luizyao/lamber-report) 是基于 Lamber 开发的测试报告，后续讨论将基于此展开。

## 对原生 pytest 脚本的支持
一个典型的 pytest 脚本如下：

```python
import pytest


@pytest.fixture(scope="module")
def module_fixture():
    print("Setup module env")
    yield
    print("Clean module env")


@pytest.fixture(scope="function", autouse=True)
def function_fixture(module_fixture):
    print("Setup function env")
    yield
    print("Clean function env")


def inc(x):
    return x + 1


def test_answer():
    assert inc(3) == 5
```
* 包含 module 和 function 级别的 fixture。

生成的测试报告主体如下：

![test_sample.png](docs/images/test_sample.png)

* 包含 Step Tree、Traceback、Sourcecode、Logs 等信息。
* 其中 pytest fixture 的调用关系会在 Step Tree 中展示。可以通过 `--lamber-ignore-fixture-step` 选项关闭此功能。

## 对原生 pytest 脚本的改造
Lamber 引入 Step 的概念，将测试执行的过程进一步细分，支持**上下文管理器**（`with lamber.Step:`）和**装饰器**（`@lamber.step`）两种方式声明步骤。

改造上述脚本：

```python
import pytest

import lamber


@pytest.fixture(scope="module")
def module_fixture():
    with lamber.Step("Setup module env"):
        print("Setup module env")

    yield

    with lamber.Step("Clean module env"):
        print("Clean module env")


@pytest.fixture(scope="function", autouse=True)
def function_fixture(module_fixture):
    with lamber.Step("Setup function env"):
        print("Setup function env")

    yield

    with lamber.Step("Clean function env"):
        print("Clean function env")


# Step decorator
@lamber.step("Plus one")
def inc(x):
    return x + 1


def test_answer():
    assert inc(3) == 5
```

生成的测试报告主体如下：

![test_sample_with_step.png](docs/images/test_sample_with_step.png)

* 在 Step Tree 上会有进一步细化。

## 支持自定义收集环境信息
Lamber 支持通过 `pytest_lamber_report_environment` 方法集成自定义的环境信息，例如：目标测试版本，备注信息等。

```python
# conftest.py

def pytest_lamber_report_environment() -> dict:
    return {
        "target version": "version 1.0.0",
        "comment": "custom comment",
    }

```

在测试报告中的体现如下：

![report_environment_items](docs/images/report_environment_items.png)

## LICENSE
[MIT LICENSE](./LICENSE.txt)
