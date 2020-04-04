###############
XML 辅助函数
###############

XML 辅助函数文件包含协助使用XML的功能。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

加载此辅助函数
===================

使用以下代码加载此辅助函数::

	helper('xml');

可用功能
===================

以下功能可用：

.. php:function:: xml_convert($str[, $protect_all = FALSE])

	:param string $str: 要转换的文本字符串
	:param bool $protect_all: 是否保护所有看起来像潜在实体的内容，而不仅仅是编号实体， 例如 &foo;
	:returns: 转化成XML结构的字符串
	:rtype:	string

	将字符串作为输入，并将以下保留的XML字符转换为实体:

	  - 与操作符: &
	  - 大于小于号: < >
	  - 单双引号: ' "
	  - 破折号: -

	如果“＆”号是已有数字字符实体（例如 &#123;）的一部分，则该函数将忽略它们。示例::

		$string = '<p>Here is a paragraph & an entity (&#123;).</p>';
		$string = xml_convert($string);
		echo $string;

	输出:

	.. code-block:: html

		&lt;p&gt;Here is a paragraph &amp; an entity (&#123;).&lt;/p&gt;
