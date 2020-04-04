==============
HTTP 响应
==============

Response类使用仅适用于服务器响应调用它的客户端的方法来扩展 :doc:`HTTP消息类 </incoming/message>`。

.. contents::
    :local:
    :depth: 2

使用Response
=========================

为您实例化了一个Response类，并将其传递给您的控制器。可以通过 ``$this->response`` 访问它。很多时候，您不需要直接接触类，因为CodeIgniter会为您发送标头和正文。如果页面成功创建了要求的内容，那就太好了。当出现问题时，或者您需要发回非常特定的状态码，或者甚至利用强大的HTTP缓存时，它就在那里。

设置输出
------------------

当您需要直接设置脚本的输出，而又不依赖CodeIgniter自动获取脚本的输出时，可以使用 ``setBody`` 方法手动进行。这通常与设置响应的状态码结合使用::

	$this->response->setStatusCode(404)
	               ->setBody($body);

原因短语（“确定”，“创建”，“永久移动”）将自动添加，但是您可以添加自定义原因作为 ``setStatusCode()`` 方法的第二个参数::

	$this->response->setStatusCode(404, 'Nope. Not here.');

您可以使用 ``setJSON`` 和 ``setXML`` 方法将数组的格式设置为JSON或XML，并将内容类型标头设置为适当的mime 。通常，您将发送要转换的数据数组::

	$data = [
		'success' => true,
		'id' => 123
	];

	return $this->response->setJSON($data);
	//	或
	return $this->response->setXML($data);

数组标头
---------------

通常，您将需要设置响应头。Response类使使用 ``setHeader()`` 方法非常简单。第一个参数是标头的名称。第二个参数是值，可以是字符串或值的数组，当发送给客户端时，这些值将正确组合。使用这些函数而不是使用本机PHP函数，可以确保不会过早发送任何标头，从而导致错误，并使测试成为可能。

::

	$response->setHeader('Location', 'http://example.com')
	         ->setHeader('WWW-Authenticate', 'Negotiate');

如果标头存在并且有多个值，则可以使用 ``appendHeader()`` 和 ``prependHeader()`` 方法将值分别添加到值列表的末尾。第一个参数是标头的名称，第二个参数是要追加或添加的值。

::

	$response->setHeader('Cache-Control', 'no-cache')
	         ->appendHeader('Cache-Control', 'must-revalidate');

可以使用 ``removeHeader()`` 方法从响应中删除标头，该方法将标头名称(不区分大小写)作为唯一参数。

::

	$response->removeHeader('Location');

强制文件下载
===================

Response类提供了一种将文件发送到客户端的简单方法，提示浏览器将数据下载到您的计算机。这将设置适当的标头以使其实现。

第一个参数是要为 **下载文件命名的名称**，第二个参数是文件数据。

如果将第二个参数设置为NULL并且 ``$filename`` 是现有的可读文件路径，则将改为读取其内容。

如果将第三个参数设置为布尔值TRUE，则将发送实际的文件MIME类型（基于文件扩展名），因此，如果您的浏览器具有该类型的处理程序，则可以使用它。

示例::

	$data = 'Here is some text!';
	$name = 'mytext.txt';
	return $response->download($name, $data);

如果要从服务器下载现有文件，则需要执行以下操作::

	// photo.jpg 的内容将被自动读取
	return $response->download('/path/to/photo.jpg', NULL);

使用可选 ``setFileName()`` 方法更改发送到客户端浏览器的文件名::

	return $response->download('awkwardEncryptedFileName.fakeExt')->setFileName('expenses.csv');

.. note:: 必须返回响应对象，下载才能发送到客户端。这使得响应全部通过后置过滤器被发送到客户端之前。

HTTP 缓存
============

HTTP规范内置了一些工具，可帮助客户端（通常是Web浏览器）缓存结果。正确使用它可以极大地提高应用程序的性能，因为它将告诉客户端它们根本不需要联系getServer，因为一切都没有改变。而且您无法获得更快的速度。

这是通过 ``Cache-Control`` 和 ``ETag`` 标头处理的。本指南不是全面介绍所有缓存标头功能的适当位置，但是您可以通过 `Google Developers <https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching>`_ 很好地了解 。

默认情况下，通过CodeIgniter发送的所有响应对象均已关闭HTTP缓存。对于我们来说，选项和确切的环境变化很大，除了关闭它以外，我们还不能创建一个很好的默认值。不过，通过 ``setCache()`` 方法将Cache值设置为所需的值很简单::

	$options = [
		'max-age'  => 300,
		's-maxage' => 900
		'etag'     => 'abcde',
	];
	$this->response->setCache($options);

