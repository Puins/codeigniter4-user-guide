#############
数字辅助函数
#############

数字辅助函数包含一些可帮助您以语言环境感知的方式处理数字数据的功能。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

加载此辅助函数
===================

使用以下代码加载此辅助函数::

	helper('number');

当事情出错时
====================

如果PHP的国际化和本地化逻辑无法处理对于给定的语言环境和选项提供的值， 则将抛出 ``BadFunctionCallException()`` 异常。

可用功能
===================

以下功能可用：

.. php:function:: number_to_size($num[, $precision = 1[, $locale = null])

    :param	mixed	$num: 字节数
    :param	int	$precision: 浮点精度 
    :returns:	格式化的数据大小字符串，如果提供的值不是数字，则为false
    :rtype:	string

    根据大小将数字格式化为字节，并添加适当的后缀。示例::

        echo number_to_size(456); // 返回 456 Bytes
        echo number_to_size(4567); // 返回 4.5 KB
        echo number_to_size(45678); // 返回 44.6 KB
        echo number_to_size(456789); // 返回 447.8 KB
        echo number_to_size(3456789); // 返回 3.3 MB
        echo number_to_size(12345678912345); // 返回 1.8 GB
        echo number_to_size(123456789123456789); // 返回 11,228.3 TB

    可选的第二个参数允许您设置结果的精度::

	    echo number_to_size(45678, 2); // 返回 44.61 KB

    可选的第三个参数使您可以指定在生成数字时应使用的语言环境，并且可以影响格式。如果未指定语言环境，则将分析请求，并从标题或app-default中获取适当的语言环境::

        // 产生 11.2 TB
        echo number_to_size(12345678912345, 1, 'en_US');
        // 产生 11,2 TB
        echo number_to_size(12345678912345, 1, 'fr_FR');

    .. note:: 在以下语言文件中找到此函数生成的文本： **language/<your_lang>/Number.php**

.. php:function:: number_to_amount($num[, $precision = 1[, $locale = null])

    :param	mixed	$num: 要格式化的数字
    :param	int	$precision: 浮点精度
    :param  string $locale: 用于格式化的语言环境
    :returns:	字符串的可读版本，如果提供的值不是数字，则返回false
    :rtype:	string

    将数字转换为人类可读的版本，例如 **123.4 trillion** ，直到四千万。 示例::

        echo number_to_amount(123456); // 返回 123 thousand
        echo number_to_amount(123456789); // 返回 123 million
        echo number_to_amount(1234567890123, 2); // 返回 1.23 trillion
        echo number_to_amount('123,456,789,012', 2); // 返回 123.46 billion

    可选的第二个参数允许您设置结果的精度::

        echo number_to_amount(45678, 2); // 返回 45.68 thousand

    可选的第三个参数允许指定语言环境::

        echo number_to_amount('123,456,789,012', 2, 'de_DE'); // 返回 123,46 billion

.. php:function:: number_to_currency($num, $currency[, $locale = null])

    :param mixed $num: 要格式化的数字
    :param string $currency: 货币类型，即USD，EUR等
    :param string $locale: 用于格式化的语言环境
    :param integer $fraction: 小数点后的小数位数
    :returns: 该数字作为区域设置的适当货币
    :rtype: string

    转换常用货币格式的数字，例如USD，EUR，GBP等::

        echo number_to_currency(1234.56, 'USD');  // 返回 $1,234.56
        echo number_to_currency(1234.56, 'EUR');  // 返回 €1,234.56
        echo number_to_currency(1234.56, 'GBP');  // 返回 £1,234.56
        echo number_to_currency(1234.56, 'YEN');  // 返回 YEN1,234.56

.. php:function:: number_to_roman($num)

    :param string $num: 要转换的数字
    :returns: 从给定参数转换的 `roman` 数字
    :rtype: string|null

    将数字转换为 `roman` 数字::

        echo number_to_roman(23);  // 返回 XXIII
        echo number_to_roman(324);  // 返回 CCCXXIV
        echo number_to_roman(2534);  // 返回 MMDXXXIV

    此函数仅处理1到3999范围内的数字。该范围外的任何值将返回null。