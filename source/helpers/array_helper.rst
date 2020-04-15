################
数组辅助函数
################

数组辅助函数提供了一些功能来简化更复杂的数组用法。它无意于复制PHP提供的任何现有功能-除非要大大简化其用法。

.. contents::
    :local:

加载辅助函数
===================

使用以下代码加载辅助函数::

	helper('array');

可用函数
===================

该辅助函数有下列可用函数：

..  php:function:: dot_array_search(string $search, array $values)

    :param  string  $search: 描述如何搜索数组的点号字符串
    :param  array   $values: 搜索的数组
    :returns: 在数组中找到的值，或者为null
    :rtype: mixed

    此方法允许您使用点符号在数组中搜索特定键，并允许使用'*'通配符。给定以下数组::

        $data = [
            'foo' => [
                'buzz' => [
                    'fizz' => 11
                ],
                'bar' => [
                    'baz' => 23
                ]
            ]
        ]

    我们可以使用搜索字符串 "foo.bar.baz" 来定位 ``fizz`` 的值。同样，可以使用 "foo.bar.baz" 找到 ``baz`` 的值::

        // 返回: 11
        $fizz = dot_array_search('foo.buzz.fizz', $data);

        // 返回: 23
        $baz = dot_array_search('foo.bar.baz', $data);

    您可以使用星号作为通配符来替换任何段。找到后，它将搜索所有子节点，直到找到为止。如果您不知道这些值，或者您的值具有数字索引，这将很方便::

        // 返回: 23
        $baz = dot_array_search('foo.*.baz', $data);
