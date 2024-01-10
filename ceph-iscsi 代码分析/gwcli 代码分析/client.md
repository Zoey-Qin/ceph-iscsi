# 1. 引用部分

```python
from gwcli.node import UIGroup, UINode

from gwcli.utils import response_message, APIRequest, get_config

from ceph_iscsi_config.client import CHAP, GWClient
import ceph_iscsi_config.settings as settings
from ceph_iscsi_config.utils import human_size, this_host

from rtslib_fb.utils import normalize_wwn, RTSLibError

# this ignores the warning issued when verify=False is used
from requests.packages import urllib3

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
```





# 2. class Clients(UIGroup)



## 2.1 help_intro

```python
    help_intro = '''
                 The host section shows the clients that have been defined
                 across each of the gateways in the configuration.

                 Clients may be created or deleted, but once defined they can
                 not be renamed.

                 e.g.
                 create iqn.1994-05.com.redhat:rh7-client4
                 '''
```

- 主机会显示已经在配置中的每个网关上的客户端
- 客户端可以被创建或者删除，但是定义好就不能重命名



 ## 2.2 __init__(*self*, *parent*)

```python
def __init__(self, parent):
    	# 调用了 gwcli.node 中 UIGroup 类的 __init__ 方法，初始化 host 键
        UIGroup.__init__(self, 'hosts', parent)

        # lun_map dict is indexed by the rbd name, pointing to a list
        # of clients that have that rbd allocated.
        # lun_map 是一个用 rbd name 作为索引来指向一个已分配这些 rbd 的客户端列表的 dict
        self.lun_map = {}
        self.client_map = {}

        # record the shortcut
        # shortcut 主要是检查快捷方式的设置，更新了程序的偏好设置，并记录日志信息
        # 从 bookmarks 中获取 key 为 hosts 的 value，并赋值给 shortcut，如果没有这个 key，就将 shortcut 设置为 None
        shortcut = self.shell.prefs['bookmarks'].get('hosts', None)
        # 如果 shortcut 为空或者与 self.path 不相等，那么将 self.path 赋值给偏好设置中的 'hosts' 键
        if not shortcut or shortcut is not self.path:
            self.shell.prefs['bookmarks']['hosts'] = self.path
            # 保存修改
            self.shell.prefs.save()
            self.shell.log.debug("Bookmarked %s as %s."
                                 % (self.path, 'hosts'))
		# 调用 gwcli.ulits 中的 get_config 函数，并将 self.config 返回赋值给实例
        self.config = get_config()
```

- `self.shell.log.debug("Bookmarked %s as %s." % (self.path, 'hosts'))`：记录了一个调试级别的日志，内容是将 `self.path` 设置为 `'hosts'` 的书签的信息。这将输出一条调试信息，记录了书签的设置情况



## 2.3 load(self, client_info)

``` python
# 
def load(self, client_info):
        for client_iqn, client_settings in client_info.items():
            Client(self, client_iqn, client_settings)
```





## 2.4 ui_command_create(self, client_iqn)

```python
def ui_command_create(self, client_iqn):
        """
        Clients may be created using the 'create' sub-command. The initial
        definition will be added to each gateway without any authentication
        set. Once a client is created the admin is automatically placed in the
        context of the new client definition for auth and disk configuration
        operations.

        e.g.
        > create <client_iqn>
        """
        
        # 用于记录调试级别的日志，记录执行了什么命令
        self.logger.debug("CMD: ../hosts/ create {}".format(client_iqn))
        cli_seed = {"luns": {}, "auth": {}}

        # is the IQN usable?
        # 检查 client 名称是否符合规范
        try:
            client_iqn, iqn_type = normalize_wwn(['iqn'], client_iqn)
        except RTSLibError:
            
            self.logger.error("IQN name '{}' is not valid for "
                              "iSCSI".format(client_iqn))
            return

        target_iqn = self.parent.name

        # Issue the API call to create the client
        # 发出调用创建客户端的 API
        client_api = ('{}://localhost:{}/api/'
                      'client/{}/{}'.format(self.http_mode,
                                            settings.config.api_port,
                                            target_iqn,
                                            client_iqn))

        self.logger.debug("Client CREATE for {}".format(client_iqn))
        api = APIRequest(client_api)
        api.put()

        if api.response.status_code == 200:
            Client(self, client_iqn, cli_seed)
            self.config = get_config()
            self.logger.debug("- Client '{}' added".format(client_iqn))
            self.logger.info('ok')

        else:
            self.logger.error("Failed: {}".format(response_message(api.response,
                                                                   self.logger)))
            return

        # switch the current directory to the new client for auth or disk
        # definitions as part of the users workflow
        return self.ui_command_cd(client_iqn)
```

- 可以使用 create 命令来创建一个 clients，初始创建时会添加给每个网关并且不会带有任何身份验证信息，创建成功后会自动转入到这个 client 路径下，然后 admin 可以在这个路径下为这个  client 添加身份验证（authuser.name\password）和 disks

  - 这意味着在客户端最初创建时，它将被“注册”到每个网关中以便进行通信，但在创建时并不会设置用于身份验证的任何信息。这种情况下，需要在稍后的步骤中为客户端进行身份验证的配置
  
  ```
          Clients may be created using the 'create' sub-command. The initial
          definition will be added to each gateway without any authentication
          set. Once a client is created the admin is automatically placed in the
          context of the new client definition for auth and disk configuration
          operations.
  
          e.g.
          > create <client_iqn>
  ```
  
  