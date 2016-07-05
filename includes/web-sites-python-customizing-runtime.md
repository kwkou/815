Azure 将按以下优先级确定要用于其虚拟环境的 Python 的版本：

1. 在根文件夹中的 runtime.txt 内指定的版本
1. 由 Web 应用配置（在 Azure 经典管理门户上"配置"页面）中的 Python 设置指定的版本
1. 如果未指定以上任何项，则 python 2.7 是默认值

内容的有效值 

    \runtime.txt

是：

- python-2.7
- python-3.4

如果指定了 micro 版本（第三个数字）时，将其忽略。
