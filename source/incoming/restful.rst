#######################################################
RESTful 资源处理
#######################################################

.. contents::
    :local:
    :depth: 2

表现层状态转移(REST)是一种对于分布式应用的架构风格，最初由Roy Fielding在他的2000年博士论文 `Architectural Styles and the Design of Network-based Software Architectures <https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm>`_ 所提出。 上文可能读起来有些枯燥，你也可以参照Martin Fowler的 `Richardson Maturity Model <https://martinfowler.com/articles/richardsonMaturityModel.html>`_ 以获得更方便的教程。

对于REST的架构方式，比起大多数软件架构体系，大家理解和误解得会更多。 或者可以这样说，当你将Roy Fielding的原则越多的使用在一个架构中，你的应用就会变得越”RESTful”。

CodeIgniter实现了一种简易的方式来创建RESTful APIs从而访问你的资源，通过其自带的资源路由和 `ResourceController` （资源控制器）。

资源路由
============================================================

您可以使用 ``resource()`` 方法为单个资源快速创建少量RESTful路由。这将创建资源完全CRUD所需的五个最常见的路由：创建新资源，更新现有资源，列出所有资源，显示单个资源以及删除单个资源。第一个参数是资源名称::

    $routes->resource('photos');

    // 相当于:
    $routes->get('photos/new',             'Photos::new');
    $routes->post('photos',                'Photos::create');
    $routes->get('photos',                 'Photos::index');
    $routes->get('photos/(:segment)',      'Photos::show/$1');
    $routes->get('photos/(:segment)/edit', 'Photos::edit/$1');
    $routes->put('photos/(:segment)',      'Photos::update/$1');
    $routes->patch('photos/(:segment)',    'Photos::update/$1');
    $routes->delete('photos/(:segment)',   'Photos::delete/$1');

.. note:: 上面的顺序是为了清楚起见，而在RouteCollection中创建路由的实际顺序可确保正确的路由解析

.. important:: 这些路由按照指定的顺序进行匹配，因此如果您在get 'photos/poll' 上方有资源照片，则show操作的资源行路由将在get行之前匹配。若要解决此问题，请将get行移到资源行上方，以便首先匹配它。


第二个参数接受一个选项数组，这些选项可用于修改生成的路由。尽管这些路由是针对API使用的，其中允许使用更多方法，但是您可以传递“websafe”选项以使其生成适用于HTML表单的更新和删除方法::

    $routes->resource('photos', ['websafe' => 1]);

    // 创建以下等效路由:
    $routes->post('photos/(:segment)/delete', 'Photos::delete/$1');
    $routes->post('photos/(:segment)',        'Photos::update/$1');

更改使用的控制器
--------------------------

您可以通过在 ``controller`` 选项中输入应该使用的控制器的名称来指定应该使用的控制器::

	$routes->resource('photos', ['controller' =>'App\Gallery']);

	// 将创建路由如下:
	$routes->get('photos', 'App\Gallery::index');

更改使用的占位符
---------------------------

默认情况下，当需要资源ID时使用 ``segment`` 占位符。您可以通过传递带有新字符串的 ``placeholder`` 选项来更改它以使用::

	$routes->resource('photos', ['placeholder' => '(:id)']);

	// 生成如下路由:
	$routes->get('photos/(:id)', 'Photos::show/$1');

限制生成的路由
---------------------

您可以限制使用该only选项生成的路由。这应该是应创建的方法名称的数组或逗号分隔列表。仅创建与这些方法之一匹配的路由。其余的将被忽略::

	$routes->resource('photos', ['only' => ['index', 'show']]);

否则，您可以使用 ``except`` 选项删除未使用的路由。该选项位于 ``only`` 之后::

	$routes->resource('photos', ['except' => 'new,edit']);

有效方法包括: index, show, create, update, new, edit and delete。

资源控制器
============================================================

`ResourceController` 为您的RESTful API提供一个方便的起点，此方法对应于上述资源的途径。

对其进行扩展，以覆盖 `modelName` 和 `format` 属性，然后实现要处理的那些方法。::

	<?php namespace App\Controllers;

	use CodeIgniter\RESTful\ResourceController;

	class Photos extends ResourceController
	{

		protected $modelName = 'App\Models\Photos';
		protected $format    = 'json';

		public function index()
		{
			return $this->respond($this->model->findAll());
		}

                // ...
	}

路由如下::

    $routes->resource('photos');

表现层路由
============================================================

您可以使用 ``presenter()`` 方法快速创建与资源控制器对齐的表现控制器。这将为控制器方法创建路由，这些路由将返回资源的视图或处理从这些视图提交的表单。

