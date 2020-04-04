****************************************************
Request 类
****************************************************

Request 类是HTTP请求的面向对象表示。这意味着既可以处理传入请求（例如来自浏览器的应用程序请求），也可以处理传出请求（例如用于将请求从应用程序发送到第三方应用程序）的工作。此类提供了它们都需要的通用功能，但是两种情况都具有自定义类，这些类从Request类扩展以添加特定功能。

有关更多用法的详细信息，请参见 :doc:`IncomingRequest 类 </incoming/incomingrequest>` 和 :doc:`CURLRequest 类 </libraries/curlrequest>` 的文档。

类参考
============================================================

.. php:class:: CodeIgniter\\HTTP\\Request

	.. php:method:: getIPAddress() 

		:returns: 可以检测到的用户 IP 地址，否则为 null ，如果 IP 地址无效，则返回 0.0.0.0
		:rtype: string

		返回当前用户的IP地址。如果IP地址无效，则该方法将返回'0.0.0.0'::

			echo $request->getIPAddress();

		.. important:: 此方法考虑了 ``App->proxyIPs`` 设置，并将为允许的IP地址返回报告的HTTP_X_FORWARDED_FOR，HTTP_CLIENT_IP，HTTP_X_CLIENT_IP或HTTP_X_CLUSTER_CLIENT_IP地址。

	.. php:method:: isValidIP($ip[, $which = ''])

		:param	string	$ip: IP地址
		:param	string	$which: IP 协议 ('ipv4' 或 'ipv6')
		:returns:	如果地址有效，则为true；否则为false
		:rtype:	bool

		将IP地址用作输入，并根据有效与否返回true或false（布尔值）。

		.. note:: $request->getIPAddress()方法会自动验证IP地址。

                ::

			if ( ! $request->isValidIP($ip))
			{
                            echo 'Not Valid';
			}
			else
			{
                            echo 'Valid';
			}

		接受可选的第二个字符串参数“ipv4”或“ipv6”以指定IP格式。默认检查两种格式。

	.. php:method:: getMethod([$upper = FALSE])

		:param	bool	$upper: 是否以大写或小写形式返回请求方法名称
		:returns:	HTTP request 方法
		:rtype:	string

		返回 ``$_SERVER['REQUEST_METHOD']``，并带有将其设置为大写或小写的选项。
		::

			echo $request->getMethod(TRUE); // 输出: POST
			echo $request->getMethod(FALSE); // 输出: post
			echo $request->getMethod(); // 输出: post

	.. php:method:: getServer([$index = null[, $filter = null[, $flags = null]]])
                :noindex:

		:param	mixed	$index: 值名称
		:param  int     $filter: 要应用的过滤器类型。可以在 `filter <https://www.php.net/manual/en/filter.filters.php>`_ 找到过滤器列表。
		:param  int     $flags: 要应用的标志。标志列表可以在 `filter flags <https://www.php.net/manual/en/filter.filters.flags.php>`_ 找到.
		:returns:	$ _SERVER项目值（如果找到），否则为NULL
		:rtype:	mixed

		此方法是和从 :doc:`IncomingRequest 类 </incoming/incomingrequest>` 所述的方法  ``post()``, ``get()`` and ``cookie()`` 相同的，该方法用来取得getServer数据（``$_SERVER``）::

			$request->getServer('some_data');

		要返回多个 ``$_SERVER`` 值的数组，请将所有必需的键作为数组传递。
		::

			$require->getServer(['SERVER_PROTOCOL', 'REQUEST_URI']);
