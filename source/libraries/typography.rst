####################
Typography 类
####################

Typography 库包含一些用于帮助您以语义相关的方式设置文本格式的方法。

*******************
加载类库
*******************

与 CodeIgniter 的所有其他服务一样，可以通过 ``Config\Services`` 来加载，通常不需要手动加载::

    $typography = \Config\Services::typography();

**************************
可用的静态方法
**************************

以下的方法是可用的:

**autoTypography()**

.. php:function:: autoTypography($str[, $reduce_linebreaks = FALSE])

	:param	string	$str: 输入字符串
	:param	bool	$reduce_linebreaks: 是否将双换行符的多个实例减少为两个
	:returns:	HTML 格式化排版的安全字符串
	:rtype: string

	格式化文本使其成为语义和排版正确的 HTML 。

	用法示例::

		$string = $typography->autoTypography($string);

	.. note:: 格式排版可能会消耗大量处理器资源，特别是在排版大量内容时。 如果你选择使用这个函数的话，你可以考虑 :doc:`caching <../general/caching>` 相关页面。

**formatCharacters()**

.. php:function:: formatCharacters($str)

	:param	string	$str: 输入字符串
	:returns:	带有格式化字符的字符串。
	:rtype:	string

	将双引号或单引号转成正确的实体，也会转化—破折号，双空格和&符号。

	用法示例::

		$string = $typography->formatCharacters($string);

**nl2brExceptPre()**

.. php:function:: nl2brExceptPre($str)

	:param	string	$str: 输入字符串
	:returns:	具有HTML格式的换行符的字符串
	:rtype:	string

	将换行转换为 <br /> 标签， 忽略 <pre> 标签中的换行符。这个函数和PHP原生的 ``nl2br()`` 函数是一样的，但忽略 <pre> 标签。

	用法示例::

		$string = $typography->nl2brExceptPre($string);
