#################
Cookie 辅助函数
#################

Cookie辅助函数文件包含协助使用Cookie的功能。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

加载辅助函数
===================

使用以下代码加载辅助函数::

	helper('cookie');

可用函数
===================

该辅助函数有下列可用函数：

.. php:function:: set_cookie($name[, $value = ''[, $expire = ''[, $domain = ''[, $path = '/'[, $prefix = ''[, $secure = false[, $httpOnly = false]]]]]]])

	:param	mixed	$name: Cookie名称或该功能可用的所有参数的关联数组
	:param	string	$value: Cookie值
	:param	int	$expire: 过期前的秒数
	:param	string	$domain: Cookie域（通常：.yourdomain.com）
	:param	string	$path: Cookie路径
	:param	string	$prefix: Cookie名称前缀
	:param	bool	$secure: 是否仅通过HTTPS发送cookie
	:param	bool	$httpOnly: 是否从JavaScript隐藏cookie
	:rtype:	void

	此辅助函数为您提供了更友好的语法来设置浏览器cookie。请参阅 :doc:`Response 库 </outgoing/response>` 以获取有关其用法的说明，因为此函数是 ``Response::setCookie()`` 的别名。

.. php:function:: get_cookie($index[, $xssClean = false])

	:param	string	$index: Cookie名称
	:param	bool	$xss_clean: 是否对返回值应用XSS过滤
	:returns:	Cookie值；如果找不到，则为NULL
	:rtype:	mixed

	该辅助函数为您提供了更友好的语法来获取浏览器cookie。请参阅 :doc:`IncomingRequest 库 </incoming/incomingrequest>` 以获取有关其用法的详细说明，因为此功能的作用与 ``IncomingRequest::getCookie()`` 十分相似，不同之处在于，它还会以您可能已在 *app/Config/App.php* 文件中 ``$cookiePrefix`` 的设置为准。

.. php:function:: delete_cookie($name[, $domain = ''[, $path = '/'[, $prefix = '']]])

	:param	string	$name: Cookie名称
	:param	string	$domain: Cookie域（通常：.yourdomain.com）
	:param	string	$path: Cookie路径
	:param	string	$prefix:  Cookie名称前缀
	:rtype:	void

	用于删除Cookie。除非您设置了自定义路径或其他值，否则仅需要Cookie的名称。
	
	::

		delete_cookie('name');

	该函数在其他方面与 ``set_cookie()`` 相同，除了它没有 `value` 和 `expiration` 参数。您可以在第一个参数中提交值数组，也可以设置离散参数。	
	
	::

		delete_cookie($name, $domain, $path, $prefix);