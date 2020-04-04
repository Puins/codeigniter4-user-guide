#################
CURLRequest 类
#################
 
 ``CURLRequest`` 类是基于 `CURL` 的轻量级HTTP客户端，可让您与其他网站和服务器进行对话。它可用于获取Google搜索的内容，检索网页或图像，或与API通信等等。

.. contents::
    :local:
    :depth: 2

这个类是根据 `Guzzle HTTP 客户端 <http://docs.guzzlephp.org/en/latest/>`_ 库（它是应用最广泛的类库之一）建模的。在可能的情况下，语法保持不变，以便您需要比这个库提供的功能强大一点的东西，就可以挪过去用几乎没什么变化的 `Guzzle`。

.. note:: 此类要求安装 `cURL 库 <https://www.php.net/manual/en/book.curl.php>`_ 。在你的PHP版本中。这是一个非常常见的库，通常是可用的，但并非所有主机都将提供它，因此请检查您的主机以确认是否遇到问题。

*******************
加载类库
*******************

该库可以手动加载，也可以通过 :doc:`Services class </concepts/services>`。

加载 `Services` 类，调用 ``curlrequest()`` 方法::

	$client = \Config\Services::curlrequest();

您可以传入默认选项数组作为第一个参数，以修改cURL处理请求的方式。这些选项将在本文档后面介绍::

	$options = [
		'base_uri' => 'http://example.com/api/v1/',
		'timeout'  => 3
	];
	$client = \Config\Services::curlrequest($options);

手动创建类时，您需要传入一些依赖关系。第一个参数是 ``Config\App`` 类的实例。 第二个参数是URI实例。 第三参数是一个Response对象。 第四个参数是可选的 ``$options`` 数组::

	$client = new \CodeIgniter\HTTP\CURLRequest(
		new \Config\App(),
		new \CodeIgniter\HTTP\URI(),
		new \CodeIgniter\HTTP\Response(new \Config\App()),
		$options
	);

************************
使用类库
************************

处理CURL请求只是创建请求并获取 :doc:`Response object </outgoing/response>` 返回。它是用来处理通讯的。之后你完全可以控制信息的处理方式。

发出 Requests
===============

大多数通信是通过 ``request()`` 方法完成的，该方法触发请求，然后返回一个响应实例。这将以HTTP方法、url和选项数组作为参数。

::

	$client = \Config\Services::curlrequest();

	$response = $client->request('GET', 'https://api.github.com/user', [
		'auth' => ['user', 'pass']
	]);

由于响应是 ``CodeIgniter\HTTP\Response`` 的实例，因此您可以使用所有常规信息::

	echo $response->getStatusCode();
	echo $response->getBody();
	echo $response->getHeader('Content-Type');
	$language = $response->negotiateLanguage(['en', 'fr']);

虽然 ``request()`` 方法是最灵活的，但您也可以使用以下快捷方法。 他们每个都将URL作为第一个参数，并将选项数组作为第二个参数::

* $client->get('http://example.com');
* $client->delete('http://example.com');
* $client->head('http://example.com');
* $client->options('http://example.com');
* $client->patch('http://example.com');
* $client->put('http://example.com');
* $client->post('http://example.com');

基本 URI
--------

可以在类实例化期间将 ``base_uri`` 设置为选项之一。 这使您能够设置基本URI，然后使用相对URL向该客户端发出所有请求。这在使用APIs时特别方便::

	$client = \Config\Services::curlrequest([
		'base_uri' => 'https://example.com/api/v1/'
	]);

	// GET http:example.com/api/v1/photos
	$client->get('photos');

	// GET http:example.com/api/v1/photos/13
	$client->delete('photos/13');

当向 ``request()`` 方法或任何快捷方式方法提供相对URI时，它将根据 `RFC 2986, 第2节 <https://tools.ietf.org/html/rfc3986#section-5.2>`_ 组合 。为了节省你的时间，这里有一些如何解决组合的示例。

	=====================   ================   ========================
	base_uri                URI                结果
	=====================   ================   ========================
	`http://foo.com`        /bar               `http://foo.com/bar`
	`http://foo.com/foo`    /bar               `http://foo.com/bar`
	`http://foo.com/foo`    bar                `http://foo.com/bar`
	`http://foo.com/foo/`   bar                `http://foo.com/foo/bar`
	`http://foo.com`        `http://baz.com`   `http://baz.com`
	`http://foo.com/?bar`   bar                `http://foo.com/bar`
	=====================   ================   ========================

使用 Responses
===============

每个 ``request()`` 调用都会返回一个Response对象，其中包含很多有用的信息和一些有用的方法。最常用的方法使您可以确定响应本身。

您可以获取响应的状态码和原因::

	$code   = $response->getStatusCode();    // 200
	$reason = $response->getReason();      // OK

