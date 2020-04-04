########################################
数据库工厂(Database Forge)类
########################################

数据库工厂类（Database Forge）包含了帮助你管理你的数据库的一些相关方法。

.. contents::
    :local:
    :depth: 2

****************************
初始化 Forge 类
****************************

.. important:: 为了初始化Forge类，你的数据库驱动程序必须已经在运行，因为Forge类是依赖它运行的。

加载 Forge 类的代码如下::

	$forge = \Config\Database::forge();

你也可以将另外一个数据库组名传递给 DB Forge加载程序，以防要管理的数据库不是默认数据库::

	$this->myforge = \Config\Database::forge('other_db');

在上面的示例中，我们传递的是另一个数据库的名称作为第一个参数来连接。

*******************************
创建和删除数据库
*******************************

**$forge->createDatabase('db_name')**

允许创建第一个参数中指定的数据库。根据成败返回 TRUE 或 FALSE::

	if ($forge->createDatabase('my_db'))
	{
		echo 'Database created!';
	}

可选的第二个参数设置为TRUE将添加 ``IF EXISTS`` 语句或将在创建数据库之前检查数据库是否存在（取决于DBMS）。

::

	$forge->createDatabase('my_db', TRUE);
	// 附加 CREATE DATABASE IF NOT EXISTS my_db
	// or will check if a database exists

**$forge->dropDatabase('db_name')**

允许删除第一个参数中指定的数据库。根据成败返回 TRUE 或 FALSE::

	if ($forge->dropDatabase('my_db'))
	{
		echo 'Database deleted!';
	}

****************************
创建和删除数据表
****************************

在创建表时，你可能希望做一些事情。如添加字段，向表中添加键，更改列。CodeIgniter 为此提供了一种机制。

添加字段
=============

字段是通过关联数组创建的。在数组中，必须包括与字段的数据类型相关的 ``type`` 键。例如， ``int``、 ``varchar``、 ``text`` 等。许多数据类型（例如 ``varchar`` ）还需要“约束”键。

::

	$fields = [
		'users' => [
			'type'       => 'VARCHAR',
			'constraint' => 100,
		],
	];
	// 添加字段时将转换为"users VARCHAR(100)"。

此外，可以使用以下键/值:

-  unsigned/true : 在字段定义中生成 "UNSIGNED" 。
-  default/value : 在字段定义中生成默认值。
-  null/true : 在字段定义中生成"NULL"。如果没有这个，该字段将默认为"NOT NULL"。
-  auto_increment/true : 在字段上生成auto_increment标志。请注意，字段类型必须是支持此类型的类型，例如整数。
-  unique/true : 为字段定义生成唯一键。

::

	$fields = [
		'id'          => [
			'type'           => 'INT',
			'constraint'     => 5,
			'unsigned'       => true,
			'auto_increment' => true
		],
		'title'       => [
			'type'           => 'VARCHAR',
			'constraint'     => '100',
			'unique'         => true,
		],
		'author'      => [
			'type'           =>'VARCHAR',
			'constraint'     => 100,
			'default'        => 'King of Town',
		],
		'description' => [
			'type'           => 'TEXT',
			'null'           => true,
		],
		'status'      => [
			'type'           => 'ENUM',
			'constraint'     => ['publish', 'pending', 'draft'],
			'default'        => 'pending',
		],
	];

定义字段后，可以使用 ``$forge->addField($fields);`` 然后调用 ``createTable()`` 方法。

**$forge->addField()**

 ``addField()`` 方法将接受上述数组。

将字符串作为字段传递
-------------------------

如果你确切知道要如何创建字段，可以使用 ``addField()`` 方法将字符串传递给字段定义

::

	$forge->addField("label varchar(100) NOT NULL DEFAULT 'default label'");

.. note:: 将原始字符串作为字段传递后，不能用 ``addKey()`` 对这些字段进行调用。

.. note:: 对 ``addField()`` 的多次调用是累积的。

创建一个id字段
--------------------

创建id字段有一个特殊例外。具有类型id的字段将自动分配为 ``INT(9) auto_incrementing`` 主键。

::

	$forge->addField('id');
	// 附加 id INT(9) NOT NULL AUTO_INCREMENT

添加键
===========

通常来说，数据表都会有键。这可以使用 ``$forge->addKey('field')`` 方法来实现。第二个参数设置是可选的，设置为 TRUE 将使其成为主键，
第三个参数设置为 TRUE 将使其成为唯一键。注意 ``addKey()`` 方法必须紧跟在 ``createTable()`` 方法后面。

包含多列的非主键必须使用数组来添加，下面是 MySQL 的例子。

