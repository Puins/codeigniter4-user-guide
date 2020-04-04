*****************
使用 URI 类
*****************

CodeIgniter提供了一个在您的应用程序中使用URI的面向对象的解决方案。使用此方法可以轻松地确保结构始终正确，无论URI可能有多复杂，也可以在现有的URI中添加相对URI并安全正确地对其进行解析。

.. contents::
    :local:
    :depth: 2

======================
创建URI实例
======================

创建URI实例就像创建新的类实例一样简单::

	$uri = new \CodeIgniter\HTTP\URI();

另外，您可以使用 ``service()`` 函数为您返回实例::

	$uri = service('uri');

创建新实例时，可以在构造函数中传递完整或部分URL，它将被解析为相应的部分::

	$uri = new \CodeIgniter\HTTP\URI('http://www.example.com/some/path');
	$uri = service('uri', 'http://www.example.com/some/path');

当前URI
---------------

很多时候，您真正想要的只是一个代表此请求当前URL的对象。可以通过两种不同的方式进行访问。首先是直接从当前请求对象中获取它。假设您在继承 ``CodeIgniter\Controller`` 的控制器中，则可以像下面这样::

	$uri = $this->request->uri;

其次，您可以使用 **url_helper** 中可用的功能之一::

	$uri = current_url(true);

您必须 ``true`` 作为第一个参数传递，否则，它将返回当前URL的字符串表示形式。

===========
URI字符串
===========

很多时候，您真正想要的只是获取URI的字符串表示形式。只需将URI转换为字符串即可轻松实现::

	$uri = current_url(true);
	echo (string)$uri;  // http://example.com

如果您知道URI的各个部分，并且只想确保其格式正确，则可以使用URI类的静态 ``createURIString()`` 方法生成一个字符串::

	$uriString = URI::createURIString($scheme, $authority, $path, $query, $fragment);

	// 创建: http://exmample.com/some/path?foo=bar#first-heading
	echo URI::createURIString('http', 'example.com', 'some/path', 'foo=bar', 'first-heading');

=============
URI部分
=============

拥有URI实例后，您可以设置或检索URI的各个部分。本节将提供有关这些部分是什么以及如何使用它们的详细信息。

Scheme
------

Scheme 通常为“http”或“https”，但支持任何方案，包括“file”，“mailto”等。

::

    $uri = new \CodeIgniter\HTTP\URI('http://www.example.com/some/path');

    echo $uri->getScheme(); // 'http'
    $uri->setScheme('https');

Authority
---------

许多URI包含几个元素，这些元素统称为'authority'。这包括任何用户信息，主机和端口号。您可以使用 ``getAuthority()`` 方法将所有这些片段作为单个字符串检索，也可以操纵各个部分。

::

	$uri = new \CodeIgniter\HTTP\URI('ftp://user:password@example.com:21/some/path');

	echo $uri->getAuthority();  // user@example.com:21

默认情况下，这不会显示密码部分，因为您不想显示给任何人。如果要显示密码，可以使用 ``showPassword()`` 方法。此URI实例将继续显示该密码，直到您再次将其关闭为止，因此请务必确保在完成密码后立即将其关闭::

	echo $uri->getAuthority();  // user@example.com:21
	echo $uri->showPassword()->getAuthority();   // user:password@example.com:21

	// 再次关闭密码显示。
	$uri->showPassword(false);

如果您不想显示端口，请传递 ``true`` 作为唯一参数::

	echo $uri->getAuthority(true);  // user@example.com

.. note:: 如果当前端口是该方案的默认端口，它将永远不会显示。

Userinfo
--------

userinfo部分只是您可能会在FTP URI中看到的用户名和密码。虽然您可以将其作为 ``Authority`` 的一部分，但也可以自己进行检索::

	echo $uri->getUserInfo();   // user

默认情况下，它将不会显示密码，但是你可以通过 ``showPassword()`` 方法来覆盖它::

	echo $uri->showPassword()->getUserInfo();   // user:password
	$uri->showPassword(false);

Host
----

