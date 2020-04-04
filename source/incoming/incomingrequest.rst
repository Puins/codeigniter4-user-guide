*********************
IncomingRequest 类
*********************

IncomingRequest类提供来自客户端（如浏览器）的HTTP请求的面向对象表示。 除了下面列出的方法之外，它还扩展并可以访问 :doc:`Request </incoming/request>` 和 :doc:`Message </incoming/message>` 类的所有方法。

.. contents::
    :local:
    :depth: 2

访问请求
--------------

如果当前类是 ``CodeIgniter\Controller`` 类的后代，并且可以作为类属性进行访问，则该请求类的实例已经为您填充::

        <?php namespace App\Controllers;

        use CodeIgniter\Controller;

	class UserController extends Controller
	{
		public function index()
		{
			if ($this->request->isAJAX())
			{
				. . .
			}
		}
	}

如果您不在控制器内，但仍需要访问应用程序的Request对象，则可以通过 :doc:`Services 类 </concepts/services>` 获取它的副本::

	$request = \Config\Services::request();

但是，如果类不是控制器，则最好将请求作为依赖项传递给您，您可以在其中将其另存为类属性::

	<?php
        use CodeIgniter\HTTP\RequestInterface;

	class SomeClass
	{
		protected $request;

		public function __construct(RequestInterface $request)
		{
			$this->request = $request;
		}
	}

	$someClass = new SomeClass(\Config\Services::request());

确定请求类型
---------------

一个请求可以是几种类型，包括AJAX请求或命令行请求。可以使用 ``isAJAX()`` 和 ``isCLI()`` 方法进行检查::

	// 检查 AJAX 请求
	if ($request->isAJAX())
	{
		. . .
	}

	// 检查 CLI 请求
	if ($request->isCLI())
	{
		. . .
	}

.. note:: ``isAJAX()`` 方法取决于 ``X-Requested-With`` 标头，在某些情况下，默认情况下，标头不会在XHR请求中通过JavaScript发送（即，获取）。有关如何避免此问题的信息，请参见 :doc:`AJAX 请求 </general/ajax>` 部分。

您可以使用以下 ``method()`` 方法检查此请求代表的HTTP 方法::

	// 返回 'post'
	$method = $request->getMethod();

默认情况下，该方法以小写字符串返回（即“get”，“post”等）。您可以传递true作为唯一参数来获得大写版本::

	// 返回 'GET'
	$method = $request->getMethod(true);

您还可以使用以下 ``isSecure()`` 方法检查请求是否通过HTTPS连接进行::

	if (! $request->isSecure())
	{
		force_https();
	}

检索输入
---------------

您可以通过Request对象从$_SERVER, $_GET, $_POST, $_ENV, 和 $_SESSION检索输入。数据不会自动过滤，并返回请求中传递的原始输入数据。使用这些方法而不是直接访问它们的主要优点是（$_POST['something']），如果该项目不存在，它们将返回null，并且可以过滤数据。这使您可以方便地使用数据，而不必先测试项目是否存在。换句话说，通常您可以执行以下操作::

	$something = isset($_POST['foo']) ? $_POST['foo'] : NULL;

使用CodeIgniter的内置方法，您可以简单地执行以下操作::

	$something = $request->getVar('foo');

``getVar()`` 方法将从$_REQUEST中获取数据，所以将从$_GET，$_POST或$_COOKIE返回任何数据。尽管这很方便，但是您通常需要使用更具体的方法，例如:

* ``$request->getGet()``
* ``$request->getPost()``
* ``$request->getServer()``
* ``$request->getCookie()``

此外，还有一些实用程序方法可用于从$_GET或$_POST检索信息，同时保持控制所查找顺序的能力:

* ``$request->getPostGet()`` - checks $_POST first, then $_GET
* ``$request->getGetPost()`` - checks $_GET first, then $_POST

**获取JSON数据**

你可以使用 ``getJSON()`` 去获取 php://input 传递的 JSON 流的内容。

.. note::  无法检查传入的数据是否为有效的JSON，仅在知道要使用JSON的情况下才应使用此方法。

::

	$json = $request->getJSON();

默认情况下，这将返回JSON数据中的所有对象作为对象。如果要将其转换为关联数组，请传递 ``true`` 作为第一个参数。

第二和第三参数相匹配到 `json_decode <https://www.php.net/manual/en/function.json-decode.php>`_ 函数的 ``depth`` 与 ``options`` 参数。

**检索原始数据 (PUT, PATCH, DELETE)**

最后，你可以使用 ``getRawInput()`` 去获取 php://input 传递的原始流的内容。::

	$data = $request->getRawInput();

这将检索数据并将其转换为数组。像这样::

	var_dump($request->getRawInput());

	[
		'Param1' => 'Value1',
		'Param2' => 'Value2'
	]

