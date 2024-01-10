# 1. import 部分

```python
import requests
from requests import Response
import sys
import re
import os
import subprocess


from rtslib_fb.utils import normalize_wwn, RTSLibError

from ceph_iscsi_config.client import GWClient
import ceph_iscsi_config.settings as settings
from ceph_iscsi_config.utils import (resolve_ip_addresses, CephiSCSIError,
                                     this_host)
```

