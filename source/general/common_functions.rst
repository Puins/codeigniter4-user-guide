##############################
全局函数和常量
##############################

CodeIgniter所使用的功能提供了一些全局定义的函数和变量，您可以随时使用它们。不需要加载任何其他库或辅助函数。

.. contents::
    :local:
    :depth: 2


================
全局函数
================

Service 访问器
=================

.. php:function:: cache ( [$key] )

    :param  string $key: 要从缓存中检索的项目的缓存名称（可选）
    :returns: 缓存对象，或从缓存中检索的项目
    :rtype: mixed

	如果未提供 ``$key``，将返回Cache实例。如果提供了 ``$key``，则将返回当前存储在缓存中的 ``$key`` 的值；如果未找到任何值，则返回null。

    示例::

     	$foo = cache('foo');
    	$cache = cache();

.. php:function:: env ( $key[, $default=null])

	:param string $key: 要检索的环境变量的名称
	:param mixed  $default: 如果未找到任何值，则返回默认值。
	:returns: 环境变量，默认值或null。
	:rtype: mixed

	用于检索事前设置在环境变量中的变量值,若无设置则返回默认值. 若没有找到健值则返回一个布尔值结果（false）。

	当与 ``.env`` 文件结合使用以设置特定于环境本身的值（例如数据库设置，API密钥等）时，特别有用。

.. php:function:: esc ( $data, $context='html' [, $encoding])

	:param   string|array   $data: 要转义的信息。
	:param   string   $context: 转义上下文。默认值为 “html”。
	:param   string   $encoding: 字符串的字符编码。
	:returns: 转义的数据。
	:rtype: mixed

	页面中包含的输出数据, 它在防止 XSS 攻击时很有用。 使用 `Laminas Escaper` 库来处理实际的数据过滤。

	若 `$data` 为字符串, 则简单转义并且返回。 若 `$data` 为数组, 则遍历数组，转义 `key/value` 键值对中的 'value'。

	有效的上下文值：html, js, css, url, attr, raw, null

.. php:function:: helper( $filename )

	:param   string|array  $filename: 要加载的辅助文件的名称或名称数组。

	 加载辅助类文件。

	有关更多信息，请参见 :doc:`helpers` 页面。

.. php:function:: lang($line[, $args[, $locale ]])

	:param string $line: 要检索的文本行
	:param array  $args: 替代占位符的数据数组。
	:param string $locale: 指定要使用的其他语言环境，而不是默认的语言环境。

	根据别名字符串检索特定于语言环境的文件。

	有关更多信息，请参见 :doc:`本地化 </outgoing/localization>` 页面。

.. php:function:: old( $key[, $default = null, [, $escape = 'html' ]] )

	:param string $key: 要检查的原有表单数据的名称。
	:param mixed  $default: 如果 ``$key`` 不存在，则返回的默认值。
	:param mixed  $escape: 一个 `escape <#esc>`_ 的上下文 或者 传值false来禁用该功能。
	:returns: 定义的键的值或默认值。
	:rtype: mixed

	提供一种简单的方法来提交表单以访问"原有的数据"。

	示例::

		// 在控制器中查看表单提交
		if (! $model->save($user))
		{
			// 'withInput'方法意味着"原有的数据"需要被存储。
			return redirect()->back()->withInput();
		}

		// 视图中
		<input type="email" name="email" value="<?= old('email') ?>">
		// 以数组的形式
		<input type="email" name="user[email]" value="<?= old('user.email') ?>">

.. note:: 如果您使用的是 :doc:`form 辅助函数 </helpers/form_helper>`，则此功能是内置的。仅在不使用辅助函数时才需要使用此功能。		

.. php:function:: session( [$key] )

	:param string $key: 在session中查找的健值名称。
	:returns: $key的值或者null，若$key不存在则返回一个session object实例。
	:rtype: mixed

	提供访问会话类和检索存储值的便捷方法。有关更多信息，请参见 :doc:`Sessions </libraries/sessions>` 页面。