不需要，因为可以使用常规控制器处理演示，这很方便。它的用法类似于资源路由::

    $routes->presenter('photos');

    // 相当于:
    $routes->get('photos/new',                'Photos::new');
    $routes->post('photos/create',            'Photos::create');
    $routes->post('photos',                   'Photos::create');   // alias
    $routes->get('photos',                    'Photos::index');
    $routes->get('photos/show/(:segment)',    'Photos::show/$1');
    $routes->get('photos/(:segment)',         'Photos::show/$1');  // alias
    $routes->get('photos/edit/(:segment)',    'Photos::edit/$1');
    $routes->post('photos/update/(:segment)', 'Photos::update/$1');
    $routes->get('photos/remove/(:segment)',  'Photos::remove/$1');
    $routes->post('photos/delete/(:segment)', 'Photos::update/$1');

.. note:: 上面的顺序是为了清楚起见，而在RouteCollection中创建路由的实际顺序可确保正确的路由解析

您将没有资源和表现层控制器的 `photos` 路由。您需要区分它们，例如::

    $routes->resource('api/photo');
    $routes->presenter('admin/photos');


第二个参数接受一个选项数组，这些选项可用于修改生成的路由。

更改使用的控制器
--------------------------

您可以通过在 ``controller`` 选项中输入应该使用的控制器的名称来指定应该使用的控制器::

	$routes->presenter('photos', ['controller' =>'App\Gallery']);

	// 将创建路由如下:
	$routes->get('photos', 'App\Gallery::index');

更改使用的占位符
---------------------------

默认情况下，当需要资源ID时使用 ``segment`` 占位符。您可以通过传递带有新字符串的 ``placeholder`` 选项来更改它以使用::

	$routes->presenter('photos', ['placeholder' => '(:id)']);

	// 生成如下路由:
	$routes->get('photos/(:id)', 'Photos::show/$1');

限制生成的路由
---------------------

您可以限制使用 ``only`` 选项生成的路由。这应该是创建的方法名称的数组或逗号分隔列表。仅创建与这些方法之一匹配的路由。其余的将被忽略::

	$routes->presenter('photos', ['only' => ['index', 'show']]);

否则，您可以使用 ``except`` 选项删除未使用的路由。该选项位于 ``only`` 之后::

	$routes->presenter('photos', ['except' => 'new,edit']);

有效方法包括: index, show, new, create, edit, update, remove and delete。

ResourcePresenter
============================================================

`ResourcePresenter` 为你输出一个资源对应的视图提供了一个便捷的起点，而它同样也可利用属于该资源路由的方法来处理这些视图里提交的表单。

对其进行扩展，以覆盖 `modelName` 属性，然后实现要处理的那些方法。::

	<?php namespace App\Controllers;

	use CodeIgniter\RESTful\ResourcePresenter;

	class Photos extends ResourcePresenter
	{

		protected $modelName = 'App\Models\Photos';

		public function index()
		{
			return view('templates/list',$this->model->findAll());
		}

                // ...
	}

路由如下::

    $routes->presenter('photos');

Presenter/Controller 对比
=============================================================

该表比较了 `resource()` 和 `presenter()` 创建的默认路由及其对应的Controller函数。

================ ========= ====================== ======================== ====================== ======================
Operation        Method    Controller Route       Presenter Route          Controller Function    Presenter Function
================ ========= ====================== ======================== ====================== ======================
**New**          GET       photos/new             photos/new               ``new()``              ``new()``
**Create**       POST      photos                 photos                   ``create()``           ``create()``
Create (alias)   POST                             photos/create                                   ``create()``
**List**         GET       photos                 photos                   ``index()``            ``index()``
**Show**         GET       photos/(:segment)      photos/(:segment)        ``show($id = null)``   ``show($id = null)``
Show (alias)     GET                              photos/show/(:segment)                          ``show($id = null)``
**Edit**         GET       photos/(:segment)/edit photos/edit/(:segment)   ``edit($id = null)``   ``edit($id = null)``
**Update**       PUT/PATCH photos/(:segment)                               ``update($id = null)`` 
Update (websafe) POST      photos/(:segment)      photos/update/(:segment) ``update($id = null)`` ``update($id = null)``
**Remove**       GET                              photos/remove/(:segment)                        ``remove($id = null)``
**Delete**       DELETE    photos/(:segment)                               ``delete($id = null)`` 
Delete (websafe) POST                             photos/delete/(:segment) ``delete($id = null)`` ``delete($id = null)``
================ ========= ====================== ======================== ====================== ======================
