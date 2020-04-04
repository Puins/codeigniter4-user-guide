##############################
处理多环境
##############################

开发人员通常希望根据应用程序是在开发环境中还是在生产环境中运行来实现不同的系统行为。例如，详细的错误输出在开发应用程序时会很有用，但在“运行”时也会带来安全问题。在开发环境中，您可能需要加载生产环境中没有的其他工具，等等。

环境常量
========================

默认情况下，CodeIgniter 使用 ``$_SERVER['CI_ENVIRONMENT']`` 的值作为 ENVIRONMENT 常量，否则默认为'production'。有几种方式可以根据您的服务器设置类设置。

.env
----

最简单的方式是在你的 :doc:`.env 文件 </general/configuration>` 里设置。

.. code-block:: ini

    CI_ENVIRONMENT = development

Apache
------

可以使用 ``.htaccess`` 文件或Apache配置用 `SetEnv <https://httpd.apache.org/docs/2.2/mod/mod_env.html#setenv>`_ 设置此服务器变量。

.. code-block:: apache

    SetEnv CI_ENVIRONMENT development

nginx
-----

在nginx下，必须通过 ``fastcgi_params`` 传递环境变量 ，以便使其显示在 `$_SERVER` 变量下。这使它可以在虚拟主机级别上工作，而不是使用 `env` 为整个服务器设置它，尽管在专用服务器上可以正常工作。然后，您将服务器配置修改为:

.. code-block:: nginx

	server {
	    server_name localhost;
	    include     conf/defaults.conf;
	    root        /var/www;

	    location    ~* \.php$ {
	        fastcgi_param CI_ENVIRONMENT "production";
	        include conf/fastcgi-php.conf;
	    }
	}

可选方法适用于 nginx 和其它服务器，或者你也可以完全移除这部分逻辑，并根据服务器的 IP 地址设置常量(实例)。

除了影响某些基本框架行为（请参阅下一节）之外，您还可以在自己的开发中使用此常量来区分正在运行的环境。

启动文件
----------

CodeIgniter要求与环境名称匹配的PHP脚本位于 **APPPATH/Config/Boot** 下。这些文件可以包含您要针对环境进行的任何自定义，无论是更新错误显示设置，加载其他开发人员工具还是进行其他任何操作。这些由系统自动加载。全新安装中已经创建了以下文件：

* development.php
* production.php
* testing.php

对默认框架行为的影响
=====================================

在CodeIgniter系统中的某些地方使用了环境常量。本节介绍如何影响默认框架行为。

错误报告
---------------

将ENVIRONMENT常量设置为 'development' 的值将导致所有PHP错误在发生时呈现给浏览器。相反，将常量设置为 'production' 将禁用所有错误输出。在生产中禁用错误​​报告是一种 :doc:`很好的安全措施 </concepts/security>` 。

配置文件
-------------------

另外，您可以让CodeIgniter加载特定于环境的配置文件。这对于在多个环境中管理诸如不同的API密钥之类的事情可能很有用。这在 :doc:`配置 </general/configuration>` 文档中的“处理不同的环境”一节进行了详细描述。