.. php:function:: timer( [$name] )

	:param string $name: 基准点的名称。
	:returns: Timer 实例
	:rtype: CodeIgniter\Debug\Timer

	提供快速访问Timer类的便捷方法。您可以将基准点的名称作为唯一参数传递。这将从此时开始计时，或者如果已经运行了具有该名称的计时器，则停止计时。

	示例::

		// 获取一个timer实例
		$timer = timer();

		// 设置计时器的开始与结束点
		timer('controller_loading');    // 开始计时器
		. . .
		timer('controller_loading');    // 停止计时器运行

.. php:function:: view ($name [, $data [, $options ]])

	:param   string   $name: 被加载的文件名
	:param   array    $data: 键值对数组，在视图中能被获取。
	:param   array    $options: 可选的参数数组，用于传递值给 ``rendering`` 类。
	:returns: 视图的输出。
	:rtype: string

	抓取当前的 ``RendererInterface-compatible`` 类（界面渲染类），告诉它展示特定的视图。给控制器、库、路由闭包提供了一种便捷的方法。

	目前，在 ``$options`` 数组里只有一个选项是可用的，`saveData` 指定在同一个请求中，在多次调用 ``view()`` 时数据将连续。默认情况下， 在显示该单一视图文件之后，该视图的数据被丢弃。

	``$option`` 数组主要用于与第三方库整合，例如Twig。

	示例::

		$data = ['user' => $user];

		echo view('user_profile', $data);

	有关更多详细信息，请参见 :doc:`视图 </outgoing/views>` 页面。

其他函数
=======================

.. php:function:: csrf_token ()

	:returns: 当前CSRF令牌的名称。
	:rtype: string

	当前CSRF令牌的名称。

.. php:function:: csrf_header ()

	:returns: 当前CSRF令牌的标头名称。
	:rtype: string

	当前CSRF令牌的标头名称。

.. php:function:: csrf_hash ()

	:returns: 当前CSRF哈希值。
	:rtype: string

	返回当前CSRF哈希值。

.. php:function:: csrf_field ()

	:returns: 带有全部请求CSRF信息的隐藏input的HTML字符串。
	:rtype: string

	返回已插入CSRF信息的隐藏input:

		<input type="hidden" name="{csrf_token}" value="{csrf_hash}">

.. php:function:: csrf_meta ()

	:returns: 具有meta标签的HTML的字符串，其中包含所有必需的CSRF信息。
	:rtype: string

	返回已插入CSRF信息的元标记:

		<meta name="{csrf_header}" content="{csrf_hash}">

.. php:function:: force_https ( $duration = 31536000 [, $request = null [, $response = null]] )

	:param  int  $duration: 浏览器将链接到此资源的链接转换为HTTPS的秒数。
	:param  RequestInterface $request: 当前Request对象的实例。
	:param  ResponseInterface $response: 当前Response对象的实例。

	检查当前是否正在通过HTTPS访问该页面。如果是这样，那么什么也不会发生。如果不是，则通过HTTPS将用户重定向回当前URI。将设置 ``HTTP Strict Transport Security`` 标头，该标头指示现代浏览器将在 ``$duration`` 时间内将所有HTTP请求自动修改为HTTPS请求。

.. php:function:: is_cli ()

	:returns: 如果脚本是从命令行执行的，则为TRUE；否则为FALSE。

.. php:function:: log_message ($level, $message [, $context])

	:param   string   $level: 严重程度
	:param   string   $message: 要记录的消息。
	:param   array    $context: 一个标记和值的联合数组被替换到 $message
	:returns: 如果写入日志成功则为 TRUE ，如果写入日志出现问题则为 FALSE 。
	:rtype: bool

	使用　**app/Config/Logger.php**　中定义的日志处理程序记录消息。

	级别可为以下值: **emergency**, **alert**, **critical**, **error**, **warning**,	**notice**, **info**, 或 **debug**。

	Context 可用于替换 message 字符串中的值。详情参见 :doc:`Logging Information <logging>` 页。

