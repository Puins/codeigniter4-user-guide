###############
Date 辅助函数
###############

Date 辅助函数文件包含协助使用Date的功能。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

加载此辅助函数
===================

使用以下代码加载此辅助函数::

	helper('date');

可用功能
===================

以下功能可用：

.. php:function:: now([$timezone = NULL])

	:param	string	$timezone: 时区
	:returns:	UNIX时间戳
	:rtype:	int

	根据配置文件中的 "time reference" 设置，以UNIX时间戳返回当前时间，该时间以服务器的本地时间或任何PHP支持的时区为参考。如果您不打算将主时间参考设置为任何其他PHP支持的时区（如果您运行一个允许每个用户设置自己的时区设置的站点，通常会这样做），则与PHP的 ``time()`` 函数相比，使用此函数没有任何好处。

	::

		echo now('Australia/Victoria');

	如果未提供时区，则它将根据 **time_reference** 设置返回 ``time()``。

.. php:function:: timezone_select([$class = '', $default = '', $what = \DateTimeZone::ALL, $country = null])

	:param	string	$class: 应用于选择字段的可选类
	:param	string	$default: 初始选择的默认值
	:param	int	$what: DateTimeZone类常量 (请参看 `listIdentifiers <https://www.php.net/manual/en/datetimezone.listidentifiers.php>`_)
	:param	string	$country: 两个字母的兼容ISO 3166-1的国家/地区代码 (请参看 `listIdentifiers <https://www.php.net/manual/en/datetimezone.listidentifiers.php>`_)
	:returns:	预格式化的HTML选择字段
	:rtype:	string

	生成可用时区的 `select` 表单字段（可选地由 `$what` 和 `$country` 过滤）。您可以提供一个选项类以应用于该字段，以简化格式设置，并提供一个默认的选定值。

	::

		echo timezone_select('custom-select', 'America/New_York');

先前在CodeIgniter 3中的 ``date_helper`` 找到的许多功能已移至CodeIgniter 4中的 ``I18n`` 模块。
