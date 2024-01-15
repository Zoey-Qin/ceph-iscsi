# 1. 引用部分

```python
#!/usr/bin/python

# prep for python 3
# 这是一条 future 语句，用于准备 python2 的代码迁移到 python3
# python2 中 print 是一个语句，但 python3 中 print() 是函数
# 通过引入这条语句，可以在 python2 中使用 print() 函数，从而与 python3 兼容
from __future__ import print_function

# requires python2-requests/python3-requests

# logging 是 python 标准库中的模块，用于记录程序运行时的日志信息
import logging
# 用于与操作系统进行交互，如文件操作、路径操作等
import os
# 用于访问 python 解释器的运行时环境变量和函数
import sys
# 用于解析命令行参数和生成帮助信息
import argparse
# 用于处理与信号相关的操作，例如处理键盘中断信号
import signal

from configshell_fb import ConfigShell, ExecutionError
from gwcli.gateway import ISCSIRoot

import ceph_iscsi_config.settings as settings

__author__ = 'Paul Cuzner'
__version__ = '2.7'
```



# 2. **class** GatewayCLI(ConfigShell)

##  default_prefs 

```python
class GatewayCLI(ConfigShell):

	# 主要是一些默认的配置参数
    default_prefs = {'color_path': 'magenta', 		// 路径显示的颜色
                     'color_command': 'cyan',		// 命令显示的颜色
                     'color_parameter': 'magenta',	// 参数显示的颜色
                     'color_keyword': 'cyan',		// 关键字显示的颜色
                     'completions_in_columns': True,	// 命令自动补全时是否以列的形式 
                     'logfile': None,		// 日志文件的路径，默认不输出到文件（none）
                     'loglevel_console': 'info', // 控制台上输出日志的级别，默认为 info
                     'loglevel_file': 'debug9', // 日志写入文件的级别，默认为 debug9
                     'color_mode': True,	// 默认启用彩色显示
                     'prompt_length': 30,	// 程序提示符的长度
                     'tree_max_depth': 0,	// 树形现实的最大深度，默认为0，表示不限制
                     'tree_status_mode': True, // 默认树形显示时包含状态信息
                     'tree_round_nodes': True, // 默认树形显示的节点为圆形
                     'tree_show_root': True,	// 默认树形显示时展示根节点
                     }
```

# 3. **def**  exception_handler 

``` python
def exception_handler(exception_type, exception, traceback,
                      debug_hook=sys.excepthook):
	# 如果命令行选项 `options.debug` 为True
    # 那么调用默认的 `debug_hook` 处理函数来打印异常信息；
    if options.debug:
        debug_hook(exception_type, exception, traceback)
    # 否则，使用彩色输出打印异常信息。
	else:
        color_red = '\x1b[31;1m'
        color_off = '\x1b[0m'
        print("{}{}: {}{}".format(color_red, exception_type.__name__,
                                  exception, color_off))
```

- 这段代码定义了一个名为 `exception_handler` 的函数，这个函数接受四个参数：`exception_type`、`exception`、`traceback` 和可选的 `debug_hook` 参数（默认值为 `sys.excepthook`）。

  这个函数的主要功能是处理异常，具体包括以下几个方面：

  1. 如果命令行选项 `options.debug` 为True，那么调用默认的 `debug_hook` 处理函数来打印异常信息；否则，使用彩色输出打印异常信息。
  2. 对于彩色输出，定义了 `color_red` 和 `color_off` 两个ANSI转义码，分别表示红色和关闭ANSI颜色输出。
  3. 最终使用print语句输出异常信息，格式化为红色的异常类型名称、异常消息，并使用 `color_off` 关闭颜色输出。

  总体来说，这个函数的作用是根据命令行选项来选择不同的异常处理方式。如果选择了调试模式，就调用默认的 `debug_hook` 来处理异常，否则使用彩色输出将异常信息打印到控制台。



# 4. def get_options