**过滤输入数据**

为了维护应用程序的安全性，您将需要在访问它时过滤所有输入。您可以传递过滤器的类型以用作任何这些方法的最后一个参数。本机 ``filter_var()`` 函数用于过滤。转至PHP手册以获取 `有效过滤器类型 <https://www.php.net/manual/en/filter.filters.php>`_ 的列表。

过滤POST变量如下所示::

	$email = $request->getVar('email', FILTER_SANITIZE_EMAIL);

除 ``getJSON()`` 外的上面提到的所有方法都支持作为最后一个参数传入过滤器类型。

检索标头
---------------

您可以使用 ``getHeaders()`` 方法访问随请求发送的任何标头，该方法返回所有标头的数组，键为标头的名称，值是 ``CodeIgniter\HTTP\Header`` 的一个实例::

	var_dump($request->getHeaders());

	[
		'Host'          => CodeIgniter\HTTP\Header,
		'Cache-Control' => CodeIgniter\HTTP\Header,
		'Accept'        => CodeIgniter\HTTP\Header,
	]

如果只需要一个标头，则可以将名称传递给 ``getHeader()`` 方法。如果存在，这将以不区分大小写的方式获取指定的标头对象。如果没有，它将返回 ``null``::

	// 这些都是等效的
	$host = $request->getHeader('host');
	$host = $request->getHeader('Host');
	$host = $request->getHeader('HOST');

您可以随时使用 ``hasHeader()`` 方法查看该请求中是否存在标头::

	if ($request->hasHeader('DNT'))
	{
		// Don't track something...
	}

如果需要将header的值作为一个字符串，且所有值都在一行上，则可以使用 ``getHeaderLine()`` 方法::

    // Accept-Encoding: gzip, deflate, sdch
    echo 'Accept-Encoding: '.$request->getHeaderLine('accept-encoding');

如果您需要整个标头，并且名称和值在单个字符串中，只需将标头转换为字符串即可::

	echo (string)$header;

请求URL
---------------

您可以通过属性检索一个 :doc:`URI </libraries/uri>` 对象，该对象通过 ``$request->uri`` 属性代表此请求的当前URI。您可以将此对象转换为字符串，以获取当前请求的完整URL::

	$uri = (string)$request->uri;

该对象使您能够完全自行捕获请求的任何部分::

	$uri = $request->uri;

	echo $uri->getScheme();         // http
	echo $uri->getAuthority();      // snoopy:password@example.com:88
	echo $uri->getUserInfo();       // snoopy:password
	echo $uri->getHost();           // example.com
	echo $uri->getPort();           // 88
	echo $uri->getPath();           // /path/to/page
	echo $uri->getQuery();          // foo=bar&bar=baz
	echo $uri->getSegments();       // ['path', 'to', 'page']
	echo $uri->getSegment(1);       // 'path'
	echo $uri->getTotalSegments();  // 3

上传文件
---------------

可以通过 ``$request->getFiles()`` 检索有关所有上传文件的信息，该信息返回一个 :doc:`FileCollection </libraries/uploaded_files>` 实例。这有助于减轻使用上传文件的痛苦，并使用最佳做法来最大程度地减少任何安全风险。