数组仅采用键/值对的数组，这些键/值对被分配给 ``Cache-Control`` 标头，但有一些例外。您可以完全根据自己的具体情况自由设置所有选项。虽然大多数选项都应用于 ``Cache-Control`` 标头，但它会智能地处理 ``etag`` 和 ``last-modified`` 选项以适合其相应的标头。

内容安全策略（CSP）
=======================

防范XSS攻击的最佳保护措施之一是在站点上实施内容安全策略。这将迫使您将从站点HTML提取的内容的每个单一来源列入白名单，包括图像，样式表，JavaScript文件等。浏览器将拒绝不符合白名单的来源的内容。此白名单是在响应的 ``Content-Security-Policy`` 标头中创建的，并具有许多不同的配置方式。

这听起来很复杂，并且在某些站点上肯定是具有挑战性的。但是，对于许多简单站点而言，所有内容都由同一域（http://example.com）提供，因此集成起来非常简单。

由于这是一个复杂的主题，因此本用户指南不会涵盖所有详细信息。有关更多信息，您应该访问以下站点:

* `内容安全政策主站点 <https://content-security-policy.com/>`_
* `W3C规格 <https://www.w3.org/TR/CSP>`_
* `HTML5Rocks上的介绍 <https://www.html5rocks.com/en/tutorials/security/content-security-policy/>`_
* `SitePoint上的文章 <https://www.sitepoint.com/improving-web-security-with-the-content-security-policy/>`_

启用CSP
--------------

默认情况下，对此功能的支持为关闭。要在您的应用程序中启用支持，请编辑在 **app/Config/App.php** 中的 ``$CSPEnabled`` 值::

	public $CSPEnabled = true;

启用后，响应对象将包含 ``CodeIgniter\HTTP\ContentSecurityPolicy`` 的实例。在 **app/Config/ContentSecurityPolicy.php** 中设置的值将应用于该实例，并且如果在运行时不需要任何更改，那么将发送格式正确的标头，您已完成所有工作。

启用CSP后，会将两个标头行添加到HTTP响应中：Content-Security-Policy标头，其中包含用于标识在不同上下文中明确允许的内容类型或来源的策略，以及Content-Security-Policy-Report-Only标头，标识允许的内容类型或来源，但也将报告给您选择的目的地。

我们的实现提供了默认的处理方式，可以通过 ``reportOnly()`` 方法进行更改。如下所示，将其他条目添加到CSP指令后，会将其添加到适合于阻止或阻止的CSP标头中。通过为添加方法调用提供可选的第二个参数，可以在每次调用的基础上覆盖该参数。

运行时配置
---------------------

如果您的应用程序需要在运行时进行更改，则可以通过 ``$response->CSP`` 访问实例。该类包含许多方法，这些方法可以很清晰地映射到您需要设置的相应标头值。下面显示的示例具有不同的参数组合，尽管它们都接受指令名称或它们的数组。::

        // 指定默认的指令处理
	$response->CSP->reportOnly(false);

        // 如果没有提供指令，则指定要使用的原点
	$response->CSP->setDefaultSrc('cdn.example.com');
        // 指定"report-only"报告发送到的URL
	$response->CSP->setReportURI('http://example.com/csp/reports');
        // 指定将HTTP请求升级到HTTPS
	$response->CSP->upgradeInsecureRequests(true);

        // 向CSP指令添加类型或起源
        // 假设默认处理是阻止而不是仅报告
	$response->CSP->addBaseURI('example.com', true); // 仅报告
	$response->CSP->addChildSrc('https://youtube.com'); // 受阻
	$response->CSP->addConnectSrc('https://*.facebook.com', false); // 受阻
	$response->CSP->addFontSrc('fonts.example.com');
	$response->CSP->addFormAction('self');
	$response->CSP->addFrameAncestor('none', true); // 报告这个
	$response->CSP->addImageSrc('cdn.example.com');
	$response->CSP->addMediaSrc('cdn.example.com');
	$response->CSP->addManifestSrc('cdn.example.com');
	$response->CSP->addObjectSrc('cdn.example.com', false); // 从这里拒绝
	$response->CSP->addPluginType('application/pdf', false); // 从媒体类型拒绝
	$response->CSP->addScriptSrc('scripts.example.com', true); // 允许但是从这里报告请求
	$response->CSP->addStyleSrc('css.example.com');
	$response->CSP->addSandbox(['allow-forms', 'allow-scripts']);

每个"add"方法的第一个参数是适当的字符串值或它们的数组。

