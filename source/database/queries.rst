#######
查询
#######

.. contents::
    :local:
    :depth: 2

************
查询基础
************

常规查询
===============

要提交查询，请使用 **query** 函数::

	$db->query('YOUR QUERY HERE');

当运行读类型查询时，query()函数将返回数据库结果对象，您可以使用该查询 :doc:`显示结果 results <results>` 。运行写类型查询时，它仅根据成功或失败返回TRUE或FALSE。检索数据时，通常将查询分配给自己的变量，如下所示::

	$query = $db->query('YOUR QUERY HERE');

简化查询
==================

**simpleQuery** 方法是$db->query()方法的简化版本。它既不返回数据库结果集，也不设置查询计时器，编译绑定数据或存储查询以进行调试。它只是让您提交查询。大多数用户很少使用此功能。

它返回数据库驱动程序的“execute”函数返回的内容。对于诸如INSERT，DELETE或UPDATE语句之类的写类型查询（这实际上应用于此操作）的成功或失败和可获取结果的查询成功的资源/对象，通常为TRUE/FALSE。

::

	if ($db->simpleQuery('YOUR QUERY'))
	{
		echo "Success!";
	}
	else
	{
		echo "Query failed!";
	}

.. note:: PostgreSQL的 ``pg_exec()`` 函数（例如）总是成功返回资源，即使是写类型查询也是如此。因此，如果您要查找布尔值，请记住这一点。

***************************************
手动使用数据库前缀
***************************************

如果您已经配置了数据库前缀，并且想要在数据库名称之前添加一个表名称以供在本机SQL查询中使用，则可以使用以下命令::

	$db->prefixTable('tablename'); // 输出表前缀

如果出于任何原因想要以编程方式更改前缀而不需要创建新连接，则可以使用以下方法::

	$db->setPrefix('newprefix');
	$db->prefixTable('tablename'); // 输出表新前缀

您可以随时使用此方法获取当前前缀::
	
	$DBPrefix = $db->getPrefix();

**********************
保护标识符
**********************

在许多数据库中，建议保护表名和字段名，例如使用MySQL中的反引号。 **Query Builder查询会自动受到保护** ，但是如果您需要手动保护标识符，则可以使用::

	$db->protectIdentifiers('table_name');

.. important:: 尽管Query Builder将尽最大努力正确引用您提供的任何字段和表名。请注意，它并非旨在与任意用​​户输入一起使用。不要将未经消毒的用户数据提供给它。

假设您在数据库配置文件中指定了前缀，此函数还将在表中添加表前缀。要启用前缀设置第二个参数TRUE（布尔值）::

	$db->protectIdentifiers('table_name', TRUE);

****************
转义查询
****************

在将数据提交到数据库之前，先对其进行转义是非常好的安全措施。CodeIgniter有三种方法可以帮助您执行此操作:

#. **$db->escape()** 此函数确定数据类型，使其只能转义字符串数据。它还会自动在数据周围添加单引号，因此您不必:
   ::

	$sql = "INSERT INTO table (title) VALUES(".$db->escape($title).")";

#. **$db->escapeString()** 此函数转义传递给它的数据，而不管类型如何。大多数情况下，您将使用上述功能而不是此功能。使用如下函数:
   ::

	$sql = "INSERT INTO table (title) VALUES('".$db->escapeString($title)."')";

#. **$db->escapeLikeString()** 当在LIKE条件下使用字符串时，应使用此方法，以便字符串中的LIKE通配符（'％'，'_'）也可以正确转义。

::

        $search = '20% raise';
        $sql = "SELECT id FROM table WHERE column LIKE '%" .
        $db->escapeLikeString($search)."%' ESCAPE '!'";

.. important:: ``escapeLikeString()`` 方法使用'！' （感叹号）转义为 *LIKE* 条件的特殊字符。因为此方法转义了您自己用引号引起来的部分字符串，所以它无法自动为您添加 ``ESCAPE '!'`` 条件，因此您必须手动执行此操作。

**************
查询绑定
**************

通过系统为您组合查询，绑定使您能够简化查询语法。考虑以下示例::

	$sql = "SELECT * FROM some_table WHERE id = ? AND status = ? AND author = ?";
	$db->query($sql, [3, 'live', 'Rick']);

查询中的问号会自动替换为查询函数第二个参数中数组中的值。

绑定也适用于数组，它将转换为IN集::

	$sql = "SELECT * FROM some_table WHERE id IN ? AND status = ? AND author = ?";
	$db->query($sql, [[3, 6], 'live', 'Rick']);

结果查询将是::

	SELECT * FROM some_table WHERE id IN (3,6) AND status = 'live' AND author = 'Rick'

使用绑定的第二个好处是，这些值会自动转义以产生更安全的查询。您不必记住手动转义数据-引擎会为您自动进行转义。

命名绑定
==============

您可以使用绑定命名，而不用使用问号标记绑定值的位置，从而允许传入的值的键匹配查询中的占位符::

        $sql = "SELECT * FROM some_table WHERE id = :id: AND status = :status: AND author = :name:";
        $db->query($sql, [
                'id'     => 3,
                'status' => 'live',
                'name'   => 'Rick'
        ]);

.. note:: 查询中的每个名称都必须用冒号括起来。

***************
处理错误
***************

**$db->error();**

