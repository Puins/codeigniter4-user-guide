=============
HTTP 消息
=============

Message类为HTTP消息的请求和响应所共有的部分提供了接口，包括消息正文，协议版本，用于标头的实用程序以及用于处理内容协商的方法。

此类是 :doc:`Request 类 </incoming/request>` 和 :doc:`Response 类 </outgoing/response>` 都从其扩展的父类。这样，某些方法（例如内容协商方法）可能仅适用于请求或响应，而不适用于其他请求或响应，但是此处将它们包括在内以将标头方法保持在一起。

什么是内容协商？
============================

从根本上讲，内容协商只是HTTP规范的一部分，该协议允许单个资源提供多种类型的内容，从而允许客户端请求最适合他们的数据类型。

一个典型的例子是无法显示PNG文件的浏览器只能请求GIF或JPEG图像。当getServer收到请求时，它将查看客户端正在请求的可用文件类型，并从其支持的图像格式中选择最佳匹配，在这种情况下，很可能选择要返回的JPEG图像。

对于四种类型的数据，可能会发生相同的协商:

* **Media/Document Type** - 可以是图像格式，也可以是HTML与XML或JSON。
* **Character Set** - 应该在其中设置返回文档的字符集。通常为UTF-8。
* **Document Encoding** - 通常在结果上使用的压缩类型。
* **Document Language** - 对于支持多种语言的网站，这有助于确定要返回的语言。

***************
类参考
***************

