########################
生成查询结果
########################

有几种方法可以生成查询结果:

.. contents::
    :local:
    :depth: 2

*************
结果数组
*************

**getResult()**

此方法以对象数组或失败时为空数组的形式返回查询结果。通常，您将在foreach循环中使用此代码，如下所示::

    $query = $db->query("YOUR QUERY");

    foreach ($query->getResult() as $row)
    {
        echo $row->title;
        echo $row->name;
        echo $row->body;
    }

上面的方法是 ``getResultObject()`` 的别名。

如果希望将结果作为数组的数组，则可以传递字符串'array'::

    $query = $db->query("YOUR QUERY");

    foreach ($query->getResult('array') as $row)
    {
        echo $row['title'];
        echo $row['name'];
        echo $row['body'];
    }

上面的用法是 ``getResultArray()`` 的别名。

您还可以将 ``getResult()`` 表示要实例化的类的字符串传递给每个结果对象

::

    $query = $db->query("SELECT * FROM users;");

    foreach ($query->getResult('User') as $user)
    {
        echo $user->name; // 访问属性
        echo $user->reverseName(); // 或者定义的'User'类的方法
    }

上面的方法是 ``getCustomResultObject()`` 的别名。

**getResultArray()**

此方法以纯数组形式返回查询结果，如果不生成结果，则以空数组形式返回查询结果。通常，您将在foreach循环中使用此代码，如下所示::

    $query = $db->query("YOUR QUERY");

    foreach ($query->getResultArray() as $row)
    {
        echo $row['title'];
        echo $row['name'];
        echo $row['body'];
    }

***********
结果行
***********

**getRow()**

此方法返回单个结果行。如果查询有多个行，则仅返回第一行。结果作为对象返回 。这是一个用法示例::

    $query = $db->query("YOUR QUERY");

    $row = $query->getRow();

    if (isset($row))
    {
        echo $row->title;
        echo $row->name;
        echo $row->body;
    }

如果要返回特定行，可以在第一个参数中以数字形式提交行号::

	$row = $query->getRow(5);

您还可以添加第二个字符串参数，这是用于实例化行的类的名称::

	$query = $db->query("SELECT * FROM users LIMIT 1;");
	$row = $query->getRow(0, 'User');

	echo $row->name; // 访问属性
	echo $row->reverse_name(); // 或者定义的'User'类的方法

**getRowArray()**

与上述 ``row()`` 方法相同，除了它返回一个数组。
示例::

    $query = $db->query("YOUR QUERY");

    $row = $query->getRowArray();

    if (isset($row))
    {
        echo $row['title'];
        echo $row['name'];
        echo $row['body'];
    }

如果要返回特定行，可以在第一个参数中以数字形式提交行号::

	$row = $query->getRowArray(5);

此外，您可以使用以下变体在结果中前进/后退/第一/最后:

	| **$row = $query->getFirstRow()**
	| **$row = $query->getLastRow()**
	| **$row = $query->getNextRow()**
	| **$row = $query->getPreviousRow()**

默认情况下，除非您在参数中输入“array”一词，否则它们将返回一个对象:

	| **$row = $query->getFirstRow('array')**
	| **$row = $query->getLastRow('array')**
	| **$row = $query->getNextRow('array')**
	| **$row = $query->getPreviousRow('array')**

.. note:: 上面的所有方法都会将整个结果加载到内存中（预取）。使用 ``getUnbufferedRow()`` f用于处理大型的结果集。

**getUnbufferedRow()**

此方法返回单个结果行，而不会像 ``row()`` 在内存中那样预取整个结果。如果查询有多行，它将返回当前行并将内部数据指针向前移动。

::

    $query = $db->query("YOUR QUERY");

    while ($row = $query->getUnbufferedRow())
    {
        echo $row->title;
        echo $row->name;
        echo $row->body;
    }

您可以选择传递“对象”（默认）或“数组”，以指定返回值的类型::

	$query->getUnbufferedRow();		    // object
	$query->getUnbufferedRow('object');	// object
	$query->getUnbufferedRow('array');	// 关联数组

*********************
自定义结果对象
*********************

