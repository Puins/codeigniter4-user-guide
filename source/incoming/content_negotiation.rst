*******************
内容协商
*******************

内容协商是一种根据客户端可以处理的内容以及服务器可以处理的内容确定要返回给客户端的内容类型的方法。这可用于确定客户端是否要返回HTML或JSON，是否应将图像以jpg或png格式返回，支持哪种压缩类型等等。这是通过分析四个不同的标头完成的，每个标头可以支持多个值选项，每个选项都有自己的优先级。尝试手动进行匹配可能会非常具有挑战性。CodeIgniter提供了 ``Negotiator`` 类可以为您处理此问题。

=================
加载类
=================

您可以通过Service类手动加载该类的实例::

	$negotiator = \Config\Services::negotiator();

这将获取当前的请求实例，并将其自动注入到Negotiator类中。

此类无需自己加载。而是可以通过此请求的 ``IncomingRequest`` 实例进行访问。虽然您不能以这种方式直接访问它，但可以通过 ``negotiate()`` 方法轻松访问所有方法::

	$request->negotiate('media', ['foo', 'bar']);

通过这种方式访问​​时，第一个参数是您要查找匹配项的内容的类型，第二个参数是受支持值的数组。

===========
Negotiating
===========

在本节中，我们将讨论可以协商的4种内容，并展示使用上述两种方法访问 ``negotiator`` 类。

Media
=====

首先要看的是处理'media' negotiations。这些由 ``Accept`` 标头提供，并且是可用的最复杂的标头之一。一个常见的示例是客户端告诉服务器它希望数据采用哪种格式。这在API中尤为常见。例如，客户端可能从API端点请求JSON格式的数据::

	GET /foo HTTP/1.1
	Accept: application/json

服务器现在需要提供可以提供的内容类型的列表。在此示例中，API可能能够以原始HTML，JSON或XML的形式返回数据。该列表应按优先顺序提供::

	$supported = [
		'application/json',
		'text/html',
		'application/xml'
	];

	$format = $request->negotiate('media', $supported);
	// 或
	$format = $negotiate->media($supported);

在这种情况下，客户端和服务器双方都可以同意将数据格式化为JSON，以便从 ``negotiation`` 方法返回'json'。默认情况下，如果找不到匹配项，则将返回$ supported数组中的第一个元素。但是，在某些情况下，您可能需要强制格式严格匹配。如果true作为最终值传递，如果找不到匹配项，它将返回一个空字符串::

	$format = $request->negotiate('media', $supported, true);
	// 或
	$format = $negotiate->media($supported, true);

Language
========

另一个常见用法是确定内容应使用的语言。如果您仅运行一个语言站点，那么这显然不会有多大区别，但是任何可以提供多种内容翻译的站点都可以找到此语言。很有用，因为浏览器通常会在 ``Accept-Language`` 标头中发送首选语言::

	GET /foo HTTP/1.1
	Accept-Language: fr; q=1.0, en; q=0.5

在此示例中，浏览器将首选法语，并选择英语。如果您的网站支持英语和德语，您将执行以下操作::

	$supported = [
		'en',
		'de'
	];

	$lang = $request->negotiate('language', $supported);
	// 或
	$lang = $negotiate->language($supported);

在此示例中，“en”将作为当前语言返回。如果未找到匹配项，它将返回$supported数组中的第一个元素，因此该语言应始终是首选语言。

Encoding
========

``Accept-Encoding`` 标头包含字符集的客户端优先接收，并用于指定压缩客户端支持的类型::

	GET /foo HTTP/1.1
	Accept-Encoding: compress, gzip

您的Web服务器将定义您可以使用的压缩类型。有些（例如Apache）仅支持 **gzip**::

	$type = $request->negotiate('encoding', ['gzip']);
	// 或
	$type = $negotiate->encoding(['gzip']);

更多请参看 `Wikipedia <https://en.wikipedia.org/wiki/HTTP_compression>`_。

Character Set
=============

所需的字符集通过 ``Accept-Charset`` 标头传递::

	GET /foo HTTP/1.1
	Accept-Charset: utf-16, utf-8

默认情况下，如果未找到匹配项，则将返回 **utf-8**::

	$charset = $request->negotiate('charset', ['utf-8']);
	// 或
	$charset = $negotiate->charset(['utf-8']);

