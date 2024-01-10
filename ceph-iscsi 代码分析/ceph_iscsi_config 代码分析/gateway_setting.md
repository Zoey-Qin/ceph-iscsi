# 1. convert_str_to_bool

- 作用：将 true/false/yes/no/1/0 这样的字符串转为 bool



```python
def convert_str_to_bool(value):
    """
    Convert true/false/yes/no/1/0 to boolean
    """
	# 检查传入的 value 是否已是 bool，如果是就直接返回
    if isinstance(value, bool):
        return value
	
    # 将 value 转为小写，以确保取消大小写敏感的影响
    value = str(value).lower()
    
    # 将 str 转为 bool
    if value in ['1', 'true', 'yes']:
        return True
    elif value in ['0', 'false', 'no']:
        return False
    # 如果输入的 value 不是 true/false/yes/no/1/0 之间的值，就会直接返回 ValueError 的异常
    raise ValueError(value)
```



- raise：

  `raise` 是 Python 中用于引发异常的关键字。当程序出现错误或不符合预期的情况时，可以使用 `raise` 来主动引发异常，从而中断程序执行并抛出相应的异常对象。

  在上面提到的代码中，`raise ValueError(value)` 语句是用来引发一个 `ValueError` 异常，表示在函数中出现了不允许的数值。当函数执行到这一行时，会立即结束，并且代码执行的控制流会跳转到最近的 `try` 块的 `except` 语句，或者如果没有处理的话，程序会终止并打印出异常信息。

  在这个具体的例子中，如果传入的字符串值既不是 `true`、`false` 或 `yes`，也不是 `0`、`1` 或 `no`，那么就会触发 `raise ValueError(value)` 这一行，从而抛出一个包含传入的值的 `ValueError` 异常