在 ``getResult()`` 和 ``getResultArray()`` 方法允许的情况下，您可以将结果作为自定义类的实例而不是 ``stdClass`` 或数组返回。如果该类尚未加载到内存中，则自动加载器将尝试加载它。该对象会将数据库返回的所有值设置为属性。如果这些已声明并且是非公开的，则应提供一种 ``__set()`` 方法来设置它们。

示例::

	class User
	{
		public $id;
		public $email;
		public $username;

		protected $last_login;

		public function lastLogin($format)
		{
			return $this->lastLogin->format($format);
		}

		public function __set($name, $value)
		{
			if ($name === 'lastLogin')
			{
				$this->lastLogin = DateTime::createFromFormat('U', $value);
			}
		}

		public function __get($name)
		{
			if (isset($this->$name))
			{
				return $this->$name;
			}
		}
	}

除了下面列出的两种方法，以下方法也可以采取一个类名返回的结果为： ``getFirstRow()``, ``getLastRow()``, ``getNextRow()``, 和 ``getPreviousRow()``。

**getCustomResultObject()**

以请求类的实例数组的形式返回整个结果集。唯一的参数是要实例化的类的名称。

示例::

	$query = $db->query("YOUR QUERY");

	$rows = $query->getCustomResultObject('User');

	foreach ($rows as $row)
	{
		echo $row->id;
		echo $row->email;
		echo $row->last_login('Y-m-d');
	}

**getCustomRowObject()**

从查询结果中返回一行。第一个参数是结果的行号。第二个参数是要实例化的类名。

示例::

	$query = $db->query("YOUR QUERY");

	$row = $query->getCustomRowObject(0, 'User');

	if (isset($row))
	{
		echo $row->email;   // 访问属性
		echo $row->last_login('Y-m-d');   // 访问类方法
	}

您也可以以完全相同的方式使用 ``getRow()`` 方法。

示例::

	$row = $query->getCustomRowObject(0, 'User');

*********************
结果辅助方法
*********************

**getFieldCount()**

查询返回的FIELDS（列）数。确保使用查询结果对象调用该方法::

	$query = $db->query('SELECT * FROM my_table');

	echo $query->getFieldCount();

**getFieldNames()**

返回一个数组，其中包含查询返回的FIELDS（列）的名称。确保使用查询结果对象调用该方法::

    $query = $db->query('SELECT * FROM my_table');

	echo $query->getFieldNames();

**freeResult()**

它将释放与结果关联的内存，并删除结果资源ID。通常，PHP在脚本执行结束时自动释放其内存。但是，如果您正在特定脚本中运行大量查询，则可能希望在生成每个查询结果之后释放结果，以减少内存消耗。

示例::

	$query = $thisdb->query('SELECT title FROM my_table');

	foreach ($query->getResult() as $row)
	{
		echo $row->title;
	}

	$query->freeResult();  // $query结果对象将不再可用

	$query2 = $db->query('SELECT name FROM some_table');

	$row = $query2->getRow();
	echo $row->name;
	$query2->freeResult(); // $query2结果对象将不再可用

**dataSeek()**

此方法为要提取的下一个结果行设置内部指针。仅与 ``getUnbufferedRow()`` 结合使用。

它接受一个正整数值，该值默认为0，成功则返回TRUE，失败则返回FALSE。

::

	$query = $db->query('SELECT `field_name` FROM `table_name`');
	$query->dataSeek(5); // 跳过前5行
	$row = $query->getUnbufferedRow();

.. note:: 并非所有的数据库驱动程序都支持此功能，并且将返回FALSE。最值得注意的是-您将无法在PDO中使用它。

***************
类参考
***************

