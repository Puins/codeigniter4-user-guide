###########################
连接你的数据库
###########################

您可以通过在需要的任何函数中添加此代码行，或在类构造函数中添加此代码行以使数据库在该类中全局可用，来连接到数据库。

::

	$db = \Config\Database::connect();

如果以上函数在第一个参数中 **不** 包含任何信息，它将连接到数据库配置文件中指定的默认组。对于大多数人来说，这是首选的使用方法。

存在一种方便的方法，它纯粹是围绕上述行的封装，为您提供方便::

    $db = db_connect();

可用参数
--------------------

#. 数据库组名称，必须与配置类的属性名称匹配的字符串。默认值为 ``$config->defaultGroup``。
#. TRUE/FALSE (boolean). 是否返回共享连接（请参阅下面的连接到多个数据库）。

手动连接到数据库
---------------------------------

可以选择使用此函数的第一个参数从配置文件中指定特定的数据库组。例子:

要从配置文件中选择特定的组，您可以执行以下操作::

	$db = \Config\Database::connect('group_name');

其中group_name是配置文件中连接组的名称。

多个连接到同一数据库
-------------------------------------

默认情况下，``connect()`` 方法每次都会返回相同的数据库连接实例。如果需要与同一个数据库建立单独的连接，请把 ``false`` 作为第二个参数发送::

	$db = \Config\Database::connect('group_name', false);

连接到多个数据库
================================

如果需要同时连接到多个数据库，则可以执行以下操作::

	$db1 = \Config\Database::connect('group_one');
	$db  = \Config\Database::connect('group_two');

.. note:: 将单词“group_one”和“group_two”更改为您要连接的特定组名。

.. note:: 如果只需要在同一连接上使用其他数据库，则无需创建单独的数据库配置。您可以根据需要切换到其他数据库，如下所示::

	$db->setDatabase($database2_name);

连接自定义设置
===============================

您可以传入一组数据库设置而不是组名来获得使用您的自定义设置的连接。传入的数组必须与配置文件中定义的组的格式相同::

    $custom = [
		'DSN'      => '',
		'hostname' => 'localhost',
		'username' => '',
		'password' => '',
		'database' => '',
		'DBDriver' => 'MySQLi',
		'DBPrefix' => '',
		'pConnect' => false,
		'DBDebug'  => (ENVIRONMENT !== 'production'),
		'cacheOn'  => false,
		'cacheDir' => '',
		'charset'  => 'utf8',
		'DBCollat' => 'utf8_general_ci',
		'swapPre'  => '',
		'encrypt'  => false,
		'compress' => false,
		'strictOn' => false,
		'failover' => [],
		'port'     => 3306,
	];
    $db = \Config\Database::connect($custom);


重新连接/使连接保持活动状态
===========================================

如果在执行一些繁重的PHP举动（例如，处理映像）时超出了数据库服务器的空闲超时，则应考虑在发送进一步的查询之前使用reconnect()方法对服务器进行ping操作，从而可以正常地保持连接活着还是重新建立它。

.. important:: 如果使用MySQLi数据库驱动程序，则reconnect()方法不会ping服务器，但会关闭连接，然后再次连接。

::

	$db->reconnect();

手动关闭连接
===============================

虽然CodeIgniter可以智能地关闭数据库连接，但是您可以显式关闭连接。

::

	$db->close();
