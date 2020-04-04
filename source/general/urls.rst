################
CodeIgniter URLs
################

默认情况下，CodeIgniter 中的 URL 被设计成对搜索引擎和用户友好的模式。CodeIgniter并未使用与动态系统同义的URL的标准“查询字符串”方法，而是使用基于段的方法::

	example.com/news/article/my_article

URI 片段
============

依照Model-View-Controller（MVC）方法, URL中的片段通常代表::

	example.com/class/method/ID

1. 第一部分代表应该调用的控制器 **class** 。
2. 第二部分代表应调用的类 **method** 。
3. 第三部分以及任何其他段代表ID和将传递给控制器​​的任何变量。

:doc:`URI 库 <../libraries/uri>` 和 :doc:`URL 辅助函数 <../helpers/url_helper>` 包含的功能可以让你更容易的处理 URI 数据。另外，可以使用 :doc:`URI 路由 </incoming/routing>` 功能重新映射您的URL，以提高灵活性。

移除 index.php 文件
===========================

默认情况下，**index.php** 文件将包含在您的URL中::

	example.com/index.php/news/article/my_article

如果您的服务器支持重写URL，则可以通过URL重写轻松移除此文件。不同的服务器对此处理方式有所不同，但是我们将在此处显示两个最常见的Web服务器的示例。

Apache服务器
-----------------

Apache必须启用 *mod_rewrite* 扩展。这样的话，我们可以使用具有一些简单规则的 ``.htaccess`` 文件。以下是使用 "negative" 方法的此类文件的示例，其中，除了指定项目外，所有内容都被重定向：

.. code-block:: apache

	RewriteEngine On
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteRule ^(.*)$ index.php/$1 [L]

在此示例中，除针对现有目录和现有文件的HTTP请求外，任何其他HTTP请求均被视为对 ``index.php`` 文件的请求。

.. note:: 这些特定规则可能不适用于所有服务器配置。

.. note:: 确保还从上述规则中排除可能需要从外界访问的任何资源。

NGINX
-----

在NGINX下，您可以定义一个位置块，并使用该 ``try_files`` 指令获得与上述Apache配置相同的效果：

.. code-block:: nginx

	location / {
		try_files $uri $uri/ /index.php$is_args$args;
	}

这将首先查找与URI匹配的文件或目录（根据root和alias伪指令的设置来构造每个文件的完整路径），然后将请求连同任何参数一起发送到 ``index.php`` 文件。