.. php:class:: CodeIgniter\\Database\\BaseResult

	.. php:method:: getResult([$type = 'object'])

		:param	string	$type: 请求结果的类型-数组，对象或类名
		:returns:	包含获取的行的数组
		:rtype:	array

		一个包装的 ``getResultArray()``, ``getResultObject()`` 和 ``getCustomResultObject()`` 方法。

		用法: 参看 `结果数组`_.

	.. php:method:: getResultArray()

		:returns:	包含获取的行的数组
		:rtype:	array

		以行数组的形式返回查询结果，其中每一行本身就是一个关联数组。

		用法: 参看 `结果数组`_.

	.. php:method:: getResultObject()

		:returns:	包含获取的行的数组
		:rtype:	array

		以行数组的形式返回查询结果，其中每一行都是一个 ``stdClass`` 类型的对象。

		用法: 参看 `结果数组`_.

	.. php:method:: getCustomResultObject($class_name)

		:param	string	$class_name: 结果行的类名
		:returns:	包含获取的行的数组
		:rtype:	array

		以行数组的形式返回查询结果，其中每一行都是指定类的实例。

	.. php:method:: getRow([$n = 0[, $type = 'object']])

		:param	int	$n: 要返回的查询结果行的索引
		:param	string	$type: 请求结果的类型-数组，对象或类名
		:returns:	请求的行；如果不存在，则为NULL
		:rtype:	mixed

		一个包装的 ``getRowArray()``, ``getRowObject()`` 和 ``getCustomRowObject()`` 方法。

		用法: 参看 `结果行`_.

	.. php:method:: getUnbufferedRow([$type = 'object'])

		:param	string	$type: 请求结果的类型-数组，对象或类名
		:returns:	结果集中的下一行；如果不存在，则为NULL
		:rtype:	mixed

		获取下一个结果行，并以请求的形式返回它。

		用法: 参看 `结果行`_.

	.. php:method:: getRowArray([$n = 0])

		:param	int	$n: 要返回的查询结果行的索引
		:returns:	请求的行；如果不存在，则为NULL
		:rtype:	array

		以关联数组的形式返回请求的结果行。

		用法: 参看 `结果行`_.

	.. php:method:: getRowObject([$n = 0])

		:param	int	$n: 要返回的查询结果行的索引
		:returns:	请求的行；如果不存在，则为NULL
		:rtype:	stdClass

		返回以 ``stdClass`` 类型为对象的请求的结果行。

		用法: 参看 `结果行`_.

	.. php:method:: getCustomRowObject($n, $type)

		:param	int	$n: 要返回的结果行的索引
		:param	string	$class_name: 结果行的类名称
		:returns:	请求的行；如果不存在，则为NULL
		:rtype:	$type

		返回请求的结果行作为请求的类的实例。

	.. php:method:: dataSeek([$n = 0])

		:param	int	$n: 下一个要返回的结果行的索引
		:returns:	成功则为TRUE，失败则为FALSE
		:rtype:	bool

		将内部结果行指针移动到所需的偏移量。

		用法: 参看 `结果辅助方法`_.

	.. php:method:: setRow($key[, $value = NULL])

		:param	mixed	$key: 列名或键/值对数组
		:param	mixed	$value: 分配给列的值，$key是单个字段名称
		:rtype:	void

		为特定列分配一个值。

	.. php:method:: getNextRow([$type = 'object'])

		:param	string	$type: 请求结果的类型-数组，对象或类名
		:returns:	结果集的下一行；如果不存在，则为NULL
		:rtype:	mixed

		返回结果集中的下一行。

	.. php:method:: getPreviousRow([$type = 'object'])

		:param	string	$type: 请求结果的类型-数组，对象或类名
		:returns:	结果集的上一行；如果不存在，则为NULL
		:rtype:	mixed

		返回结果集中的上一行。

	.. php:method:: getFirstRow([$type = 'object'])

		:param	string	$type: 请求结果的类型-数组，对象或类名
		:returns:	结果集的第一行；如果不存在，则为NULL
		:rtype:	mixed

		返回结果集中的第一行。

	.. php:method:: getLastRow([$type = 'object'])

		:param	string	$type: 请求结果的类型-数组，对象或类名
		:returns:	结果集的最后一行；如果不存在，则为NULL
		:rtype:	mixed

		返回结果集中的最后一行。

	.. php:method:: getFieldCount()

		:returns:	结果集中的字段数
		:rtype:	int

		返回结果集中的字段数。

		用法: 参看 `结果辅助方法`_.

    .. php:method:: getFieldNames()

		:returns:	列名数组
		:rtype:	array

		返回一个包含结果集中的字段名称的数组。

	.. php:method:: getFieldData()

		:returns:	包含字段元数据的数组
		:rtype:	array

		生成 ``stdClass`` 包含字段元数据的对象数组。

	.. php:method:: freeResult()

		:rtype:	void

		释放结果集。

		用法: 参看 `结果辅助方法`_.
