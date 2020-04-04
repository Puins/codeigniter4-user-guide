########
服务
########

.. contents::
    :local:
    :depth: 2

介绍
============

CodeIgniter中的所有类均作为“服务”提供。这仅意味着，要对要调用的类进行硬定义，而不是对要加载的类名称进行硬编码，而是在一个非常简单的配置文件中定义它们。该文件是一种工厂类型，用于创建所需类的新实例。

一个简单的示例可能会使事情变得更清楚，因此可以想象您需要引入Timer类的实例。最简单的方法就是创建该类的新实例::

	$timer = new \CodeIgniter\Debug\Timer();

这很棒。直到您决定要使用其他计时器类代替它。也许这有一些高级的报告，默认计时器没有提供。为此，您现在必须在使用计时器类的应用程序中找到所有位置。由于您可能已将它们留在原处以保持应用程序的性能日志持续运行，因此这可能是一种耗时且容易出错的方法。那就是服务派上用场的地方了。

我们不用自己创建实例，而是让中央类为我们创建该类的实例。此类非常简单。它仅包含我们要用作服务的每个类的方法。该方法通常返回该类的共享实例，并将其可能具有的所有依赖关系传递给它。然后，我们将用调用此新类的代码替换计时器创建代码::

	$timer = \Config\Services::timer();

当需要更改所使用的实现时，可以修改服务配置文件，并且不必做任何事情，更改即可自动在整个应用程序中进行。现在，您只需要使用所有新功能，就可以了。很简单而且防错。

.. note:: 建议仅在控制器内创建服务。其他文件，例如模型和库，应将依赖项传递到构造函数中或通过 `setter` 方法传递。

便利功能
---------------------

提供了两种功能来获得服务。这些功能始终可用。

第一个是 ``service()`` 返回所请求服务的新实例。唯一需要的参数是服务名称。这与Services文件中的方法名称始终返回该类的SHARED实例相同，因此多次调用该函数应始终返回同一实例::

	$logger = service('logger');

如果创建方法需要其他参数，则可以在服务名称之后传递它们::

	$renderer = service('renderer', APPPATH.'views/');

第二个函数的 ``single_service()`` 的工作原理和 ``service()`` 的类似，但返回类的新实例::

	$logger = single_service('logger');

定义 Services
=================

为了使服务正常运行，您必须能够依赖具有恒定API或 `接口 <https://www.php.net/manual/en/language.oop5.interfaces.php>`_ 才能使用的每个类 。几乎所有的CodeIgniter类都提供了它们遵循的接口。当您要扩展或替换核心类时，只需要确保满足接口的要求并且知道这些类是兼容的即可。

例如，``RouterCollection`` 类实现 ``RouterCollectionInterface``。当您想创建一个提供不同方式创建路由的替代品时，只需创建一个实现以下内容的新类 ``RouterCollectionInterface``::

	class MyRouter implements \CodeIgniter\Router\RouteCollectionInterface
	{
		// Implement required methods here.
	}

最后，修改 **/app/Config/Services.php** 以创建新的实例 ``MyRouter``，而不是 ``CodeIgniter\Router\RouterCollection``::

	public static function routes()
	{
		return new \App\Router\MyRouter();
	}

允许参数
-------------------

在某些情况下，您希望在类实例化期间将设置选项传递给类。由于服务文件是非常简单的类，因此轻松进行此工作。

``renderer`` 服务就是一个很好的例子。默认情况下，我们希望此类能够在 ``APPPATH.views/`` 中找到视图文件。但是，如果开发人员需要，我们希望开发人员可以选择更改该路径。因此，该类接受 ``$viewPath`` 作为构造函数参数。服务方法如下::

	public static function renderer($viewPath=APPPATH.'views/')
	{
		return new \CodeIgniter\View\View($viewPath);
	}

这将在构造方法中设置默认路径，但允许轻松更改其使用的路径::

	$renderer = \Config\Services::renderer('/shared/views');

共享类
-----------------

在某些情况下，您要求仅创建服务的单个实例。用从工厂方法内部调用的方法 ``getSharedInstance()`` 很容易处理。这用于检查实例是否已在类中创建并保存，如果没有，则创建一个新实例。所有工厂方法都提供一个 ``$getShared = true`` 值作为最后一个参数。您也应该坚持使用该方法::

    class Services
    {
        public static function routes($getShared = false)
        {
            if (! $getShared)
            {
                return new \CodeIgniter\Router\RouteCollection();
            }

            return static::getSharedInstance('routes');
        }
    }

Service 发现
-----------------

CodeIgniter可以自动发现您可能在任何定义的命名空间中创建的任何 ``Config\\Services.php`` 文件。这样可以简单地使用任何模块服务文件。为了发现自定义服务文件，它们必须满足以下要求：

- 其命名空间必须在 ``Config\Autoload.php`` 定义
- 在命名空间内，必须在 ``Config\Services.php`` 找到文件 
- 它必须扩展 ``CodeIgniter\Config\BaseService``

一个小例子应该能说明这一点。

想象一下，您已经在根目录中创建了一个新目录， ``Blog``。这将包含一个带有控制器，模型等的 **blog module**，并且您想将某些类作为服务使用。第一步是创建一个新文件： ``Blog\Config\Services.php``。该文件的框架应为::

    <?php namespace Blog\Config;

    use CodeIgniter\Config\BaseService;

    class Services extends BaseService
    {
        public static function postManager()
        {
            ...
        }
    }

现在，您可以如上所述使用此文件。当您想从任何控制器获取 ``posts`` 服务时，只需使用框架的 ``Config\Services`` 类即可获取服务::

    $postManager = Config\Services::postManager();

.. note:: 如果多个服务文件具有相同的方法名称，则第一个找到的将是返回的实例。
