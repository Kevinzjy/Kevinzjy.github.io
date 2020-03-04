---
title: 上传 python 模块到 pypi.org
date: 2020-3-4 18:55:20
categories: Python
tags: [Linux, Python]
---

#### 准备工作

1. python 模块中已编写好的 setup.py

2. 安装所需模块

```bash
pip install wheel setuptools
pip install twine
```

#### 打包脚本

```bash
python setup.py sdist bdist_wheel
```

打包完成后，会在 `dist/` 文件夹下产生两个打包好的压缩包, 分别对应 setuptools 和 easy_install 两种安装方式

#### 测试脚本

```bash
twine check dist/*
```

这一步目的是为了检查脚本是否符合规范

#### 预发布测试

```bash
twine upload -repository-url https://test.pypi.org/legacy/ dist/*
```

`test.pypi.org` 是一个与 `pypi.org` 互相独立的平台，可以用于在正式版本发布前进行测试，这里进行的修改不会影响到 pypi.org 上的内容

#### 上传到 pypi.org

```bash
twine upload dist/*
```

根据提示输入用户名/密码后，即可成功发布软件到 pypi.org

**发布成功后，就可以直接使用 `pip install` 命令来安装我们的软件了**