.. php:function:: redirect( string $uri )

	:param  string  $uri: 将用户重定向到的URI。

	返回一个RedirectResponse实例，使您可以轻松创建重定向::

		// 回到上一个页面
		return redirect()->back();

		// 跳转至指定的URI
		return redirect()->to('/admin');

		// 跳转到一个命名路由或反向路由 URI
		return redirect()->route('named_route');

		// 在跳转中保持原有的输入值，使得它们可以被 `old()` 函数调用。
		return redirect()->back()->withInput();

		// 显示一个消息
		return redirect()->back()->with('foo', 'message');

	当将URI传给这个函数时。它将会被作为一个反向路由请求，而不是一个完整的URI，就像使用 ``redirect()->route()`` 一样::

                // 跳转到一个命名路由或反向路由 URI
		return redirect('named_route');

.. php:function:: remove_invisible_characters($str[, $urlEncoded = TRUE])

	:param	string	$str: 输入字符串
	:param	bool	$urlEncoded: 是否移除URL编码字符
	:returns:	已过滤的字符串
	:rtype:	string

	这个函数防止在 ASCII 字符之间插入空字符(NULL)，例如 Java\0script。

	示例::

		remove_invisible_characters('Java\\0script');
		// 返回: 'Javascript'

.. php:function:: route_to ( $method [, ...$params] )

	:param   string   $method: 命名的路由别名，或要匹配的控制器/方法的名称。
	:param   mixed   $params: 一个或多个要传递的参数，以在路由中进行匹配。

	根据命名的路由别名或 ``controller::method`` 组合为您生成一个相对URI。如果提供参数，将执行参数。

	有关完整的详细信息，请参见 :doc:`/incoming/routing` 页面。

.. php:function:: service ( $name [, ...$params] )

	:param   string   $name: 加载的服务名称
	:param   mixed    $params: 一个或多个参数传递到服务方法。
	:returns: 指定的服务类的实例。
	:rtype: mixed

	提供对系统中定义的任何 :doc:`服务 <../concepts/services>` 的轻松访问。这将始终返回该类的共享实例，因此，无论在单个请求中调用该实例多少次，都只会创建一个类实例。

	示例::

		$logger = service('logger');
		$renderer = service('renderer', APPPATH.'views/');

.. php:function:: single_service ( $name [, ...$params] )

	:param   string   $name: 加载的服务名称
	:param   mixed    $params: 一个或多个参数传递到服务方法。
	:returns: 指定的服务类的实例。
	:rtype: mixed

	与上述 **service()** 函数相同，不同之处在于，对该函数的所有调用都将返回该类的新实例，其中 **service** 每次都返回相同的实例。

.. php:function:: stringify_attributes ( $attributes [, $js] )

	:param   mixed    $attributes: 字符串, 键值对数组, 或者对象
	:param   boolean  $js: 若值不需要引用，则为TRUE (Javascript风格)
	:returns: 字符串包含键值对属性, 逗号分隔
	:rtype: string

	辅助函数，用于将字符串，数组或属性对象转换为字符串。

================
全局常量
================

以下常量始终在您的应用程序中的任何位置可用。

核心常量
==============

.. php:const:: APPPATH

	**app** 目录的路径。

.. php:const:: ROOTPATH

	项目根目录的路径。就在上面 ``APPPATH``。

.. php:const:: SYSTEMPATH

	**system** 目录的路径。

.. php:const:: FCPATH

	存放前端控制器的目录的路径。

.. php:const:: WRITEPATH

	**writable** 目录的路径。

Time 常量
==============

.. php:const:: SECOND

	等于 1.

.. php:const:: MINUTE

	等于 60.

.. php:const:: HOUR

	等于 3600.

.. php:const:: DAY

	等于 86400.

.. php:const:: WEEK

	等于 604800.

.. php:const:: MONTH

	等于 2592000.

.. php:const:: YEAR

	等于 31536000.

.. php:const:: DECADE

	等于 315360000.