.. php:class:: CodeIgniter\\HTTP\\Message

	.. php:method:: body()

		:returns: 当前信息正文
		:rtype: string

		返回当前的信息正文（如果已设置）。如果不存在body，则返回null::

			echo $message->body();

	.. php:method:: setBody([$str])

	   :param  string  $str: 消息的主体。
	   :returns: Message实例，允许将方法链接在一起。
	   :rtype: CodeIgniter\\HTTP\\Message 实例。

		设置当前请求的主体。

	.. php:method:: populateHeaders()

		:returns: void

		扫描并解析在SERVER数据中找到的标头，并将其存储以供以后访问。这是使用的 :doc:`IncomingRequest 类 </incoming/incomingrequest>` ，使当前请求的头可用。

		标头是任何以 ``HTTP_`` 开头的SERVER数据，例如HTTP_HOST。每个消息都从其标准的大写和下划线格式转换为ucwords和破折号格式。前一个 ``HTTP_`` 从字符串中删除。如此 ``HTTP_ACCEPT_LANGUAGE`` 成为 ``Accept-Language``。

	.. php:method:: getHeaders()

		:returns: 找到的所有标头的数组。
		:rtype: array

		返回找到或先前设置的所有标头的数组。

	.. php:method:: getHeader([$name[, $filter = null]])

		:param  string  $name: 要检索其值的标头的名称。
		:param  int  $filter: 要应用的过滤器类型。可以在 `filter <https://www.php.net/manual/en/filter.filters.php>`_ 找到过滤器列表。
		:returns: 标头的当前值。如果标头有多个值，它们将作为数组返回。
		:rtype: string|array|null

		允许您检索单个消息头的当前值。``$name`` 是不区分大小写的标头名称。如上所述，在头文件内部进行转换时，可以使用任何类型的大小写来访问头文件::

			// 以下都是一样的:
			$message->getHeader('HOST');
			$message->getHeader('Host');
			$message->getHeader('host');

		如果标头具有多个值，则这些值将作为值的数组返回。您可以使用 ``headerLine()`` 方法以字符串形式检索值::

			echo $message->getHeader('Accept-Language');

			// 输出如下:
			[
				'en',
				'en-US'
			]

		您可以通过将过滤器值作为第二个参数传递来过滤标头::

			$message->getHeader('Document-URI', FILTER_SANITIZE_URL);

	.. php:method:: headerLine($name)

		:param  string $name: 要检索的标头名称。
		:returns: 表示标头值的字符串。
		:rtype: string

		以字符串形式返回标头的值。当标头具有多个值时，此方法使您可以轻松获取标头值的字符串表示形式。这些值适当地结合在一起::

			echo $message->headerLine('Accept-Language');

			// 输出:
			en, en-US

	.. php:method:: setHeader([$name[, $value]])
                :noindex:

		:param string $name: 要为其设置值的标头名称。
		:param mixed  $value: 设置标头的值。
		:returns: 当前消息实例
		:rtype: CodeIgniter\\HTTP\\Message

		设置单个标头的值。``$name`` 是标头的不区分大小写的名称。如果标头在集合中尚不存在，则将创建它。``$value`` 可以是一个字符串或字符串的数组::

			$message->setHeader('Host', 'codeigniter.com');

	.. php:method:: removeHeader([$name])

		:param string $name: 要删除的标头的名称。
		:returns: 当前消息实例
		:rtype: CodeIgniter\\HTTP\\Message

		从消息中删除标头。``$name`` 是标头的不区分大小写的名称::

			$message->remove('Host');

	.. php:method:: appendHeader([$name[, $value]]))

		:param string $name:  要修改的标头的名称
		:param mixed  $value: 要添加到标头的值。
		:returns: 当前消息实例
		:rtype: CodeIgniter\\HTTP\\Message

		向现有标头添加值。标头必须已经是值的数组，而不是单个字符串。如果它是一个字符串，则将抛出LogicException。
		::

			$message->appendHeader('Accept-Language', 'en-US; q=0.8');

	.. php:method:: protocolVersion()

		:returns: 当前的HTTP协议版本
		:rtype: string

		返回消息的当前HTTP协议。如果未设置，将返回 ``null``。可接受的值为 ``1.0`` 和 ``1.1``。

	.. php:method:: setProtocolVersion($version)

		:param string $version: HTTP协议版本
		:returns: 当前消息实例
		:rtype: CodeIgniter\\HTTP\\Message

		设置此消息使用的HTTP协议版本。有效值为： ``1.0`` 或 ``1.1``::

			$message->setProtocolVersion('1.1');

	.. php:method:: negotiateMedia($supported[, $strictMatch=false])

		:param array $supported: 应用程序支持的媒体类型数组
		:param bool $strictMatch: 是否强制完全匹配。
		:returns: 支持的最符合要求的媒体类型。
		:rtype: string

		解析 ``Accept`` 标头，然后与应用程序支持的媒体类型进行比较，以确定最佳匹配。返回适当的媒体类型。第一个参数是应用程序支持的媒体类型的数组，应将其与标头值进行比较::

			$supported = [
				'image/png',
				'image/jpg',
				'image/gif'
			];
			$imageType = $message->negotiateMedia($supported);

		``$supported`` 应该对数组进行结构化，以便应用程序的首选格式是数组中的第一种格式，其余格式按照优先级从高到低的顺序排列。如果标头值和支持的值之间无法匹配，则将返回数组的第一个元素。

		根据RFC，匹配项可以选择返回默认值（如此方法一样）或返回空字符串。如果您需要完全匹配并且想要返回一个空字符串，则将 ``true`` 作为第二个参数传递::

			// 如果不匹配，则返回空数组
			$imageType = $message->negotiateMedia($supported, true);

		匹配过程考虑了RFC的优先级和特殊性。这意味着，更具体的标头值将具有更高的优先级，除非用其他 ``q`` 值修改。有关更多详细信息，请阅读 `RFC的相应部分 <https://tools.ietf.org/html/rfc7231#section-5.3.2>`_ 。

	.. php:method:: negotiateCharset($supported)

		:param array $supported: 应用程序支持的字符集数组。
		:returns: 支持的最符合要求的字符集。
		:rtype: string

		此方法与 ``negotiateMedia()`` 方法相同，只是与 ``Accept-Charset`` 标头字符串匹配::

			$supported = [
				'utf-8',
				'iso-8895-9'
			];
			$charset = $message->negotiateCharset($supported);

		如果找不到匹配项，则系统默认为 ``utf-8``。

	.. php:method:: negotiateEncoding($supported)

		:param array $supported: 应用程序支持的字符编码数组。
		:returns: 支持的最符合要求的字符集。
		:rtype: string

		确定应用程序支持的值和 ``Accept-Encoding`` 标头值之间的最佳匹配。如果找不到匹配项，将返回 ``$supported`` 数组的第一个元素::

			$supported = [
				'gzip',
				'compress'
			];
			$encoding = $message->negotiateEncoding($supported);

	.. php:method:: negotiateLanguage($supported)

		:param array $supported: 应用程序支持的语言数组。
		:returns: 支持的最符合要求的语言。
		:rtype: string

		确定应用程序支持的语言和 ``Accept-Language`` 标头值之间的最佳匹配。如果找不到匹配项，将返回 ``$supported`` 数组的第一个元素::

			$supported = [
				'en',
				'fr',
				'x-pig-latin'
			];
			$language = $message->negotiateLanguage($supported);

		有关语言标签的更多信息，请参见 `RFC 1766 <https://www.ietf.org/rfc/rfc1766.txt>`_。