```python
def get_options():

    # Set up the runtime overrides, any of these could be provided
    # by the cfg file(s)
    # 创建一个 argparse.ArgumentParser 对象，并设定程序的名称和描述信息
    # 这里设定了程序名称为 gwcli，描述了程序是用于管理 iscsi gateways
    parser = argparse.ArgumentParser(prog='gwcli',
                                     description='Manage iSCSI gateways')
    # -c 用于指定 pool 和 obj 的名称
    parser.add_argument('-c', '--config-object', type=str,
                        help='pool and object name holding the iSCSI config'
                             ' object (pool/object_name)')
    # -d 用于开启额外的调试信息
    parser.add_argument('-d', '--debug', action='store_true',
                        default=False,
                        help='run with additional debug')
    # -t 设置用于扫描 rbd 的线程数，默认是8
    parser.add_argument('-t', '--threads', type=int,
                        default=8,
                        help='threads used for rbd scanning (default is 8)')
    # -v 显示版本信息
    parser.add_argument('-v', '--version', action='version',
                        version='%(prog)s - {}'.format(__version__))
    # cli_command 指定了一个位置参数，用于接收残余的命令行参数，且将它们作为列表存储
    parser.add_argument('cli_command', type=str, nargs=argparse.REMAINDER)

    # create the opts object
    # 调用 parser.parse_args() 解析命令行参数并将结果存储在 opts 中
    opts = parser.parse_args()

    # establish defaults, just in case they're missing from the config
    # file(s) AND run time call
    # 如果 config_object 选项没有提供，则将其设置为默认值 'rbd/gateway.conf'
    if not opts.config_object:
        opts.config_object = 'rbd/gateway.conf'
	
	# 将 cli_command 中的参数列表合并为一个字符串
    opts.cli_command = ' '.join(opts.cli_command)
	
    # 最后返回了这个 opts 对象
    return opts
```

- 这段代码定义了一个用于解析命令行参数的函数
- 引用部分提到了通过 `import argparse`：导入 Python 的 `argparse` 模块，该模块用于解析命令行参数



# 5.def kbd_handler:pass

 ```python
 def kbd_handler(*args):
     pass
 ```



# 6. def main

```python
def main():
    # 检视当前运行者是否为 root
    is_root = True if os.getuid() == 0 else False
    # 如果不是 root 就提出告警并退出程序
    if not is_root:
        print("CLI only supports root level access")
        sys.exit(-1)
	
    # 创建了一个 GatewayCLI 的实例：shell
    shell = GatewayCLI('~/.gwcli')
    
	# 创建了一个 ISCSIRoot 的实例：root_node
    root_node = ISCSIRoot(shell,
                          scan_threads=options.threads)
	
    # 根据命令行参数 options.cli_command 的存在与否，
    # 设定了 root_node 和 settings.config 的 interactive 属性
    root_node.interactive = False if options.cli_command else True
    settings.config.interactive = False if options.cli_command else True
	
    # Load the config to populate the object model
    # 加载配置以填充对象模型，刷新根节点，如果刷新过程中出现错误，则打印错误信息并退出程序
    root_node.refresh()
    if root_node.error:
        print("Unable to contact the local API endpoint "
              "({})".format(root_node.local_api))
        sys.exit(-1)

    # Account for invocation which includes a command to run i.e. batch mode
    # 判断用户是否输入了命令
    if options.cli_command:
        
		# 如果用户输入了命令，就运行用户传入的 cli 命令，遇到异常时就退出
        try:
            shell.run_cmdline(options.cli_command)
        except Exception as e:
            print(str(e), file=sys.stderr)
            sys.exit(-1)

        sys.exit(0)

    # Main loop - run the interactive shell, until the user exits
    # 一直以交互模式运行一个命令行环境，直到用户退出
    while not shell._exit:
        # 尝试运行用户输入的命令，遇到异常时记录到 log
        try:
            shell.run_interactive()
        except ExecutionError as msg:
            shell.log.error(str(msg))

```

- if 与 while 这两段的联系：

  - 第一段代码是用来处理是否有命令行参数传入的情况。它首先检查是否有命令行参数传入，若存在命令行参数，则尝试运行对应的命令；如果命令执行成功，程序会正常退出并返回0，如果在执行命令的过程中出现异常，程序会打印异常信息到标准错误输出（stderr），然后退出并返回-1。
  - 第二段代码是在没有命令行参数传入的情况下执行的逻辑。它进入一个主循环，不断地运行交互式命令行界面，直到用户选择退出。在这个交互式命令行界面的循环中，它会不断尝试运行用户输入的命令，并在执行过程中捕获可能抛出的 `ExecutionError` 异常，并将异常信息记录在日志中。这保证了用户可以通过交互式界面来与程序进行交互，并处理可能出现的异常情况。

  因此，第一段代码处理了命令行参数存在的情况，执行对应的命令；而第二段代码处理了没有命令行参数时的逻辑，即进入一个主循环，保持程序持续运行并提供交互式操作。这两段代码分别处理了两种不同情况下用户输入的方式和程序的执行逻辑。



