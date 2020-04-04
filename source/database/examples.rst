##################################
快速入门: 示例代码
##################################

以下页面包含示例代码，显示了如何使用数据库类。有关完整的详细信息，请阅读描述每个功能的各个页面。

初始化数据库类
===============================

以下代码根据您的 :doc:`配置 <configuration>` 设置加载并初始化数据库类::

	$db = \Config\Database::connect();

加载后，该类可以按如下所述使用。

.. note:: 如果所有页面都需要数据库访问权限，则可以自动连接。有关详细信息，请参见 :doc:`连接 <connecting>` 页面。

具有多个结果的标准查询（对象版本）
=====================================================

::

	$query = $db->query('SELECT name, title, email FROM my_table');
	$results = $query->getResult();

	foreach ($results as $row)
	{
		echo $row->title;
		echo $row->name;
		echo $row->email;
	}

	echo 'Total Results: ' . count($results);

上面的getResult()函数返回一个对象数组。示例：$row->title

具有多个结果的标准查询（数组版本）
====================================================

::

	$query   = $db->query('SELECT name, title, email FROM my_table');
	$results = $query->getResultArray();

	foreach ($results as $row)
	{
		echo $row['title'];
		echo $row['name'];
		echo $row['email'];
	}

上面的getResultArray()函数返回一个标准数组。示例：$row['title']

标准查询单个结果
=================================

::

	$query = $db->query('SELECT name FROM my_table LIMIT 1');
	$row   = $query->getRow();
	echo $row->name;

上面的getRow()函数返回一个对象。 示例: $row->name

具有单个结果的标准查询（数组版本）
=================================================

::

	$query = $db->query('SELECT name FROM my_table LIMIT 1');
	$row   = $query->getRowArray();
	echo $row['name'];

上面的getRowArray()函数返回一个数组。 示例: $row['name']

标准插入
===============

::

	$sql = "INSERT INTO mytable (title, name) VALUES (".$db->escape($title).", ".$db->escape($name).")";
	$db->query($sql);
	echo $db->affectedRows();

查询生成器查询
===================

:doc:`Query Builder 模式 <query_builder>` 给你检索数据的简化方式::

	$query = $db->table('table_name')->get();

	foreach ($query->getResult() as $row)
	{
		echo $row->title;
	}

上面的get()函数从提供的表中检索所有结果。 :doc:`Query Builder <query_builder>` 类包含的功能，用于处理数据的完整补充。

查询生成器插入
====================

::

	$data = [
		'title' => $title,
		'name'  => $name,
		'date'  => $date
	];

	$db->table('mytable')->insert($data);  // Produces: INSERT INTO mytable (title, name, date) VALUES ('{$title}', '{$name}', '{$date}')

