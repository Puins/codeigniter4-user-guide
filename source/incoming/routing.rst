###########
URI 路由
###########

.. contents::
    :local:
    :depth: 2

通常，URL字符串与其对应的控制器类/方法之间存在一对一的关系。URI中的片段通常遵循以下模式::

    example.com/class/function/id/

但是，在某些情况下，您可能需要重新映射此关系，以便可以调用不同的类/方法，而不是对应于URL的类/方法。

例如，假设您希望您的URL具有此原型::

    example.com/product/1/
    example.com/product/2/
    example.com/product/3/
    example.com/product/4/

通常，URL的第二段是为方法名称保留的，但在上面的示例中，它具有产品ID。为了克服这个问题，CodeIgniter允许您重新映射URI处理程序。

设置路由规则
==============================

路由规则在 **app/config/Routes.php** 文件中定义。在其中，您将看到它创建了RouteCollection类的实例，该实例允许您指定自己的路由条件。可以使用通配符或正则表达式指定路由。

路由仅使用左侧的URI，并将其与应传递给控制器​​的所有参数一起映射到右侧的控制器和方法。控制器和方法的列出方式应与使用静态方法的方式相同，方法是使用双冒号（如）分隔完全命名空间的类及其方法 ``Users::list``。如果该方法要求将参数传递给它，则它们将在方法名称后列出，并用正斜杠分隔::

	// 调用 $Users->list()
	Users::list
	// 调用 $Users->list(1, 23)
	Users::list/1/23

通配符
============

典型的路由可能如下所示::

    $routes->add('product/(:num)', 'App\Catalog::productLookup');

在路由中，第一个参数包含要匹配的URI，而第二个参数包含应重新路由到的目标。在上面的示例中，如果在URL的第一段中找到单词“product”，而在第二段中找到数字，则使用 **App\\Catalog** 类和 **productLookup** 方法代替。

通配符只是代表正则表达式模式的字符串。在路由过程中，这些通配符将替换为正则表达式的值。它们主要用于可读性。

您可以在路由中使用以下通配符:

* **(:any)** 将匹配从该点到URI末尾的所有字符。这可能包括多个URI段。
* **(:segment)** 将匹配任何字符，但正斜杠（/）会将结果限制为单个段。
* **(:num)** 将匹配任何整数。
* **(:alpha)** 将匹配任何字母字符串
* **(:alphanum)** 将匹配任何字母字符串或整数，或两者的任意组合。
* **(:hash)** 与 **:segment** 相同，但可用于轻松查看哪些路由使用哈希ID（请参见 :doc:`Model </models/model>` 文档）。

.. note:: **{locale}** 不能用作通配符或路由的其他部分，因为它保留供 :doc:`本地化 </outgoing/localization>` 使用。

示例
========

以下是一些基本的路由示例::

	$routes->add('journals', 'App\Blogs');

第一段中包含单词"journals"的URL将被重新映射到 "App\\Blogs" 类和默认方法，该方法通常是 ``index()``::

	$routes->add('blog/joe', 'Blogs::users/34');

包含段"blog/joe"的URL将重新映射到“\\Blogs”类和“users”方法。ID将设置为“34”::

	$routes->add('product/(:any)', 'Catalog::productLookup');