如果需要获取最近发生的错误，则error()方法将返回一个包含其代码和消息的数组。这是一个简单的示例::

	if ( ! $db->simpleQuery('SELECT `example_field` FROM `example_table`'))
	{
		$error = $db->error(); // 有 keys 'code' 和 'message'
	}

****************
准备好的查询
****************

大多数数据库引擎都支持某种形式的预处理语句，这些语句使您可以准备一次查询，然后使用新的数据集多次运行该查询。由于数据以与查询本身不同的格式传递到数据库，因此消除了SQL注入的可能性。当您需要多次运行同一查询时，它也可以更快。但是，对每个查询使用它都会对性能产生重大影响，因为您两次调用数据库的频率很高。由于Query Builder和数据库连接已经为您处理了转义数据，因此安全方面已为您处理。但是，有时候您需要能够通过运行一个准备好的语句或准备好的查询来优化查询。

准备查询
===================

使用 ``prepare()`` 方法可以轻松完成此操作。这需要一个参数，该参数是一个Closure，它返回查询对象。查询对象由任何“最终”类型的查询自动生成，包括 **insert**, **update**, **delete**, **replace**, 和 **get**。使用查询生成器运行查询可以最轻松地处理它。该查询实际上并未运行，并且这些值无关紧要，因为它们从未应用过，而是充当占位符。这将返回Prepared查询对象::

    $pQuery = $db->prepare(function($db)
    {
        return $db->table('user')
                   ->insert([
                        'name'    => 'x',
                        'email'   => 'y',
                        'country' => 'US'
                   ]);
    });

如果您不想使用Query Builder，则可以使用问号为值占位符手动创建查询对象::

    use CodeIgniter\Database\Query;

    $pQuery = $db->prepare(function($db)
    {
        $sql = "INSERT INTO user (name, email, country) VALUES (?, ?, ?)";

        return (new Query($db))->setQuery($sql);
    });

如果数据库需要在prepare语句阶段传递给它的选项数组，则可以在第二个参数中传递该数组::

    use CodeIgniter\Database\Query;

    $pQuery = $db->prepare(function($db)
    {
        $sql = "INSERT INTO user (name, email, country) VALUES (?, ?, ?)";

        return (new Query($db))->setQuery($sql);
    }, $options);

执行查询
===================

准备好查询后，就可以使用 ``execute()`` 方法实际运行查询。您可以在查询参数中根据需要传入任意多个变量。您传递的参数数量必须与查询中的占位符数量匹配。还必须按照占位符在原始查询中出现的顺序传递它们::

    // 准备查询
    $pQuery = $db->prepare(function($db)
    {
        return $db->table('user')
                   ->insert([
                        'name'    => 'x',
                        'email'   => 'y',
                        'country' => 'US'
                   ]);
    });

    // 收集数据
    $name    = 'John Doe';
    $email   = 'j.doe@example.com';
    $country = 'US';

    // 运行查询
    $results = $pQuery->execute($name, $email, $country);

这将返回一个标准 :doc:`结果集 </database/results>`.

其他方法
=============

除了这两种主要方法外，准备好的查询对象还具有以下方法:

**close()**

虽然PHP在关闭数据库中所有打开的语句方面做得很好，但是在完成后关闭准备好的语句始终是一个好主意::

    $pQuery->close();

**getQueryString()**

这将使准备好的查询作为字符串返回。

**hasError()**

如果最后一次execute()调用创建任何错误，则返回布尔值true/false。

**getErrorCode()**
**getErrorMessage()**

如果遇到任何错误，则可以使用这些方法来检索错误代码和字符串。

**************************
使用查询对象
**************************

在内部，所有查询都作为 ``\CodeIgniter\Database\Query`` 实例进行处理和存储。此类负责绑定参数，否则准备查询以及存储有关其查询的性能数据。

**getLastQuery()**

当您只需要检索最后一个Query对象时，请使用getLastQuery()方法::

	$query = $db->getLastQuery();
	echo (string)$query;

查询类
===============

每个查询对象存储有关查询本身的几条信息。时间线功能部分使用了此功能，但也可供您使用。

**getQuery()**

完成所有处理后，返回最终查询。这是发送到数据库的确切查询::

	$sql = $query->getQuery();

可以通过将Query对象转换为字符串来检索相同的值::

	$sql = (string)$query;

**getOriginalQuery()**

返回传递给对象的原始SQL。这将没有任何绑定，也没有换掉前缀，等等::

	$sql = $query->getOriginalQuery();

**hasError()**

如果在执行此查询期间遇到错误，则此方法将返回true::

	if ($query->hasError())
	{
		echo 'Code: '. $query->getErrorCode();
		echo 'Error: '. $query->getErrorMessage();
	}

**isWriteType()**

如果查询被确定为写类型查询（即INSERT，UPDATE，DELETE等），则返回true::

	if ($query->isWriteType())
	{
		... do something
	}

**swapPrefix()**

在最终SQL中，用另一个值替换一个表前缀。第一个参数是要替换的原始前缀，第二个参数是要替换为的值::

	$sql = $query->swapPrefix('ci3_', 'ci4_');

**getStartTime()**

获取以微秒为单位执行查询的时间::

	$microtime = $query->getStartTime();

**getDuration()**

返回带有查询持续时间的浮点数，以微秒为单位::

	$microtime = $query->getDuration();