::

	$files = $request->getFiles();

	// 按HTML表单中给定的名称获取文件
	if ($files->hasFile('uploadedFile')
	{
		$file = $files->getFile('uploadedfile');

		// 生成新的安全名称
		$name = $file->getRandomName();

		// 把文件移到新目录
		$file->move('/path/to/dir', $name);

		echo $file->getSize('mb');      // 1.23
		echo $file->getExtension();     // jpg
		echo $file->getType();          // image/jpg
	}

您可以根据HTML文件输入中提供的文件名，检索单独上传的单个文件::

	$file = $request->getFile('uploadedfile');

您可以根据HTML文件输入中提供的文件名，检索作为多文件上传的一部分上传的同名文件数组::

	$files = $request->getFileMultiple('uploadedfile');

内容协商
---------------

您可以轻松地通过 ``negotiate()`` 方法与请求协商内容类型::

	$language    = $request->negotiate('language', ['en-US', 'en-GB', 'fr', 'es-mx']);
	$imageType   = $request->negotiate('media', ['image/png', 'image/jpg']);
	$charset     = $request->negotiate('charset', ['UTF-8', 'UTF-16']);
	$contentType = $request->negotiate('media', ['text/html', 'text/xml']);
	$encoding    = $request->negotiate('encoding', ['gzip', 'compress']);

有关更多详细信息，请参见 :doc:`内容协商 </incoming/content_negotiation>` 页面。

类参考
--------------

.. note:: 除了此处列出的方法之外，此类还继承了 :doc:`Request 类 </incoming/request>` 和 :doc:`Message 类 </incoming/message>` 中的方法。

可用的父类提供的方法是:

* :meth:`CodeIgniter\\HTTP\\Request::getIPAddress`
* :meth:`CodeIgniter\\HTTP\\Request::validIP`
* :meth:`CodeIgniter\\HTTP\\Request::getMethod`
* :meth:`CodeIgniter\\HTTP\\Request::getServer`
* :meth:`CodeIgniter\\HTTP\\Message::getBody`
* :meth:`CodeIgniter\\HTTP\\Message::setBody`
* :meth:`CodeIgniter\\HTTP\\Message::populateHeaders`
* :meth:`CodeIgniter\\HTTP\\Message::headers`
* :meth:`CodeIgniter\\HTTP\\Message::header`
* :meth:`CodeIgniter\\HTTP\\Message::headerLine`
* :meth:`CodeIgniter\\HTTP\\Message::setHeader`
* :meth:`CodeIgniter\\HTTP\\Message::removeHeader`
* :meth:`CodeIgniter\\HTTP\\Message::appendHeader`
* :meth:`CodeIgniter\\HTTP\\Message::protocolVersion`
* :meth:`CodeIgniter\\HTTP\\Message::setProtocolVersion`
* :meth:`CodeIgniter\\HTTP\\Message::negotiateMedia`
* :meth:`CodeIgniter\\HTTP\\Message::negotiateCharset`
* :meth:`CodeIgniter\\HTTP\\Message::negotiateEncoding`
* :meth:`CodeIgniter\\HTTP\\Message::negotiateLanguage`
* :meth:`CodeIgniter\\HTTP\\Message::negotiateLanguage`

.. php:class:: CodeIgniter\\HTTP\\IncomingRequest

	.. php:method:: isCLI()

		:returns: 如果请求是从命令行启动的，则为true，否则为false。
		:rtype: bool

	.. php:method:: isAJAX()

		:returns: 如果请求是AJAX请求，则为true，否则为false。
		:rtype: bool

	.. php:method:: isSecure()

		:returns: 如果请求是HTTPS请求，则为true，否则为false。
		:rtype: bool

	.. php:method:: getVar([$index = null[, $filter = null[, $flags = null]]])

		:param  string  $index: 要查找的变量/键的名称。
		:param  int     $filter: 要应用的过滤器类型。可以在 `filters <https://www.php.net/manual/en/filter.filters.php>`_ 找到过滤器列表。
		:param  int     $flags: 要应用的标志。可以在 `filters flags <https://www.php.net/manual/en/filter.filters.flags.php>`_ 找到标志列表
		:returns:   不传参数会返回 ``$_REQUEST`` 中的所有元素，传参并且参数存在则返回对应的 ``$_REQUEST`` 值，不存在返回 null
		:rtype: mixed|null

		第一个参数将包含您要查找的REQUEST项的名称::

			$request->getVar('some_data');

		如果您要检索的项目不存在，则该方法返回null。

		第二个可选参数使您可以通过PHP的过滤器运行数据。传递所需的过滤器类型作为第二个参数::

			$request->getVar('some_data', FILTER_SANITIZE_STRING);

		要返回所有POST项目的数组，请不带任何参数调用。

		要返回所有POST项目并通过过滤器，请将第一个参数设置为null，同时将第二个参数设置为要使用的过滤器::

			$request->getVar(null, FILTER_SANITIZE_STRING); // 返回所有清理字符串的POST项目

		要返回包含多个POST参数的数组，请将所有必需的键作为数组传递::

			$request->getVar(['field1', 'field2']);

		此处应用了相同的规则，要通过过滤检索参数，请将第二个参数设置为要应用的过滤器类型::

			$request->getVar(['field1', 'field2'], FILTER_SANITIZE_STRING);

	.. php:method:: getGet([$index = null[, $filter = null[, $flags = null]]])

		:param  string  $index: 要查找的变量/键的名称。
		:param  int  $filter: 要应用的过滤器类型。可以在 `filters <https://www.php.net/manual/en/filter.filters.php>`_ 找到过滤器列表。
		:param  int     $flags: 要应用的标志。可以在 `filters flags <https://www.php.net/manual/en/filter.filters.flags.php>`_ 找到标志列表 
		:returns:   不传参数会返回 ``$_GET`` 中的所有元素，传参并且参数存在则返回对应的 ``$_GET`` 值，不存在返回 null
		:rtype: mixed|null

		此方法与 ``getVar()`` 相同，只不过它仅获取GET数据。

	.. php:method:: getPost([$index = null[, $filter = null[, $flags = null]]])

		:param  string  $index: 要查找的变量/键的名称。
		:param  int  $filter: 要应用的过滤器类型。可以在 `filters <https://www.php.net/manual/en/filter.filters.php>`_ 找到过滤器列表。
		:param  int     $flags: 要应用的标志。可以在 `filters flags <https://www.php.net/manual/en/filter.filters.flags.php>`_ 找到标志列表 
		:returns:   不传参数会返回 ``$_POST`` 中的所有元素，传参并且参数存在则返回对应的 ``$_POST`` 值，不存在返回 null
		:rtype: mixed|null

			此方法与 ``getVar()`` 相同，只不过它仅获取POST数据。

	.. php:method:: getPostGet([$index = null[, $filter = null[, $flags = null]]])

		:param  string  $index: 要查找的变量/键的名称。
		:param  int     $filter: 要应用的过滤器类型。可以在 `filters <https://www.php.net/manual/en/filter.filters.php>`_ 找到过滤器列表。
		:param  int     $flags: 要应用的标志。可以在 `filters flags <https://www.php.net/manual/en/filter.filters.flags.php>`_ 找到标志列表
		:returns:   不传参数会返回 ``$_POST`` 中的所有元素，传参并且参数存在则返回对应的 ``$_POST`` 值，不存在返回 null
		:rtype: mixed|null

		此方法的工作方式与 ``getPost()`` 和 ``getGet()`` 几乎相同，仅相结合。它将在POST和GET流中搜索数据，首先在POST中查找，然后在GET中查找::

			$request->getPostGet('field1');

	.. php:method:: getGetPost([$index = null[, $filter = null[, $flags = null]]])

		:param  string  $index: 要查找的变量/键的名称。
		:param  int     $filter: 要应用的过滤器类型。可以在 `filters <https://www.php.net/manual/en/filter.filters.php>`_ 找到过滤器列表。
		:param  int     $flags: 要应用的标志。可以在 `filters flags <https://www.php.net/manual/en/filter.filters.flags.php>`_ 找到标志列表
		:returns:   不传参数会返回 ``$_POST`` 中的所有元素，传参并且参数存在则返回对应的 ``$_POST`` 值，不存在返回 null
		:rtype: mixed|null

		此方法的工作方式与 ``getPost()`` 和 ``getGet()`` 几乎相同，仅相结合。它将在POST和GET流中搜索数据，首先在GET中查找，然后在POST中查找::

			$request->getGetPost('field1');

	.. php:method:: getCookie([$index = null[, $filter = null[, $flags = null]]])

                :noindex:
		:param	mixed	$index: COOKIE名称
		:param  int     $filter: 要应用的过滤器类型。可以在 `filters <https://www.php.net/manual/en/filter.filters.php>`_ 找到过滤器列表。
		:param  int     $flags: 要应用的标志。可以在 `filters flags <https://www.php.net/manual/en/filter.filters.flags.php>`_ 找到标志列表 
		:returns:	不传参数会返回 ``$_COOKIE`` 中的所有元素，传参并且参数存在则返回对应的 ``$_COOKIE`` 值，不存在返回 null
		:rtype:	mixed

		此方法与 ``getPost()`` 和 ``getGet()`` 相同，只不过它仅获取cookie数据。::

			$request->getCookie('some_cookie');
			$request->getCookie('some_cookie', FILTER_SANITIZE_STRING); // 带过滤器

		要返回包含多个Cookie值的数组，请将所有必需的键作为数组传递::

			$request->getCookie(['some_cookie', 'some_cookie2']);

		.. note:: 与 :doc:`Cookie Helper <../helpers/cookie_helper>` 函数中的 :php:func:`get_cookie()` 不同，该方法不会自动添加配置中 ``$config['cookie_prefix']`` 的值。

	.. php:method:: getServer([$index = null[, $filter = null[, $flags = null]]])

		:param	mixed	$index: 值名称
		:param  int     $filter: 要应用的过滤器类型。可以在 `filters <https://www.php.net/manual/en/filter.filters.php>`_ 找到过滤器列表。
		:param  int     $flags: 要应用的标志。可以在 `filters flags <https://www.php.net/manual/en/filter.filters.flags.php>`_ 找到标志列表
		:returns:	$ _SERVER项目值（如果找到），否则为NULL
		:rtype:	mixed

		此方法是与 ``getPost()``, ``getGet()`` 和 ``getCookie()`` 方法相同的，只不过仅用来取得getServer数据（$_SERVER）::

			$request->getServer('some_data');

		要返回多个 ``$_SERVER`` 值的数组，请将所有必需的键作为数组传递。
		::

			$request->getServer(['SERVER_PROTOCOL', 'REQUEST_URI']);

	.. php:method:: getUserAgent([$filter = null])

		:param  int  $filter: 要应用的过滤器类型。可以在 `filters <https://www.php.net/manual/en/filter.filters.php>`_ 找到过滤器列表。  
		:returns:  在SERVER数据中找到的用户代理字符串，如果未找到，则为null。
		:rtype: mixed

		此方法从SERVER数据返回用户代理字符串::

			$request->getUserAgent();