以“product”为第一段的URL，第二部分为第二段的URL将重新映射到“\\Catalog”类和“productLookup”方法::

	$routes->add('product/(:num)', 'Catalog::productLookupByID/$1';

将以“product”作为第一个段的URL，将第二个数字作为第二个段的URL，将重新映射到“\\Catalog”类和“productLookupByID”方法中，并将匹配项作为该方法的变量传入。

.. important:: 尽管 ``add()`` 方法很方便，但建议始终使用基于HTTP动词的路由（如下所述），因为它更安全。由于仅存储与当前请求方法匹配的路由，因此它也会提供轻微的性能提高，从而在尝试查找匹配项时减少了要扫描的路由。

自定义通配符
===================

您可以创建自己的通配符，这些通配符可在路由文件中使用，以完全自定义体验和可读性。

您使用 ``addPlaceholder`` 方法添加新的通配符。第一个参数是用作通配符的字符串。第二个参数是应替换为的正则表达式模式。在添加路由之前，必须先调用此方法::

	$routes->addPlaceholder('uuid', '[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}');
	$routes->add('users/(:uuid)', 'Users::show/$1');

正则表达式
===================

如果愿意，可以使用正则表达式定义路由规则。允许使用任何有效的正则表达式，也可以使用反向引用。

.. important:: 注意：如果使用反向引用，则必须使用dollar语法而不是双反斜杠语法。典型的RegEx路由可能如下所示::

	$routes->add('products/([a-z]+)/(\d+)', 'Products::show/$1/id_$2');

在上面的示例中，类似于 **products/shirts/123** 的URI会调用控制器类 ``Products`` 的 ``show`` 方法，并将原始的第一段和第二段作为参数传递给它。

使用正则表达式，您还可以捕获包含正斜杠（'/'）的分段，该斜杠通常表示多个分段之间的分隔符。

例如，如果用户访问您的Web应用程序的受密码保护的区域，并且您希望能够在他们登录后将其重定向回同一页面，那么您可能会发现此示例很有用::

	$routes->add('login/(.+)', 'Auth::login/$1');

对于那些不了解正则表达式并想了解更多有关正则表达式的人， `regular-expressions.info <https://www.regular-expressions.info/>`_ 可能是一个不错的起点。

.. important:: 注意: 您还可以将通配符与正则表达式混合并匹配。

闭包
========

您可以使用匿名函数或闭包作为路由映射到的目的地。用户访问该URI时将执行此功能。这对于快速执行小任务，甚至只是显示一个简单的视图都很方便::

    $routes->add('feed', function()
    {
        $rss = new RSSFeeder();
        return $rss->feed('general');
    });

映射多个路由
=======================

尽管 ``add()`` 方法易于使用，但使用 ``add()`` 方法同时处理多个路由通常比较麻烦。您可以定义路由数组，然后将其作为第一个参数传递给 ``map()`` 方法，而不是为需要添加的每个路由调用方法::

	$routes = [];
	$routes['product/(:num)']      = 'Catalog::productLookupById';
	$routes['product/(:alphanum)'] = 'Catalog::productLookupByName';

	$collection->map($routes);

重定向路由
==================

任何生存时间足够长的网站都必然会移动页面。您可以使用 ``addRedirect()`` 方法指定应重定向到其他路由的路由。第一个参数是旧路由的URI模式。第二个参数是要重定向到的新URI或命名路由的名称。第三个参数是应该与重定向一起发送的HTTP状态码。默认值是302一个临时重定向，在大多数情况下建议使用::

    $routes->add('users/profile', 'Users::profile', ['as' => 'profile']);

    // 重定向到命名路由
    $routes->addRedirect('users/about', 'profile');
    // 重定向到URI
    $routes->addRedirect('users/about', 'users/profile');

如果在页面加载期间匹配了重定向路由，则在可以加载控制器之前，将立即将用户重定向到新页面。

分组路由
===============

您可以使用 ``group()`` 方法以通用名称将路由分组。组名称成为一个段，该段出现在组内部定义的路由之前。这使您可以减少构建所有共享开头字符串的扩展路由集所需的类型，例如在构建管理区域时::

	$routes->group('admin', function($routes)
	{
		$routes->add('users', 'Admin\Users::index');
		$routes->add('blog', 'Admin\Blog::index');
	});

这将在“users”和“blog” URI的前面加上“admin”，处理诸如 ``/admin/users`` 和 ``/admin/blog`` 的URL。如果需要，可以将组嵌套在组中以进行更精细的组织::

	$routes->group('admin', function($routes)
	{
		$routes->group('users', function($routes)
		{
			$routes->add('list', 'Admin\Users::list');
		});

	});

这将处理位于 ``admin/users/list`` 的URL。

如果您需要将选项分配给组（例如 `命名空间 <#id20>`_），请在回调之前进行操作::

	$routes->group('api', ['namespace' => 'App\API\v1'], function($routes)
	{
		$routes->resource('users');
	});

这将处理到 ``App\API\v1\Users`` 控制器和 ``/api/users`` URI的资源路由。

您也可以对一组路由使用特定的 `过滤器 <filters.html>`_。这将始终在控制器之前或之后运行过滤器。这在身份验证或api日志记录期间特别方便::

    $routes->group('api', ['filter' => 'api-auth'], function($routes)
    {
        $routes->resource('users');
    });

过滤器的值必须与 ``app/Config/Filters.php`` 中定义的别名之一匹配。

环境限制
========================

您可以创建一组仅在特定环境中可见的路由。这样，您可以创建仅开发人员可以在测试或生产服务器上无法访问的本地计算机上使用的工具。这可以通过 ``environment()`` 方法来完成。第一个参数是环境的名称。此闭包中定义的任何路由只能从给定环境访问::

	$routes->environment('development', function($routes)
	{
		$routes->add('builder', 'Tools\Builder::index');
	});

反向路由
===============

反向路由允许您定义链接应到达的控制器和方法以及任何参数，并让路由器查找到它的当前路由。这样就可以更改路由定义，而不必更新应用程序代码。通常在视图中使用它来创建链接。

例如，如果您有要链接到照片库的路由，则可以使用 ``route_to()`` 辅助函数获取应使用的当前路由。第一个参数是完全限定的控制器和方法，由双冒号(::)分隔，就像编写初始路由本身时使用的一样。接下来应传入任何应传递给路由的参数::

	// 路由定义为:
	$routes->add('users/(:id)/gallery(:any)', 'App\Controllers\Galleries::showUserGallery/$1/$2');

	// 生成user ID 15, gallery 12的相对URL
	// 生成: /users/15/gallery/12
	<a href="<?= route_to('App\Controllers\Galleries::showUserGallery', 15, 12) ?>">View Gallery</a>

使用命名路由
==================

您可以命名路由以减少应用程序的脆弱性。命名路由以后可以调用，即使路由定义发生更改，使用该应用程序 ``route_to`` 构建的应用程序中的所有链接仍将起作用，而无需进行任何更改。通过在 ``as`` 选项中输入路由名称来命名路由::

    // 路由定义为:
    $routes->add('users/(:id)/gallery(:any)', 'Galleries::showUserGallery/$1/$2', ['as' => 'user_gallery');

    // 生成user ID 15, gallery 12的相对URL
    // 生成: /users/15/gallery/12
    <a href="<?= route_to('user_gallery', 15, 12) ?>">View Gallery</a>

这还有使视图更具可读性。

在路由中使用 HTTP 动词
==========================

可以使用HTTP动词（请求方法）来定义路由规则。这在构建RESTFUL应用程序时特别有用。您可以使用任何标准的HTTP动词（GET，POST，PUT，DELETE等）。每个动词都有自己可以使用的方法::

	$routes->get('products', 'Product::feature');
	$routes->post('products', 'Product::feature');
	$routes->put('products/(:num)', 'Product::feature');
	$routes->delete('products/(:num)', 'Product::feature');

您可以通过将它们作为数组传递给 ``match`` 方法来提供路由应匹配的多个动词::

	$routes->match(['get', 'put'], 'products', 'Product::feature');

命令行专用的路由
========================

使用 ``cli()`` 方法，您可以创建仅在命令行工作且无法从Web浏览器访问的路由 。这对于构建cronjobs或仅CLI工具非常有用。通过CLI不能访问由任何基于HTTP动词的路由方法创建的任何路由，但 ``any()`` 仍可从命令行使用该方法创建的路由::

	$routes->cli('migrate', 'App\Database::migrate');

全局选项
==============

创建路由的所有方法（add, get, post, `resource <restful.html>`_ 等）都可以采用一系列选项，这些选项可以修改生成的路由或进一步限制它们。``$options`` 数组始终是最后一个参数::

	$routes->add('from', 'to', $options);
	$routes->get('from', 'to', $options);
	$routes->post('from', 'to', $options);
	$routes->put('from', 'to', $options);
	$routes->head('from', 'to', $options);
	$routes->options('from', 'to', $options);
	$routes->delete('from', 'to', $options);
	$routes->patch('from', 'to', $options);
	$routes->match(['get', 'put'], 'from', 'to', $options);
	$routes->resource('photos', $options);
	$routes->map($array, $options);
	$routes->group('name', $options, function());

应用过滤器
----------------

您可以通过提供在控制器之前或之后运行的过滤器来更改特定路由的行为。这在身份验证或api日志记录期间特别方便::

    $routes->add('admin',' AdminController::index', ['filter' => 'admin-auth']);

过滤器的值必须与 ``app/Config/Filters.php`` 中定义的别名之一匹配。您还可以提供要传递给过滤器 ``before()`` 和 ``after()`` 方法的参数::

    $routes->add('users/delete/(:segment)', 'AdminController::index', ['filter' => 'admin-auth:dual,noreturn']);

有关设置过滤器的更多信息，请参见 `控制器过滤器 <filters.html>`_。

指定命名空间
-------------------

尽管默认命名空间将添加到生成的控制器之前（请参见下文），但您也可以使用选项指定要在任何选项数组中使用的其他命名空间 ``namespace``。该值应该是要修改的命名空间::

	// 路由到 \Admin\Users::index()
	$routes->add('admin/users', 'Users::index', ['namespace' => 'Admin']);

新命名空间仅在该调用期间应用于创建单个路由的任何方法，例如get，post等。对于创建多个路由的任何方法，新命名空间将附加到该函数生成的所有路由，或者在这种情况下的 ``group()``，所有路径在闭合时生成。

主机名限制
-----------------

您可以通过传递 ``hostname`` 选项和所需的域以允许其作为选项数组的一部分来限制路由组仅在应用程序的某些域或子域中起作用::

	$collection->get('from', 'to', ['hostname' => 'accounts.example.com']);

此示例仅在域与 "accounts.example.com" 完全匹配时才允许指定的主机工作。在主网站"example.com"下无法正常运行。

子域名限制
-------------------

如果存在 ``subdomain`` 选项，则系统将限制路由仅在该子域上可用。仅当子域是正在查看应用程序的子域时，才会匹配该路由::

	// 限制到 media.example.com
	$routes->add('from', 'to', ['subdomain' => 'media']);

通过将值设置为星号(*)，可以将其限制为任何子域。如果您正在从不存在任何子域的URL进行查看，则将不匹配::

	// 限制到任何子域名
	$routes->add('from', 'to', ['subdomain' => '*']);

.. important:: 该系统并不完美，应在生产中使用之前针对您的特定领域进行测试。大多数域应能正常工作，但某些边缘情况可能会导致误报，特别是在域本身中带有句点（不用于分隔后缀或www）的情况。

偏移匹配的参数
---------------------------------

您可以使用 ``offset`` 选件将路由中匹配的参数偏移任何数值，该值是要偏移的路段数。

在开发以第一个URI段为版本号的API时，这可能是有益的。当第一个参数是语言字符串时，也可以使用它::

	$routes->get('users/(:num)', 'users/show/$1', ['offset' => 1]);

	// 创建:
	$routes['users/(:num)'] = 'users/show/$2';

路由配置选项
============================

RoutesCollection类提供了影响所有路由的几个选项，可以对其进行修改以满足您的应用程序的需求。这些选项位于 `/app/Config/Routes.php` 的顶部。

默认命名空间
-----------------

将控制器与路由匹配时，路由器会将默认命名空间值添加到路由指定的控制器的前面。默认情况下，此值为空，这使每个路由都可以指定完全命名空间的控制器::

    $routes->setDefaultNamespace('');

    // 控制器是 \Users
    $routes->add('users', 'Users::index');

    // 控制器是 \Admin\Users
    $routes->add('users', 'Admin\Users::index');

如果您的控制器没有显式命名空间，则无需更改此名称。如果您为控制器命名空间，则可以更改此值以保存输入::

	$routes->setDefaultNamespace('App');

	// 控制器是 \App\Users
	$routes->add('users', 'Users::index');

	// 控制器是 \App\Admin\Users
	$routes->add('users', 'Admin\Users::index');

默认控制器
------------------

当用户访问您网站的根目录（例如example.com）时，除非该方法明确存在路由，否则要使用的控制器由 ``setDefaultController()`` 方法设置的值确定。默认值 ``Home`` 与控制器 ``/app/Controllers/Home.php`` 匹配::

	// example.com 路由到 app/Controllers/Welcome.php
	$routes->setDefaultController('Welcome');

如果找不到匹配的路由，并且URI指向controllers目录中的目录，也会使用默认控制器。例如，如果用户访问 ``example.com/admin``，则将使用找到的控制器 ``/app/Controllers/admin/Home.php``。

默认方法
--------------

此工作方式与默认控制器设置相似，但是用于确定找到与URI匹配但没有该方法段的控制器时使用的默认方法。默认值为 ``index``::

	$routes->setDefaultMethod('listAll');

在此示例中，如果用户访问 **example.com/products**，并且存在Products控制器，则将执行 ``Products::listAll()`` 方法。

URI(-)转换
--------------------

此选项使您可以在控制器和方法URI段中的连字符（'-'）转换为下划线（'_'），从而在需要时为您节省了其他路由条目。这是必需的，因为破折号不是有效的类或方法名称字符，如果尝试使用它将导致致命错误::

	$routes->setTranslateURIDashes(true);

仅使用定义的路由
-----------------------

当未找到与URI匹配的已定义路由时，系统将尝试将URI与上述控制器和方法进行匹配。通过将 ``setAutoRoute()`` 选项设置为false ，可以禁用此自动匹配，并将路由限制为仅由您定义的路由::

	$routes->setAutoRoute(false);

404 重写
------------

当找不到与当前URI匹配的页面时，系统将显示通用404视图。您可以通过指定 ``set404Override()`` 选项执行的操作来更改此行为。该值可以是有效的类/方法对，就像您在任何路径中显示的一样，也可以是闭包::

    // 将执行 App\Errors 类的 show404 方法
    $routes->set404Override('App\Errors::show404');

    // 将显示自定义视图
    $routes->set404Override(function()
    {
        echo view('my_errors/not_found.html');
    });