::

	$forge->addKey('blog_id', TRUE);
	// 附加 PRIMARY KEY `blog_id` (`blog_id`)

	$forge->addKey('blog_id', TRUE);
	$forge->addKey('site_id', TRUE);
	// 附加 PRIMARY KEY `blog_id_site_id` (`blog_id`, `site_id`)

	$forge->addKey('blog_name');
	// 附加 KEY `blog_name` (`blog_name`)

	$forge->addKey(['blog_name', 'blog_label']);
	// 附加 KEY `blog_name_blog_label` (`blog_name`, `blog_label`)

	$forge->addKey(['blog_id', 'uri'], FALSE, TRUE);
	// 附加 UNIQUE KEY `blog_id_uri` (`blog_id`, `uri`)

为了使代码读取更加客观，还可以使用特定的方法添加主键和唯一键。::

	$forge->addPrimaryKey('blog_id');
	// 附加 PRIMARY KEY `blog_id` (`blog_id`)

	$forge->addUniqueKey(['blog_id', 'uri']);
	// 附加 UNIQUE KEY `blog_id_uri` (`blog_id`, `uri`)


添加外键
===================

外键有助于跨表强制执行关系和操作。对于支持外键的表，可以直接在 ``forge`` 中添加它们。::

        $forge->addForeignKey('users_id','users','id');
        // 附加 CONSTRAINT `TABLENAME_users_foreign` FOREIGN KEY(`users_id`) REFERENCES `users`(`id`)

你可以为约束的 ``on delete`` 和 ``on update`` 属性指定所需的操作::

        $forge->addForeignKey('users_id','users','id','CASCADE','CASCADE');
        // 附加 CONSTRAINT `TABLENAME_users_foreign` FOREIGN KEY(`users_id`) REFERENCES `users`(`id`) ON DELETE CASCADE ON UPDATE CASCADE

创建数据表
================

声明字段和键后，你可以根据如下代码创建一张新表

::

	$forge->createTable('table_name');
	// 附加 CREATE TABLE table_name

可选的第二个参数设置为TRUE时会在定义中添加 ``IF NOT EXISTS`` 子句

::

	$forge->createTable('table_name', TRUE);
	// 附加 CREATE TABLE IF NOT EXISTS table_name

你还可以传递可选的表属性，例如MySQL的 ``ENGINE`` ::

	$attributes = ['ENGINE' => 'InnoDB'];
	$forge->createTable('table_name', FALSE, $attributes);
	// 生成: CREATE TABLE `table_name` (...) ENGINE = InnoDB DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci

.. note:: 除非你指定 ``CHARACTER SET`` 和/或 ``COLLATE`` 属性,``createTable()`` 否则将始终使用你配置的 *charset* 和 *DBCollat* 值, 只要它们不为空 (仅限MySQL).

删除数据表
================

执行DROP TABLE语句时，可以选择添加一个 ``IF EXISTS`` 子句。

::

	// 生成: DROP TABLE table_name
	$forge->dropTable('table_name');

	// 生成: DROP TABLE IF EXISTS table_name
	$forge->dropTable('table_name',TRUE);

删除外键
======================

执行数据表外键删除语句。

::

	// 生成: ALTER TABLE 'tablename' DROP FOREIGN KEY 'users_foreign'
	$forge->dropForeignKey('tablename','users_foreign');

重命名数据表
================

执行数据表重命名。

::

	$forge->renameTable('old_table_name', 'new_table_name');
	// 附加 ALTER TABLE old_table_name RENAME TO new_table_name

****************
修改数据表
****************

向数据表中添加列
==========================

**$forge->addColumn()**

使用 ``addColumn()`` 方法用于对现有数据表进行修改，它的参数和上面介绍的字段数组一样，并且可以用于无限数量的附加字段。

::

	$fields = [
		'preferences' => ['type' => 'TEXT']
	];
	$forge->addColumn('table_name', $fields);
	// 执行: ALTER TABLE table_name ADD preferences TEXT

如果你使用 `MySQL` 或 `CUBIRD` ，你可以使用 `AFTER` 和 `FIRST` 语句来为新添加的列指定位置。

例如::

	// 将新列放在 ``another_field`` 列之后:
	$fields = [
		'preferences' => ['type' => 'TEXT', 'after' => 'another_field']
	];

	// 将新列放在表定义的开头:
	$fields = [
		'preferences' => ['type' => 'TEXT', 'first' => TRUE]
	];

从数据表中删除列
==============================

**$forge->dropColumn()**

该语句用于从表中删除列。

::

	$forge->dropColumn('table_name', 'column_to_drop'); // 删除一列

该语句用于从表中删除多列。

::

    $forge->dropColumn('table_name', 'column_1,column_2'); // 通过提供用‘，’分隔的列名
    $forge->dropColumn('table_name', ['column_1', 'column_2']); // 通过提供数组的列名

修改数据表中的列
=============================

