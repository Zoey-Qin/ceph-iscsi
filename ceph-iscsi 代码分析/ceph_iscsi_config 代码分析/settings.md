# 1. 引入部分

```python
# 尝试导入 ConfigParser 模块，如果导入失败（即抛出 ImportError 异常），则导入 configparser 模块
try:
    from ConfigParser import ConfigParser
except ImportError:
    from configparser import ConfigParser

import hashlib
import json
import rados
import re

from ceph_iscsi_config.gateway_setting import (TGT_SETTINGS, SYS_SETTINGS,
                                               TCMU_SETTINGS,
                                               TCMU_DEV_STATUS_SETTINGS)
```



- 关于导入的异常处理

```python
try:
    from ConfigParser import ConfigParser
except ImportError:
    from configparser import ConfigParser
```

在 Python 2 中，配置文件解析器的模块名为 `ConfigParser`，而在 Python 3 中被重命名为 `configparser`。为了在不同版本的 Python 中保持兼容性，通常可以使用这样的导入方式。

- `try` 块中的代码尝试导入 `ConfigParser` 模块。`from ConfigParser import ConfigParser`的意思是从 `ConfigParser` 模块中导入 `ConfigParser` 类。

- 如果 `ImportError` 异常被抛出（即在当前环境下没有找到 `ConfigParser` 模块），那么 `except` 块中的代码会被执行。在这种情况下，它会尝试导入 `configparser` 模块，通过 `from configparser import ConfigParser` 将模块中的 `ConfigParser` 类导入。

通过这样的异常处理，代码可以在不同版本的 Python 中灵活地选择导入合适的模块，确保代码的兼容性和可移植性