您可以从响应中检索标头::

	// 获取标头行
	echo $response->getHeaderLine('Content-Type');

	// 获取所有标头
	foreach ($response->getHeaders() as $name => $value)
	{
		echo $name .': '. $response->getHeaderLine($name) ."\n";
	}

可以使用 ``getBody()`` 方法检索body::

	$body = $response->getBody();

body是远程 `getServer` 提供的原始body。如果内容类型需要格式化，则需要确保您的脚本能够处理::

	if (strpos($response->getHeader('content-type'), 'application/json') !== false)
	{
		$body = json_decode($body);
	}

***************
Request 选项
***************

本节介绍可以传递给构造函数的所有可用选项，即 ``request()`` 方法或任何快捷方式。

allow_redirects
===============

默认情况下， `cURL` 将遵循远程服务器发送回来的所有 ``Location:`` 头。 ``allow_redirects`` 选项允许您修改其工作方式。

如果将该值设置为 ``false``，则它根本不会遵循任何重定向::

	$client->request('GET', 'http://example.com', ['allow_redirects' => false]);

将其设置为 ``true`` 会将默认设置应用于请求::

	$client->request('GET', 'http://example.com', ['allow_redirects' => true]);

	// 设置以下默认值:
	'max'       => 5, // 停止前要遵循的最大重定向数
	'strict'    => true, // 确保POST请求通过重定向保留POST请求
	'protocols' => ['http', 'https'] // 限制重定向到一个或多个协议

您可以传入数组作为 ``allow_redirects`` 选项的值，以指定新设置来代替默认设置::

	$client->request('GET', 'http://example.com', ['allow_redirects' => [
		'max'       => 10,
		'protocols' => ['https'] // Force HTTPS domains only.
	]]);

.. note:: 当PHP处于 `safe_mode` 或启用了 `open_basedir` 时，以下重定向不起作用。

auth
====

允许您为 `HTTP Basic <https://www.ietf.org/rfc/rfc2069.txt>`_ 和 `Digest <https://www.ietf.org/rfc/rfc2069.txt>`_ 提供身份验证的详细信息。你的脚本可能需要额外的支持 `Digest` 身份验证-这只是为您传递用户名和密码。值必须是数组，其中第一个元素是用户名，第二个是密码。第三个参数应该是要使用的身份验证类型，可以是 ``basic`` 或 ``digest`` ::

	$client->request('GET', 'http://example.com', ['auth' => ['username', 'password', 'digest']]);

body
====

有两种方法可以将请求主体设置为支持它们的请求类型， 像 PUT, 或者 POST。第一种方法是使用 ``setBody()`` 方法::

	$client->setBody($body)
	       ->request('put', 'http://example.com');

第二种方法是传入一个 ``body`` 选项。提供此选项是为了保持Guzzle API的兼容性，功能与上一个示例完全相同。值必须是字符串::

	$client->request('put', 'http://example.com', ['body' => $body]);

cert
====