**$forge->modifyColumn()**

此方法的用法与 ``addColumn()`` 相同，只是它是更改现有列，而不是添加新列。为了更改名称，可以将“名称”键添加到字段定义数组中。

::

	$fields = [
		'old_name' => [
			'name' => 'new_name',
			'type' => 'TEXT',
		],
	];
	$forge->modifyColumn('table_name', $fields);
	// 附加 ALTER TABLE table_name CHANGE old_name new_name TEXT

***************
类参考
***************

.. php:class:: CodeIgniter\\Database\\Forge

	.. php:method:: addColumn($table[, $field = []])

		:param	string	$table: 要将列添加到的表名
		:param	array	$field: 列定义(s)
		:returns:	TRUE 为成功, FALSE 为失败
		:rtype:	bool

		向数据表中添加列。 用法：  参考 `向数据表中添加列`_.

	.. php:method:: addField($field)

		:param	array	$field: 要添加的字段定义
		:returns:	\CodeIgniter\Database\Forge instance (方法链)
		:rtype:	\CodeIgniter\Database\Forge

        将字段添加到将用于创建表的集合。 用法：  参考 `添加字段`_.

	.. php:method:: addKey($key[, $primary = FALSE[, $unique = FALSE]])

		:param	mixed	$key: 键字段或字段数组的名称
		:param	bool	$primary: 如果应该是主键或常规键，则设置为TRUE
		:param	bool	$unique: 如果应该是唯一键或常规键，则设置为TRUE
		:returns:	\CodeIgniter\Database\Forge instance (方法链)
		:rtype:	\CodeIgniter\Database\Forge

		将键添加到将用于创建表的集合。 用法：  参考 `添加键`_.

	.. php:method:: addPrimaryKey($key)

		:param	mixed	$key: 键字段或字段数组的名称
		:returns:	\CodeIgniter\Database\Forge instance (方法链)
		:rtype:	\CodeIgniter\Database\Forge

		将主键添加到将用于创建表的集合。 用法：  参考 `添加键`_.

	.. php:method:: addUniqueKey($key)

		:param	mixed	$key: 键字段或字段数组的名称
		:returns:	\CodeIgniter\Database\Forge instance (方法链)
		:rtype:	\CodeIgniter\Database\Forge

		将唯一键添加到将用于创建表的集合。 用法：  参考 `添加键`_.

	.. php:method:: createDatabase($dbName[, $ifNotExists = FALSE])

		:param	string	$db_name: 要创建的数据库的名称
		:param	string	$ifNotExists: 设置为TRUE以添加“IF NOT EXISTS”子句或检查数据库是否存在
		:returns:	TRUE 为成功, FALSE 为失败
		:rtype:	bool

		创建新数据库。 用法：  参考 `创建和删除数据库`_.

	.. php:method:: createTable($table[, $if_not_exists = FALSE[, array $attributes = []]])

		:param	string	$table: 要创建的表的名称
		:param	string	$if_not_exists: 设置为TRUE以添加“IF NOT EXISTS”子句
		:param	string	$attributes: 表属性的关联数组
		:returns:  Query object 为成功, FALSE 为失败
		:rtype:	mixed

		创建新表。 用法：  参考 `创建数据表`_.

	.. php:method:: dropColumn($table, $column_name)

		:param	string	$table: 表名
		:param	mixed	$column_names: “,”分隔的字符串或列名数组
		:returns:	TRUE 为成功, FALSE 为失败
		:rtype:	bool

		从表中删除单个或多个列。 用法：  参考 `从数据表中删除列`_.

	.. php:method:: dropDatabase($dbName)

		:param	string	$dbName: 要删除的数据库的名称
		:returns:	TRUE 为成功, FALSE 为失败
		:rtype:	bool

		删除数据库。 用法：  参考 `创建和删除数据库`_.

	.. php:method:: dropTable($table_name[, $if_exists = FALSE])

		:param	string	$table: 要删除的表的名称
		:param	string	$if_exists: 设置为TRUE以添加“IF EXISTS”子句
		:returns:	TRUE 为成功, FALSE 为失败
		:rtype:	bool

		删除数据表。 用法：  参考 `删除数据表`_.

	.. php:method:: modifyColumn($table, $field)

		:param	string	$table: 表名
		:param	array	$field: 列定义(s)
		:returns:	TRUE 为成功, FALSE 为失败
		:rtype:	bool

		修改数据表的列。 用法：  参考 `修改数据表中的列`_.

	.. php:method:: renameTable($table_name, $new_table_name)

		:param	string	$table: 当前表
		:param	string	$new_table_name: 表的新名称
		:returns:  Query object 为成功, FALSE 为失败
		:rtype:	mixed

		重命名表。 用法：  参考 `重命名数据表`_.