``reportOnly`` 方法允许您为后续来源指定默认的报告处理方式，除非被覆盖。例如，您可以指定允许使用youtube.com，然后提供一些允许但已报告的来源::

    $response->addChildSrc('https://youtube.com'); // 允许
    $response->reportOnly(true);
    $response->addChildSrc('https://metube.com'); // 允许但已报告
    $response->addChildSrc('https://ourtube.com',false); // 允许

内联内容
--------------

可以将网站设置为甚至不保护其自身页面上的内联脚本和样式，因为这可能是用户生成内容的结果。为了防止这种情况，CSP允许您在<style>和<script>标记中指定一个随机数 ，并将这些值添加到响应的标头中。这是在现实生活中很难处理的，并且在动态生成时最安全。为简单起见，您可以在标记中包含 ``{csp-style-nonce}`` 或 ``{csp-script-nonce}`` 占位符，它将自动为您处理::

	// 原版的
	<script {csp-script-nonce}>
	    console.log("Script won't run as it doesn't contain a nonce attribute");
	</script>

	// 成为
	<script nonce="Eskdikejidojdk978Ad8jf">
	    console.log("Script won't run as it doesn't contain a nonce attribute");
	</script>

	// 或
	<style {csp-style-nonce}>
		. . .
	</style>

类参考
=========================

.. note:: 除了此处列出的方法之外，此类还从 :doc:`Message 类 </incoming/message>` 继承这些方法 。

父类提供的可用方法有:

* :meth:`CodeIgniter\\HTTP\\Message::body`
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