要指定PEM格式的客户端证书的位置，将具有文件完整路径的字符串作为 ``cert`` 选项传递。 如果需要密码，则将值设置为数组的第一个元素作为证书的路径，第二个作为密码::

    $client->request('get', '/', ['cert' => ['/path/getServer.pem', 'password']);

connect_timeout
===============

默认情况下，CodeIgniter不会限制cURL尝试连接到网站。 如果你需要修改此值，您可以通过使用 ``connect_timeout`` 选项传递以秒为单位的时间来实现。您可以传递0以无限期等待::

	$response->request('GET', 'http://example.com', ['connect_timeout' => 0]);

cookie
======

这指定了 `CURL` 从中读取 **Cookie** 值和将 **Cookie** 值保存到文件时应使用的文件名。 这是使用 **CURL_COOKIEJAR** 和 **CURL_COOKIEFILE** 选项完成的。
示例::

	$response->request('GET', 'http://example.com', ['cookie' => WRITEPATH . 'CookieSaver.txt']);

debug
=====

当 ``debug`` 被设置为 ``true`` 传递时，这将使其他调试能够在调试期间回显到 `STDERR` 脚本执行。 这是通过传递 ``CURLOPT_VERBOSE`` 并回显输出来完成的。 因此，当您通过 ``spark serve`` 运行内置服务器，您将在控制台中看到输出。 否则，输出将被写入服务器的错误日志。

	$response->request('GET', 'http://example.com', ['debug' => true]);

可以将文件名作为debug的值传递，以便将输出写入文件::

	$response->request('GET', 'http://example.com', ['debug' => '/usr/local/curl_log.txt']);

delay
=====

允许您在发送请求之前延迟几毫秒::

	// 延迟2秒
	$response->request('GET', 'http://example.com', ['delay' => 2000]);

form_params
===========

您可以通过在 ``form_params`` 选项中设置关联数组发送 ``application/x-www-form-urlencoded`` POST请求中的表单数据。 如果 ``Content-Type`` 标头尚未设置的话，将会设置为 ``application/x-www-form-urlencoded`` ::

	$client->request('POST', '/post', [
		'form_params' => [
			'foo' => 'bar',
			'baz' => ['hi', 'there']
		]
	]);

.. note:: ``form_params`` 不能与 ``multipart`` 选项一起使用。你只能使用其中一个。使用 ``form_params`` 用于 ``application/x-www-form-urlencoded`` 请求， ``multipart`` 用于 ``multipart/form-data`` 请求。

headers
=======

虽然您可以使用 ``setHeader()`` 方法设置此请求所需的任何标头，但是您也可以传递关联的标头数组作为一个选项。 每个键是标头的名称，每个值是一个字符串或字符串数组表示标头字段值::

	$client->request('get', '/', [
		'headers' => [
			'User-Agent' => 'testing/1.0',
			'Accept'     => 'application/json',
			'X-Foo'      => ['Bar', 'Baz']
		]
	]);

如果将标头传递到构造函数中，则将它们视为默认值，以后任何情况都将覆盖它们进一步的标头数组或对 ``setHeader()`` 的调用。

http_errors
===========

默认情况下，如果返回的HTTP代码大于或等于400， `CURLRequest` 将失败。可以将 ``http_errors`` 设置为 ``false`` 以返回内容::

    $client->request('GET', '/status/500');
    // 将会彻底失败

    $res = $client->request('GET', '/status/500', ['http_errors' => false]);
    echo $res->getStatusCode();
    // 500

json
====

 ``json``  选项用于轻松上传JSON编码的数据作为请求的主体。 `Content-Type` 标头添加了 ``application/json`` 的内容，覆盖了可能已经设置的任何 `Content-Type`。 提供给此选项的可以是 ``json_encode()`` 接受的任何值::

	$response = $client->request('PUT', '/put', ['json' => ['foo' => 'bar']]);

.. note:: 此选项不允许对 ``json_encode()`` 函数或 `Content-Type` 进行任何自定义标头。 如果需要此功能，则需要手动编码数据，并将其通过 `CURLRequest类` 的 ``setBody()`` 方法，并使用 ``setHeader()`` 方法设置 `Content-Type` 标头。

multipart
=========

当您需要通过POST请求发送文件和其他数据时，可以使用 ``multipart`` 选项，以及 `CURLFile 类 <https://www.php.net/manual/en/class.curlfile.php>`_。值应为关联数组发送的POST数据。 为了更安全地使用，传统的上传文件方法是在文件名前添加一个已被禁用的 `@` 前缀。您要发送的任何文件都必须作为CURLFile的实例传递::

	$post_data = [
		'foo'      => 'bar',
		'userfile' => new \CURLFile('/path/to/file.txt')
	];

.. note:: ``multipart`` 不能与 ``form_params`` 选项一起使用。你只能使用其中一个。使用 ``form_params`` 用于 ``application/x-www-form-urlencoded`` 请求， ``multipart`` 用于 ``multipart/form-data`` 请求。

query
=====

通过将关联数组作为 ``query`` 选项传递，可以将要发送的数据作为查询字符串变量传递::

	// 向...发送一个 GET 请求 /get?foo=bar
	$client->request('GET', '/get', ['query' => ['foo' => 'bar']]);

timeout
=======

默认情况下， **cURL** 函数可以运行多长时间，没有时间限制。您可以用 ``timeout`` 修改这个选项。该值应为要执行函数的秒数。使用0无限期等待::

	$response->request('GET', 'http://example.com', ['timeout' => 5]);

verify
======

此选项描述SSL证书验证行为。 如果 ``verify`` 选项为 ``true``，则启用SSL证书验证，并使用操作系统提供的默认CA包。 如果设置为 ``false`` 将禁用证书验证（这是不安全的，并且允许中间人攻击！）。 你可以设置到包含CA捆绑包路径的字符串，以启用使用自定义证书的验证。默认值是true::

	// 使用系统的CA包（这是默认设置）
	$client->request('GET', '/', ['verify' => true]);

	// 使用磁盘上自定义的SSL证书。
	$client->request('GET', '/', ['verify' => '/path/to/cert.pem']);

	// 完全禁用验证。 （不安全！）
	$client->request('GET', '/', ['verify' => false]);

version
=======

要设置使用的HTTP协议，可以传递带有版本号的字符串或float（通常为 1.0 或 1.1，当前不支持2.0。）::

	// 强制 HTTP/1.0
	$client->request('GET', '/', ['version' => 1.0]);