URI的主机部分通常是URL的域名。可以使用 ``getHost()`` 和 ``setHost()`` 方法轻松设置和检索该代码::

	$uri = new \CodeIgniter\HTTP\URI('http://www.example.com/some/path');

	echo $uri->getHost();   // www.example.com
	echo $uri->setHost('anotherexample.com')->getHost();    // anotherexample.com

Port
----

该端口是0到65535之间的整数。每个sheme都有一个与之关联的默认值。

::

	$uri = new \CodeIgniter\HTTP\URI('ftp://user:password@example.com:21/some/path');

	echo $uri->getPort();   // 21
	echo $uri->setPort(2201)->getPort(); // 2201

使用 ``setPort()`` 方法时，将检查端口是否在有效范围内并进行了分配。

Path
----

路径是网站本身内的所有片段。如预期的那样， ``getPath()`` 和 ``setPath()`` 方法可用于对其进行操作::

	$uri = new \CodeIgniter\HTTP\URI('http://www.example.com/some/path');

	echo $uri->getPath();   // 'some/path'
	echo $uri->setPath('another/path')->getPath();  // 'another/path'

.. note:: 以这种方式或类允许的其他方式设置 path 的时候，将会对危险字符进行编码，并移除点分段来确保安全。

Query
-----

可以使用简单的字符串表示形式在类中操纵查询变量。查询值目前只能设置为字符串。

::

	$uri = new \CodeIgniter\HTTP\URI('http://www.example.com?foo=bar');

	echo $uri->getQuery();  // 'foo=bar'
	$uri->setQuery('foo=bar&bar=baz');

.. note:: 查询值不能包含片段。如果这样做，则将抛出InvalidArgumentException异常。

您可以使用数组设置查询值::

    $uri->setQueryArray(['foo' => 'bar', 'bar' => 'baz']);

``setQuery()`` 和 ``setQueryArray()`` 方法覆盖任何现有的查询变量。您可以在不破坏现有查询变量的情况下将值添加到查询变量集合 ``addQuery()`` 方法中。第一个参数是变量的名称，第二个参数是值::

    $uri->addQuery('foo', 'bar');

**过滤查询值**

您可以通过将options数组传递给 ``getQuery()`` 方法来过滤返回的查询值，方法是使用 *only* 或 *except* 键::

    $uri = new \CodeIgniter\HTTP\URI('http://www.example.com?foo=bar&bar=baz&baz=foz');

    // Returns 'foo=bar'
    echo $uri->getQuery(['only' => ['foo']);

    // Returns 'foo=bar&baz=foz'
    echo $uri->getQuery(['except' => ['bar']]);

这只会更改当次调用期间返回的值。如果您需要更永久地修改URI的查询值，则可以使用 ``stripQuery()`` 和 ``keepQuery()`` 方法更改实际对象的查询变量集合::

    $uri = new \CodeIgniter\HTTP\URI('http://www.example.com?foo=bar&bar=baz&baz=foz');

    // 只保留'baz'变量
    $uri->stripQuery('foo', 'bar');

    // 只保留'foo'变量
    $uri->keepQuery('foo');

Fragment
--------

片段是URL末尾的部分，后跟井号（＃）。在HTML URL中，这些是指向页面锚的链接。媒体URI可以通过其他各种方式使用它们。

::

	$uri = new \CodeIgniter\HTTP\URI('http://www.example.com/some/path#first-heading');

	echo $uri->getFragment();   // 'first-heading'
	echo $uri->setFragment('second-heading')->getFragment();    // 'second-heading'

============
URI片段
============

斜线之间的路径的每个部分都是单个片段。URI类提供一种确定段值的简单方法。分段从路径最左边的1开始。

::

	// URI = http://example.com/users/15/profile

	// 打印 '15'
	if ($request->uri->getSegment(1) == 'users')
	{
		echo $request->uri->getSegment(2);
	}

您可以计算出全部片段::

	$total = $request->uri->getTotalSegments(); // 3

最后，您可以检索所有片段的数组::

	$segments = $request->uri->getSegments();

	// $segments =
	[
		0 => 'users',
		1 => '15',
		2 => 'profile'
	]
