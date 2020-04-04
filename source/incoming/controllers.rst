###########
控制器
###########

控制器是应用程序的核心，因为它们确定应如何处理HTTP请求。

.. contents::
    :local:
    :depth: 2


什么是控制器？
=====================

控制器只是一个类文件，它以可以与URI关联的方式命名。

考虑以下URI::

	example.com/index.php/helloworld/

在上面的示例中，CodeIgniter将尝试查找名为 ``Helloworld.php`` 的控制器并将其加载。

**当控制器的名称与URI的第一片段匹配时，它将被加载。**

让我们尝试一下: Hello World!
==================================

让我们创建一个简单的控制器，以便您可以实际看到它。使用您的文本编辑器，创建一个名为 ``Helloworld.php`` 的文件，并将以下代码放入其中::

	<?php namespace App\Controllers;

        use CodeIgniter\Controller;

	class Helloworld extends Controller
        {
		public function index()
		{
			echo 'Hello World!';
		}
	}

然后将文件保存到 **/app/Controllers/** 目录。

.. important:: 该文件必须名为“Helloworld.php”，大写字母“ H”。

现在，使用类似于以下网址的网址访问您的网站::

	example.com/index.php/helloworld

如果操作正确，您应该看到::

	Hello World!

.. important:: 控制器类名称必须以大写字母开头，并且第一个字符只能是大写字母。

这是 **有效的**::

	<?php namespace App\Controllers;

        use CodeIgniter\Controller;

	class Helloworld extends Controller {

	}

这是 **无效的**::

	<?php namespace App\Controllers;

        use CodeIgniter\Controller;

	class helloworld extends Controller {

	}

这是 **无效的**::

	<?php namespace App\Controllers;

        use CodeIgniter\Controller;

	class HelloWorld extends Controller {

	}

另外，请始终确保您的控制器扩展了父控制器类，以便它可以继承其所有方法。

方法
=======

在上面的示例中，方法名称为 ``index()``。如果URI的 **第二段** 为空，则默认情况下始终加载"index"方法。显示“Hello World”消息的另一种方式是::

	example.com/index.php/helloworld/index/

**URI的第二部分确定调用控制器中的哪个方法。**

让我们尝试一下。向您的控制器添加新方法::

	<?php namespace App\Controllers;

        use CodeIgniter\Controller;

	class Helloworld extends Controller
        {

		public function index()
		{
			echo 'Hello World!';
		}

		public function comment()
		{
			echo 'I am not flat!';
		}
	}

现在加载以下URL以查看 ``comment()`` 方法::

	example.com/index.php/helloworld/comment/

您应该会看到新消息。

传递URI片段到方法
====================================

如果您的URI包含两个以上的片段，它们将作为参数传递给您的方法。

例如，假设您有一个这样的URI::

	example.com/index.php/products/shoes/sandals/123

您的方法将传递URI片段3和4（"sandals" 和 "123"）::

	<?php namespace App\Controllers;

        use CodeIgniter\Controller;

	class Products extends Controller
        {

		public function shoes($sandals, $id)
		{
			echo $sandals;
			echo $id;
		}
	}

.. important:: 如果您使用的是 :doc:`URI路由 <routing>` 功能，则传递给您的方法的片段将是重新路由的片段。

定义默认控制器
=============================

当URI不存在时，可以告诉CodeIgniter加载默认控制器，就像仅请求站点根URL时的情况一样。让我们使用Helloworld控制器尝试一下。

要指定默认控制器，请打开您的 **app/Config/Routes.php** 文件并设置以下变量::

	$routes->setDefaultController('Helloworld');

其中“Helloworld”是要使用的控制器类的名称。

在“路由定义”部分的 **Routes.php** 后面几行，将该行注释掉

$routes->get('/', 'Home::index');

如果您现在浏览到站点而未指定任何URI细分，则会看到“ Hello World”消息。

.. note:: ``$routes->get('/', 'Home::index');`` 行是您将要在“real-world”应用程序中使用的优化。但是出于演示目的，我们不想使用该功能。``$routes->get()`` 在 :doc:`URI路由 <routing>` 中进行了解释

有关更多信息，请参考 :doc:`URI路由 <routing>` 文档的“路由配置选项”部分 。

重新映射方法调用
======================

如上所述，URI的第二部分通常确定调用控制器中的哪个方法。CodeIgniter允许您通过使用 ``_remap()`` 方法来覆盖此行为::

	public function _remap()
	{
		// Some code here...
	}

.. important:: 如果您的控制器包含一个名为_remap()的方法，则无论您的URI包含什么内容，它将始终被调用。它覆盖了URI确定调用哪个方法的常规行为，从而允许您定义自己的方法路由规则。

重写的方法调用（通常是URI的第二段）将作为参数传递给 ``_remap()`` 方法::

	public function _remap($method)
	{
		if ($method === 'some_method')
		{
			return $this->$method();
		}
		else
		{
			return $this->default_method();
		}
	}

方法名称之后的所有其他段都将传递到中 ``_remap()``。可以将这些参数传递给该方法，以模拟CodeIgniter的默认行为。

示例::

	public function _remap($method, ...$params)
	{
		$method = 'process_'.$method;
		if (method_exists($this, $method))
		{
			return $this->$method(...$params);
		}
		throw \CodeIgniter\Exceptions\PageNotFoundException::forPageNotFound();
	}

私有方法
===============

在某些情况下，您可能希望对公共访问隐藏某些方法。为此，只需将方法声明为private或protected。这样可以防止URL请求为它提供服务。例如，如果要为 `Helloworld` 控制器定义这样的方法::

	protected function utility()
	{
		// some code
	}

那么尝试使用以下URL进行访问将不起作用::

	example.com/index.php/helloworld/utility/

将控制器放入子目录中
================================================

如果要构建大型应用程序，则可能需要按层次结构将控制器组织或构造为子目录。CodeIgniter允许您执行此操作。

只需在主 *app/Controllers/* 目录下创建一个子目录， 然后将控制器类放入其中。

.. note:: 使用此功能时，URI的第一部分必须指定文件夹。例如，假设您在此处有一个控制器::

		app/Controllers/products/Shoes.php

	要调用上述控制器，您的URI将如下所示::

		example.com/index.php/products/shoes/show/123

您的每个子目录可能包含一个默认控制器，如果URL 仅包含子目录，则将调用该默认控制器。只需在其中放置一个与 *app/Config/Routes.php* 文件中指定的“ default_controller”名称匹配的控制器即可。

CodeIgniter还允许您使用其 :doc:`URI路由 <routing>` 功能重新映射URI 。

包含属性
===================

您创建的每个控制器都应扩展 ``CodeIgniter\Controller`` 类。此类提供了一些可用于所有控制器的功能。

**Request 对象**

应用程序的主 :doc:`Request实例 </incoming/request>` 始终可作为类属性来使用 ``$this->request``。

**Response 对象**

应用程序的主 :doc:`Response实例 </outgoing/response>` 始终可作为类属性来使用 ``$this->response``。

**Logger 对象**

:doc:`Logger <../general/logging>` 类的实例可用作类属性 ``$this->logger``。

**forceHTTPS**

在所有控制器中都提供了一种强制通过HTTPS访问方法的便捷方法::

	if (! $this->request->isSecure())
	{
		$this->forceHTTPS();
	}

默认情况下，在支持 ``HTTP Strict Transport Security`` 标头的现代浏览器中，此调用应强制浏览器将非HTTPS调用转换为HTTPS调用一年。您可以通过将持续时间（以秒为单位）作为第一个参数传递来进行修改::

	if (! $this->request->isSecure())
	{
		$this->forceHTTPS(31536000);    // 1年
	}

.. note:: 始终可以使用许多 :doc:`基于时间的常量 </general/common_functions>` ，包括YEAR，MONTH等。

辅助函数
-----------

您可以将辅助程序文件数组定义为类属性。无论何时加载控制器，这些辅助文件都会自动加载到内存中，以便您可以在控制器内部的任何位置使用它们的方法::

	namespace App\Controllers;
        use CodeIgniter\Controller;

	class MyController extends Controller
	{
		protected $helpers = ['url', 'form'];
	}

验证数据
======================

为了简化数据检查，控制器还提供了便捷方法 ``validate()``。该方法在第一个参数中接受一个规则数组，在可选的第二个参数中接受一个自定义错误消息数组，以显示这些项是否无效。在内部，使用控制器的 **$this->request** 实例来获取要验证的数据。 :doc:`验证库文档 </libraries/validation>` 对规则和消息数组形式的详细信息，以及现有的规则::

    public function updateUser(int $userID)
    {
        if (! $this->validate([
            'email' => "required|is_unique[users.email,id,{$userID}]",
            'name'  => 'required|alpha_numeric_spaces'
        ]))
        {
            return view('users/update', [
                'errors' => $this->validator->getErrors()
            ]);
        }

        // do something here if successful...
    }

如果您发现将规则保留在配置文件中更为简单，则可以将$rules数组替换为组的名称，定义在 ``Config\Validation.php``::

    public function updateUser(int $userID)
    {
        if (! $this->validate('userRules'))
        {
            return view('users/update', [
                'errors' => $this->validator->getErrors()
            ]);
        }

        // do something here if successful...
    }

.. note:: 验证也可以在模型中自动处理，但有时在控制器中更容易进行。在哪里取决于你。

就这样了！
==========

简而言之，这就是控制器的全部知识。
