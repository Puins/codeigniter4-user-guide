####################
Security 辅助函数
####################

Security 辅助函数包含与安全相关的功能。

.. contents::
  :local:

加载此辅助函数
===================

使用以下代码加载此辅助函数::

	helper('security');

可用功能
===================

以下功能可用：

.. php:function:: sanitize_filename($filename)

	:param	string	$filename: 文件名
    	:returns:	清理文件名
    	:rtype:	string

    	提供防止目录遍历的保护。

	此函数是 ``\CodeIgniter\Security::sanitize_filename()`` 的别名。有关更多信息，请参见 :doc:`Security 库 <../libraries/security>` 文档。

.. php:function:: strip_image_tags($str)

	:param	string	$str: 输入字符串
    	:returns:	无 ``image`` 标签的输入字符串
    	:rtype:	string

	这是一项安全函数，可以从字符串中剥离 ``image`` 标签。它将 ``image URL`` 保留为纯文本。

    	示例::

		$string = strip_image_tags($string);

.. php:function:: encode_php_tags($str)

	:param	string	$str: 输入字符串
    	:returns:	安全格式化的字符串
    	:rtype:	string

    	这是一个将PHP标记转换为实体的安全函数。

	示例::

		$string = encode_php_tags($string);
