#########################
生成用户指南
#########################

最终目的是使正在进行的用户指南自动作为PR合并的一部分而生成。 同时该文章说明了如何手动完成。

用户指南充分利用了Github页面，其中的 "gh-pages" 分支可通过以下方式访问仅包含HTML的存储库 `github.io <https://bcit-ci.github.io/CodeIgniter4>`_。

存储库设置
==========================

您已经将存储库克隆到项目文件夹中的 ``CodeIgniter4`` 中。创建与此级别相同的另一个文件夹 ``CodeIgniter4-guide``。再次将CodeIgniter4存储库克隆到 ``CodeIgniter4-guide/html`` 中。

在 ``html`` 文件夹中，``git checkout gh-pages``。您应该看到的是用户指南生成的HTML。

重新生成用户指南
============================

在 ``user_guide_src`` src文件夹中，生成传统的用户指南，使用如下命令测试::

	make html

已配置另一个目标，它将生成相同的HTML，但在第二个repo克隆的 ``html`` 文件夹中::

	make ghpages

设定此目标后，通过切换到 ``CodeIgniter4-guide/html`` 文件夹更新在线用户指南，然后::

	git add .
	git commit -S -m "Suitable comment"
	git push origin gh-pages

过程
=======

应该只有一个维护人员可以这样做，以避免发生冲突。每当出现PR合并时，都会重新生成用户指南。

Note: 您可能必须先删除 ``user_guide_src/doctree`` 文件夹制作指南的 ``gh-pages`` 版本之前，以确保目录已正确重建，尤其是在您多次重建 ``html`` 目标时。
