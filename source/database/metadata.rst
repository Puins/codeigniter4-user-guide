#################
数据库元数据
#################

.. contents::
    :local:
    :depth: 2

**************
表元数据
**************

这些功能使您可以获取表信息。

列出数据库中的所有表
================================

**$db->listTables();**

返回一个数组，其中包含当前连接到的数据库中所有表的名称。示例::

	$tables = $db->listTables();

	foreach ($tables as $table)
	{
		echo $table;
	}
	
.. note:: 某些驱动程序具有其他系统表，该表不包含在此返回中。

确定表是否存在
===========================

**$db->tableExists();**

有时在对特定表进行操作之前了解其是否存在很有用。返回布尔值TRUE/FALSE。用法示例::

	if ($db->tableExists('table_name'))
	{
		// some code...
	}

.. note:: 将 *table_name* 替换为您要查找的表的名称。

**************
字段元数据
**************

列出表中的所有字段
==========================

**$db->getFieldNames()**

返回包含字段名称的数组。可以通过两种方式调用此查询:

1. 你可以提供表名称从 $db->object 中调用它::

	$fields = $db->getFieldNames('table_name');

	foreach ($fields as $field)
	{
		echo $field;
	}

2. 你可以从任何查询结果对象上调用该方法，获取查询返回的所有字段::

	$query = $db->query('SELECT * FROM some_table');

	foreach ($query->getFieldNames() as $field)
	{
		echo $field;
	}

检查表中是否存在某字段
==========================================

**$db->fieldExists()**

有时，在执行一个操作之前先确定某个字段是否存在是很有用的。该方法返回布尔值 TRUE/FALSE。 用法示例::

	if ($db->fieldExists('field_name', 'table_name'))
	{
		// some code...
	}

.. note:: 将 *field_name* 替换为你要查找的字段名, 并且将 *table_name* 替换为你要查找的表的名称

检索字段元数据
=======================

**$db->getFieldData()**

返回包含字段信息的对象数组。

有时收集字段名称或其他元数据（例如列类型，最大长度等）会很有帮助。

.. note:: 并非所有数据库都提供元数据。

用法示例::

	$fields = $db->getFieldData('table_name');

	foreach ($fields as $field)
	{
		echo $field->name;
		echo $field->type;
		echo $field->max_length;
		echo $field->primary_key;
	}

如果已经运行查询，则可以使用结果对象而不是提供表名::

	$query  = $db->query("YOUR QUERY");
	$fields = $query->fieldData();

如果您的数据库支持，则可以从此函数获取以下数据:

-  name - 列名称
-  max_length - 列的最大长度
-  primary_key - 如果列是主键，则为1
-  type - 列的类型

列出表中的索引
===========================

**$db->getIndexData()**

返回包含索引信息的对象数组。

用法示例::

	$keys = $db->getIndexData('table_name');

	foreach ($keys as $key)
	{
		echo $key->name;
		echo $key->type;
		echo $key->fields;  // 字段名数组
	}

密钥类型对于您正在使用的数据库可能是唯一的。例如，MySQL将为与表关联的每个键返回主键，全文，空间，索引或唯一索引之一。

**$db->getForeignKeyData()**

返回包含外键信息的对象数组。

用法示例::

	$keys = $db->getForeignKeyData('table_name');

	foreach ($keys as $key)
	{
		echo $key->constraint_name;
		echo $key->table_name;
		echo $key->column_name;
		echo $key->foreign_table_name;
		echo $key->foreign_column_name;
	}

对象字段对于您正在使用的数据库可能是唯一的。例如，SQLite3不返回有关列名的数据，但是具有用于复合外键定义的附加 *sequence* 字段。
