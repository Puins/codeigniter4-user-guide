##############
错误处理
##############

CodeIgniter通过 `SPL 集合 <https://www.php.net/manual/en/spl.exceptions.php>`_ 以及框架提供的一些自定义异常来将错误报告构建到系统中。根据环境的设置，除非应用程序在 ``production`` 环境下运行，否则抛出错误或异常时的默认操作是显示详细的错误报告。在这种情况下，将显示一条更通用的消息，为您提供最佳的用户体验。

.. contents::
    :local:
    :depth: 2

使用异常处理
================

本节为新手程序员或不熟悉使用异常的开发人员提供快速概述。

异常处理只是在“抛出”异常时发生的事件。这将停止脚本的当前流程，然后将执行发送到错误处理程序，该处理程序将显示相应的错误页面::

	throw new \Exception("Some message goes here");

如果您正在调用可能引发异常的方法，则可以使用代码 ``try/catch`` 块捕获该异常::

	try {
		$user = $userModel->find($id);
	}
	catch (\Exception $e)
	{
		die($e->getMessage());
	}

如果 ``$userModel`` 引发异常，则将其捕获并执行catch块中的代码。在此示例中，脚本终止并输出了 ``UserModel`` 定义的错误信息。

在此示例中，我们可以捕捉任意类型的异常。如果我们只想监视特定类型的异常，例如 ``UnknownFileException``，则可以在catch参数中进行指定。引发且不是捕获的异常的子类的任何其他异常将传递给错误处理程序::

	catch (\CodeIgniter\UnknownFileException $e)
	{
		// do something here...
	}

这对于自己处理错误或在脚本结束之前执行清除很方便。如果希望错误处理程序正常运行，则可以在catch块中引发新的异常::

	catch (\CodeIgniter\UnknownFileException $e)
	{
		// do something here...

		throw new \RuntimeException($e->getMessage(), $e->getCode(), $e);
	}

配置
=============

默认情况下，CodeIgniter将在 ``development`` 和 ``testing`` 环境中显示所有错误，并且在 ``production`` 环境中不显示任何错误。您可以通过 ``CI_ENVIRONMENT`` 在 ``.env`` 文件中设置变量来更改此设置。

.. important:: 禁用错误报告不会在出现错误时停止写入日志。

日志异常
------------------

默认情况下，将记录“404-未找到页面”以外的所有异常。可以通过设置 ``Config\Exceptions`` 中的 **$log** 的值来打开和关闭它::

    class Exceptions
    {
        public $log = true;
    }

要忽略记录其他状态代码，可以将状态代码设置为在同一文件中忽略::

    class Exceptions
    {
        public $ignoredCodes = [ 404 ];
    }

.. note:: 如果您当前的日志设置未设置为记录 **critical** 错误（所有异常都记录为该错误），则对于异常仍然无法进行日志记录。

自定义异常
=================

以下自定义异常可用:

PageNotFoundException
---------------------

这用于表示404，找不到页面错误。引发时，系统将显示在 ``/app/views/errors/html/error_404.php`` 处找到的视图。您应该自定义网站的所有错误视图。如果在 ``Config/Routes.php`` 中指定了404的重写规则，那么它将代替标准的 404 页来被调用::

	if (! $page = $pageModel->find($id))
	{
		throw \CodeIgniter\Exceptions\PageNotFoundException::forPageNotFound();
	}

您可以将消息传递到异常中，该消息将代替404页上的默认消息显示。

ConfigException
---------------
当配置类中的值无效或配置类不是正确的类型时等等，应使用此异常::

	throw new \CodeIgniter\Exceptions\ConfigException();

这提供了HTTP状态代码 `500` 和退出代码 `3`。

DatabaseException
-----------------

由于数据库错误（例如，无法创建数据库连接或临时丢失数据库连接）时，抛出此异常::

	throw new \CodeIgniter\Database\Exceptions\DatabaseException();

这提供了HTTP状态代码 `500` 和退出代码 `8`。

RedirectException
-----------------

此异常是一种特殊情况，它允许覆盖所有其他响应路由并强制重定向到特定路由或URL::

	throw new \CodeIgniter\Router\Exceptions\RedirectException($route);

``$route`` 可以是命名路由，相对URI或完整URL。您还可以提供重定向代码来代替默认代码（ ``302``，"temporary redirect"）::

	throw new \CodeIgniter\Router\Exceptions\RedirectException($route, 301);
