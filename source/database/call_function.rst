#####################
自定义函数调用
#####################

$db->callFunction();
============================

此函数使您能够以平台无关的方式调用CodeIgniter中未原生包含的PHP数据库函数。例如，假设您要调用mysql_get_client_info()函数，而CodeIgniter本身 **不** 支持该函数。您可以这样做::

	$db->callFunction('get_client_info');

您必须在第一个参数中提供 **不** 带 ``mysql_`` 前缀的函数名称。根据当前使用的数据库驱动程序自动添加前缀。这使您可以在不同的数据库平台上运行相同的功能。显然，并非所有函数调用在平台之间都是相同的，因此就可移植性而言，此函数的有用性受到限制。

您正在调用的函数所需的任何参数都将添加到第二个参数中。

::

	$db->callFunction('some_function', $param1, $param2, etc..);

通常，您将需要提供数据库连接ID或数据库结果ID。可以使用以下命令访问连接ID::

	$db->connID;

可以从结果对象中访问结果ID，如下所示::

	$query = $db->query("SOME QUERY");

	$query->resultID;