.. php:class:: CodeIgniter\\HTTP\\Response

	.. php:method:: getStatusCode()

		:returns: 此响应的当前HTTP状态码
		:rtype: int

		返回此响应的当前状态码。如果未设置状态码，则将引发BadMethodCallException::

			echo $response->getStatusCode();

	.. php:method:: setStatusCode($code[, $reason=''])

		:param int $code: HTTP状态码
		:param string $reason: 可选原因短语。
		:returns: 当前的Response实例。
		:rtype: CodeIgniter\\HTTP\\Response

		设置应与此响应一起发送的HTTP状态码::

		    $response->setStatusCode(404);

		原因短语将根据官方列表自动生成。如果需要为自定义状态码设置自己的名称，则可以将原因短语作为第二个参数传递::

			$response->setStatusCode(230, "Tardis initiated");

	.. php:method:: getReason()

		:returns: 当前原因短语。
		:rtype: string

		返回此响应的当前状态码。如果未设置状态，将返回一个空字符串::

			echo $response->getReason();

	.. php:method:: setDate($date)

		:param DateTime $date: 具有为此响应设置时间的DateTime实例。
		:returns: 当前的Response实例。
		:rtype: CodeIgniter\HTTP\Response

		设置用于此响应的日期。 ``$date`` 参数必须是一个 ``DateTime`` 实例::

			$date = DateTime::createFromFormat('j-M-Y', '15-Feb-2016');
			$response->setDate($date);

	.. php:method:: setContentType($mime[, $charset='UTF-8'])

		:param string $mime: 此响应表示的内容类型。
		:param string $charset: 此响应使用的字符集。
		:returns: 当前的Response实例。
		:rtype: CodeIgniter\HTTP\Response

		设置此响应表示的内容类型::

			$response->setContentType('text/plain');
			$response->setContentType('text/html');
			$response->setContentType('application/json');

		默认情况下，该方法将字符集设置为 ``UTF-8``。如果需要更改此设置，则可以将字符集作为第二个参数传递::

			$response->setContentType('text/plain', 'x-pig-latin');

	.. php:method:: noCache()

		:returns: 当前的Response实例。
		:rtype: CodeIgniter\HTTP\Response

		设置 ``Cache-Control`` 标头以关闭所有HTTP缓存。这是所有响应消息的默认设置::

		    $response->noCache();

		    // 设置以下标头:
		    Cache-Control: no-store, max-age=0, no-cache

	.. php:method:: setCache($options)

		:param array $options: 键/值缓存控制设置的数组
		:returns: 当前的Response实例。
		:rtype: CodeIgniter\HTTP\Response

		设置 ``Cache-Control`` 标头，包括 ``ETags`` 和 ``Last-Modified``。典型的键有:

		* etag
		* last-modified
		* max-age
		* s-maxage
		* private
		* public
		* must-revalidate
		* proxy-revalidate
		* no-transform

		传递上次修改的选项时，它可以是日期字符串或DateTime对象。

	.. php:method:: setLastModified($date)

		:param string|DateTime $date: 设置Last-Modified标头的日期
		:returns: 当前的Response实例。
		:rtype: CodeIgniter\HTTP\Response

		设置 ``Last-Modified`` 标头。``$date`` 对象可以是字符串或 ``DateTime`` 实例::

			$response->setLastModified(date('D, d M Y H:i:s'));
			$response->setLastModified(DateTime::createFromFormat('u', $time));

	.. php:method:: send()
                :noindex:

		:returns: 当前的Response实例。
		:rtype: CodeIgniter\HTTP\Response

		告诉响应将所有内容发送回客户端。这将首先发送标头，然后发送响应正文。对于主应用程序响应，您无需调用它，因为它由CodeIgniter自动处理。

	.. php:method:: setCookie($name = ''[, $value = ''[, $expire = ''[, $domain = ''[, $path = '/'[, $prefix = ''[, $secure = FALSE[, $httponly = FALSE]]]]]]])

		:param	mixed	$name: Cookie名称或参数数组
		:param	string	$value: Cookie值
		:param	int	$expire: Cookie的有效时间，以秒为单位
		:param	string	$domain: Cookie域
		:param	string	$path: Cookie路径
		:param	string	$prefix: Cookie名称前缀
		:param	bool	$secure: 是否仅通过HTTPS传输cookie
		:param	bool	$httponly: 是否仅使cookie可访问HTTP请求（无JavaScript）
		:rtype:	void

		设置一个cookie，其中包含您指定的值。有两种方法可以将信息传递给此方法，以便可以设置cookie：数组方法和离散参数:

		**数组方法**

		使用此方法，将关联数组作为第一个参数传递::

			$cookie = [
				'name'   => 'The Cookie Name',
				'value'  => 'The Value',
				'expire' => '86500',
				'domain' => '.some-domain.com',
				'path'   => '/',
				'prefix' => 'myprefix_',
				'secure' => TRUE,
                                'httponly' => FALSE
			];

			$response->setCookie($cookie);

		**笔记**

		只需要名称和值。要删除Cookie，请将其设置为空白。

		到期时间以 **秒** 为单位设置，它将添加到当前时间。不包括时间，而是仅包括从 **现在** 开始您希望cookie有效的秒数。如果将到期时间设置为零，则cookie仅在打开浏览器时才持续存在。

		对于网站范围的Cookie，无论您如何请求您的网站，都请以句点开头将URL添加到 **域** 中，例如：.your-domain.com

		通常不需要该路径，因为该方法设置了根路径。

		仅当需要避免与服务器的其他同名cookie发生名称冲突时，才需要使用前缀。

		仅当您想通过将其设置为TRUE使其成为安全cookie时，才需要安全布尔值。

		**离散参数**

		如果愿意，可以通过使用各个参数传递数据来设置Cookie::

			$response->setCookie($name, $value, $expire, $domain, $path, $prefix, $secure, $httponly);

	.. php:method:: deleteCookie($name = ''[, $domain = ''[, $path = '/'[, $prefix = '']]])

		:param	mixed	$name: Cookie名称或参数数组
		:param	string	$domain: Cookie域
		:param	string	$path: Cookie路径
		:param	string	$prefix: Cookie名称前缀
		:rtype:	void

		将有效期设为空白，以删除现有的Cookie。

		**笔记**

		仅名称是必需的。

		仅当需要避免与服务器的其他同名cookie发生名称冲突时，才需要使用前缀。

		如果仅删除子集cookie，请提供前缀。如果仅删除域cookie，请提供一个域名。如果仅删除路径cookie，请提供路径名。

		如果任何可选参数为空，则所有适用的同名Cookie都将被删除。

		示例::

			$response->deleteCookie($name);

	.. php:method:: hasCookie($name = ''[, $value = null[, $prefix = '']])

		:param	mixed	$name: Cookie名称或参数数组
		:param	string	$value: Cookie值
		:param	string	$prefix: Cookie名称前缀
		:rtype:	boolean

		检查响应是否具有指定的cookie。

		**笔记**

		仅名称是必需的。如果指定了前缀，前缀将被添加在cookie名称之前。

		如果没有给出值，则该方法仅检查命名cookie的存在。如果提供了一个值，则该方法将检查cookie是否存在以及它是否具有规定的值。

		示例::

			if ($response->hasCookie($name)) ...

	.. php:method:: getCookie($name = ''[, $prefix = ''])
                :noindex:

		:param	mixed	$name: Cookie name
		:param	string	$prefix: Cookie名称前缀
		:rtype:	boolean

		返回命名的cookie（如果找到），或者返回null。

		如果没有给出名称，则返回cookie数组。

		每个cookie作为关联数组返回。

		示例::

			$cookie = $response->getCookie($name);
