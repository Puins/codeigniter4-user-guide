######################
CodeIgniter4 用户指南
######################

******************
安装说明
******************

CodeIgniter4 用户指南是使用 Sphinx 软件进行管理，并可以生成各种不同的格式。 
所有的页面都是采用 `ReStructured Text <http://sphinx.pocoo.org/rest.html>`_ 格式书写，这种格式非常方便人们阅读。

安装条件
=============

Python
------

如果你使用的是 OS X 或 Linux系统，Sphinx 需要 Python 3.5+版本。你可以在终端中执行 ``python --version`` 或 ``python3 --version`` 命令，以确认你的系统Python是否已安装或版本。

.. code-block:: bash

	python --version
	Python 2.7.17

	python3 --version
	Python 3.6.9

如果您使用的不是3.5+版本，请继续并从以下版本安装最新的3.x版本 `Python.org <https://www.python.org/downloads/>`_。 Linux用户应使用系统内置的软件包管理器更新。

pip
---

现在您已经启动并运行了Python 3.x，我们将进行安装 `pip <https://pip.pypa.io/en/stable/>`_ （Python软件包安装程序）。

您可以通过pip或pip3检查是否已安装了pip。如您所见，pip遵循与Python相同的命名约定。请注意，它应该在最后说  ``python 3.x`` 。

.. code-block:: bash

	pip --version
	pip 9.0.1 from /usr/lib/python2.7/dist-packages (python 2.7)

	pip3 --version
	pip 9.0.1 from /usr/lib/python3/dist-packages (python 3.6)

Linux
^^^^^

使用 `Linux软件包管理器 <https://packaging.python.org/guides/installing-using-linux-tools/>`_ 安装 pip/setuptools/wheel


Other
^^^^^

如果您使用的是从 `Python.org <https://www.python.org/downloads/>`_ 下载的Python 3.5+，则已经安装了pip。

安装
============

现在我们需要安装Sphinx及其依赖项。 使用 ``pip`` or ``pip3`` 取决于操作系统。 完成此步骤后，您需要重新启动终端窗口，因为Python无法找到我们刚刚安装过的其他所有应用程序。

.. code-block:: bash

	pip install -r user_guide_src/requirements.txt

	pip3 install -r user_guide_src/requirements.txt

现在是时候整理一下内容并生成文档了。

.. code-block:: bash

	cd user_guide_src
	make html

编辑并创建文档
==================================

所有的源文件都在 *source/* 目录下，在这里你可以添加新的文档或修改已有的文档。就像代码更改一样，我们建议从功能分支工作，并向此仓库的 *develop* 分支发出请求。

那么，HTML 文档在哪里？
====================

很显然，HTML文档才是我们最关心的，因为这毕竟才是用户最终看到的。由于对自动生成的文件进行版本控制没有意义，所以它们并不在版本控制之下。你如果想要预览 HTML 文档，你可以重新生成它们。生成 HTML 文档非常简单。首先进入你的用户指南存储库根目录，然后执行上面安装步骤中的最后一步::

	make html

你将会看到正在编译中的信息，编译成功后，生成的用户指南和图片都位于 *HTMLDIR* 定义(例 Makefile在第9行中定义 HTMLDIR= html)的目录下。在 HTML 第一次编译之后，后面将只会针对修改的文件进行重编译，这将大大的节约我们的时间。如果你想再重新全部编译一次，只需删除 *build* 目录然后编译即可。

***************
风格准则
***************

使用 Sphinx 为 CodeIgniter4 编写文档，请参考 /contributing/documentation.rst 的一般准则。
