#####################
Inflector 辅助函数
#####################

Inflector 辅助函数文件包含允许您将 **English** 单词更改为复数，单数，驼峰大小写等的功能。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

加载辅助函数
===================

使用以下代码加载辅助函数::

	helper('inflector');

可用函数
===================

该辅助函数有下列可用函数：

.. php:function:: singular($string)

	:param	string	$string: 输入字符串
	:returns:	单数单词
	:rtype:	string

	改变复数单词为单数。 示例::

		echo singular('dogs'); // 打印 'dog'

.. php:function:: plural($string)

	:param	string	$string: 输入字符串
	:returns:	复数单词
	:rtype:	string

	改变单数单词为复数。示例::

		echo plural('dog'); // 打印 'dogs'

.. php:function:: counted($count, $string)

	:param	int 	$count:  数字项
	:param	string	$string: 输入字符串
	:returns:	单数或复数短语
	:rtype:	string

	将单词及其计数更改为短语。 示例::

		echo counted(3, 'dog'); // 打印 '3 dogs'

.. php:function:: camelize($string)

	:param	string	$string: 输入字符串
	:returns:	驼峰化字符串
	:rtype:	string

	将由空格或下划线分隔的单词字符串更改为驼峰式大小写。 示例::

		echo camelize('my_dog_spot'); // 打印 'myDogSpot'

.. php:function:: pascalize($string)

	:param	string	$string: 输入字符串
	:returns:	Pascal大小写字符串
	:rtype:	string

	将由空格或下划线分隔的单词字符串更改为Pascal大小写，即首字母大写的驼峰式大小写。示例::

		echo pascalize('my_dog_spot'); // 打印 'MyDogSpot'

.. php:function:: underscore($string)

	:param	string	$string: 输入字符串
	:returns:	包含下划线而不是空格的字符串
	:rtype:	string

	获取多个用空格隔开的单词并加下划线。示例::

		echo underscore('my dog spot'); // 打印 'my_dog_spot'

.. php:function:: humanize($string[, $separator = '_'])

	:param	string	$string: 输入字符串
	:param	string	$separator: 输入分隔符
	:returns:	人性化的字符串
	:rtype:	string

	用下划线分隔多个单词，并在它们之间添加空格。每个单词都大写。示例::

		echo humanize('my_dog_spot'); // 打印 'My Dog Spot'

	用破折号代替下划线::

		echo humanize('my-dog-spot', '-'); // 打印 'My Dog Spot'

.. php:function:: is_pluralizable($word)

	:param	string	$word: 输入字符串
	:returns:	如果单词可数则为TRUE，否则为FALSE
	:rtype:	bool

	检查给定单词是否具有复数形式。示例::

		is_pluralizable('equipment'); // 返回 FALSE

.. php:function:: dasherize($string)

	:param	string	$string: 输入字符串
	:returns:	虚线
	:rtype:	string

	在字符串中用虚线代替下划线。 示例::

		dasherize('hello_world'); // 返回 'hello-world'

.. php:function:: ordinal($integer)

	:param	int	$integer: 确定后缀的整数
	:returns:	序数后缀
	:rtype:	string

	返回应该添加到数字以表示位置的后缀，例如1st，2nd，3rd，4th。示例::

		ordinal(1); // 返回 'st'

.. php:function:: ordinalize($integer)

	:param	int	$integer: 序号
	:returns:	序数化 integer
	:rtype:	string

	将数字转换为用于表示位置的序数字符串，例如1st，2nd，3rd，4th。示例::

		ordinalize(1); // 返回 '1st'
