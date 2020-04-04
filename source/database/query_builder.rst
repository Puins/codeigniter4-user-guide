###################
Query Builder 类
###################

CodeIgniter使您可以访问Query Builder类。此模式允许使用最少的脚本在数据库中检索，插入和更新信息。在某些情况下，只需执行一两行代码即可执行数据库操作。CodeIgniter不需要每个数据库表都是其自己的类文件。相反，它提供了更简化的界面。

除了简单之外，使用Query Builder功能的主要好处是它允许您创建独立于数据库的应用程序，因为查询语法是由每个数据库适配器生成的。它还允许更安全的查询，因为这些值会由系统自动转义。

.. contents::
    :local:
    :depth: 2

*************************
加载 Query Builder
*************************

通过数据库连接上的方法 ``table()`` 加载Query Builder。这将为您设置 ``FROM`` 查询的一部分，并返回Query Builder类的新实例::

    $db      = \Config\Database::connect();
    $builder = $db->table('users');

Query Builder仅在您明确请求类时才加载到内存中，因此默认情况下不使用任何资源。

**************
选择数据
**************

以下函数允许您构建SQL的 **SELECT** 语句。

**$builder->get()**

运行选择查询并返回结果。可以单独用于检索表中的所有记录::

    $builder = $db->table('mytable');
    $query   = $builder->get();  // 产生: SELECT * FROM mytable

第一个和第二个参数使您可以设置limit和offset子句::

	$query = $builder->get(10, 20);

	// 执行: SELECT * FROM mytable LIMIT 20, 10
	// (在 MySQL。 其他数据库的语法略有不同)

您会注意到上述函数已分配给名为$query的变量，该变量可用于显示结果::

	$query = $builder->get();

	foreach ($query->getResult() as $row)
	{
		echo $row->title;
	}

请访问 :doc:`结果函数 <results>` 页面，以获取有关结果生成的完整讨论。

**$builder->getCompiledSelect()**

像 **$builder->get()** 一样编译选择查询，但不运行 **查询**。此方法只是将SQL查询作为字符串返回。

示例::

	$sql = $builder->getCompiledSelect();
	echo $sql;

	// 打印字符串: SELECT * FROM mytable

第一个参数使您可以设置是否重置query builder查询（默认情况下，它将重置，就像使用 `$builder->get()` 一样）::

	echo $builder->limit(10,20)->getCompiledSelect(false);

	// 打印字符串: SELECT * FROM mytable LIMIT 20, 10
	// (in MySQL. Other databases have slightly different syntax)

	echo $builder->select('title, content, date')->getCompiledSelect();

	// 打印字符串: SELECT title, content, date FROM mytable LIMIT 20, 10

在上面的示例中要注意的关键是，第二个查询没有使用 **$builder->from()** 并且没有将表名传递给第一个参数。产生此结果的原因是因为尚未使用 **$builder->get()** 执行查询，该查询会重置值或直接使用 **$builder->resetQuery()** 进行重置。

**$builder->getWhere()**

与 ``get()`` 函数相同，不同之处在于它允许您在第一个参数中添加“where”子句，而不是使用 ``db->where()`` 函数::

	$query = $builder->getWhere(['id' => $id], $limit, $offset);

请阅读以下有关 `where` 函数的详细信息。

**$builder->select()**

允许您编写查询的SELECT部分::

	$builder->select('title, content, date');
	$query = $builder->get();

	// 执行: SELECT title, content, date FROM mytable

.. note:: 如果要从表中选择所有(*)，则无需使用此功能。省略时，CodeIgniter假定您希望选择所有字段并自动添加 ``SELECT *``。

``$builder->select()`` 接受可选的第二个参数。如果将其设置为FALSE，则CodeIgniter不会尝试保护您的字段或表名。如果您需要复合选择语句，其中的字段自动转义可能会破坏它们，这将很有用。

::

	$builder->select('(SELECT SUM(payments.amount) FROM payments WHERE payments.invoice_id=4) AS amount_paid', FALSE);
	$query = $builder->get();

**$builder->selectMax()**

该方法用于编写查询语句中的 ``SELECT MAX(field)`` 部分，你 可以使用第二个参数重命名结果字段。

::

	$builder->selectMax('age');
	$query = $builder->get();  // 产生: SELECT MAX(age) as age FROM mytable

	$builder->selectMax('age', 'member_age');
	$query = $builder->get(); // 产生: SELECT MAX(age) as member_age FROM mytable

**$builder->selectMin()**

该方法用于编写查询语句中的 ``SELECT MIN(field)`` 部分，和 ``selectMax()`` 方法一样， 你可以使用第二个参数（可选）重命名结果字段。

::

	$builder->selectMin('age');
	$query = $builder->get(); // 产生: SELECT MIN(age) as age FROM mytable

**$builder->selectAvg()**

该方法用于编写查询语句中的 ``SELECT AVG(field)`` 部分，和 ``selectMax()`` 方法一样， 你可以使用第二个参数（可选）重命名结果字段。

::

	$builder->selectAvg('age');
	$query = $builder->get(); // 产生: SELECT AVG(age) as age FROM mytable

**$builder->selectSum()**

该方法用于编写查询语句中的 ``SELECT SUM(field)`` 部分，和 ``selectMax()`` 方法一样， 你可以使用第二个参数（可选）重命名结果字段。

::

	$builder->selectSum('age');
	$query = $builder->get(); // 产生: SELECT SUM(age) as age FROM mytable

**$builder->selectCount()**

该方法用于编写查询语句中的 ``SELECT COUNT(field)`` 部分，和 ``selectMax()`` 方法一样， 你可以使用第二个参数（可选）重命名结果字段。

.. note:: 与 ``groupBy()`` 结合使用时，此方法特别有用。要计算结果，通常参见 ``countAll()`` 或 ``countAllResults()``。

::

	$builder->selectCount('age');
	$query = $builder->get(); // 产生: SELECT COUNT(age) as age FROM mytable

**$builder->from()**

允许您编写查询的FROM部分::

	$builder->select('title, content, date');
	$builder->from('mytable');
	$query = $builder->get();  // 产生: SELECT title, content, date FROM mytable

.. note:: 如前所示，您查询的FROM部分可以在$db->table()函数中指定。对from()的其他调用会将更多表添加到查询的FROM部分。

**$builder->join()**

允许您编写查询的JOIN部分::

    $builder->db->table('blog');
    $builder->select('*');
    $builder->join('comments', 'comments.id = blogs.id');
    $query = $builder->get();

    // 产生:
    // SELECT * FROM blogs JOIN comments ON comments.id = blogs.id

如果在一个查询中需要多个join，则可以进行多个函数调用。

如果需要特定类型的JOIN，则可以通过函数的第三个参数进行指定。选项包括：left, right, outer, inner, left outer, 和 right outer。

::

	$builder->join('comments', 'comments.id = blogs.id', 'left');
	// 产生: LEFT JOIN comments ON comments.id = blogs.id

*************************
查找特定数据
*************************

**$builder->where()**

此函数使您可以使用以下四种方法之一来设置 **WHERE** 子句:

.. note:: 传递给此函数的所有值都将自动转义，从而产生更安全的查询。

#. **简单的 key/value 方法:**

	::

		$builder->where('name', $name); // 产生: WHERE name = 'Joe'

	请注意，为您添加了等号。

	如果使用多个函数调用，它们将用AND链接在一起:

	::

		$builder->where('name', $name);
		$builder->where('title', $title);
		$builder->where('status', $status);
		// WHERE name = 'Joe' AND title = 'boss' AND status = 'active'

#. **自定义的 key/value 方法:**

	为了控制比较，你可以在第一个参数中包含一个运算符:

	::

		$builder->where('name !=', $name);
		$builder->where('id <', $id); // 产生: WHERE name != 'Joe' AND id < 45

#. **关联数组方法:**

	::

		$array = ['name' => $name, 'title' => $title, 'status' => $status];
		$builder->where($array);
		// 产生: WHERE name = 'Joe' AND title = 'boss' AND status = 'active'

	您也可以使用此方法包括自己的运算符:

	::

		$array = ['name !=' => $name, 'id <' => $id, 'date >' => $date];
		$builder->where($array);

#. **自定义字符串:**
	您可以手动编写自己的子句::

		$where = "name='Joe' AND status='boss' OR status='active'";
		$builder->where($where);

    ``$builder->where()`` 接受可选的第三个参数。如果将其设置为FALSE，则CodeIgniter不会尝试保护您的字段或表名。

    ::

        $builder->where('MATCH (field) AGAINST ("value")', NULL, FALSE);

#. **子查询:**
    您可以使用匿名函数创建子查询。

    ::

        $builder->where('advance_amount <', function(BaseBuilder $builder) {
            return $builder->select('MAX(advance_amount)', false)->from('orders')->where('id >', 2);
        });
        // 产生: WHERE "advance_amount" < (SELECT MAX(advance_amount) FROM "orders" WHERE "id" > 2)

**$builder->orWhere()**

此函数与上述函数相同，不同之处在于多个实例通过OR进行连接

    ::

	$builder->where('name !=', $name);
	$builder->orWhere('id >', $id);  // 产生: WHERE name != 'Joe' OR id > 50

**$builder->whereIn()**

生成 ``WHERE field IN ('item', 'item')`` SQL查询，多个子句之间使用 AND 连接（如果适用）

    ::

        $names = ['Frank', 'Todd', 'James'];
        $builder->whereIn('username', $names);
        // 产生: WHERE username IN ('Frank', 'Todd', 'James')

您可以使用子查询而不是值数组。

    ::

        $builder->whereIn('id', function(BaseBuilder $builder) {
            return $builder->select('job_id')->from('users_jobs')->where('user_id', 3);
        });
        // 产生: WHERE "id" IN (SELECT "job_id" FROM "users_jobs" WHERE "user_id" = 3)

**$builder->orWhereIn()**

生成 ``WHERE field IN ('item', 'item')`` SQL查询，多个子句之间使用 OR 连接（如果适用）

    ::

        $names = ['Frank', 'Todd', 'James'];
        $builder->orWhereIn('username', $names);
        // 产生: OR username IN ('Frank', 'Todd', 'James')

您可以使用子查询而不是值数组。

    ::

        $builder->orWhereIn('id', function(BaseBuilder $builder) {
            return $builder->select('job_id')->from('users_jobs')->where('user_id', 3);
        });

        // 产生: OR "id" IN (SELECT "job_id" FROM "users_jobs" WHERE "user_id" = 3)

**$builder->whereNotIn()**

生成 ``WHERE field NOT IN ('item', 'item')`` SQL查询，多个子句之间使用 AND 连接（如果适用）

    ::

        $names = ['Frank', 'Todd', 'James'];
        $builder->whereNotIn('username', $names);
        // 产生: WHERE username NOT IN ('Frank', 'Todd', 'James')

您可以使用子查询而不是值数组。

    ::

        $builder->whereNotIn('id', function(BaseBuilder $builder) {
            return $builder->select('job_id')->from('users_jobs')->where('user_id', 3);
        });

        // 产生: WHERE "id" NOT IN (SELECT "job_id" FROM "users_jobs" WHERE "user_id" = 3)


**$builder->orWhereNotIn()**

生成 ``WHERE field NOT IN ('item', 'item')`` SQL查询，多个子句之间使用 OR 连接（如果适用）

    ::

        $names = ['Frank', 'Todd', 'James'];
        $builder->orWhereNotIn('username', $names);
        // 产生: OR username NOT IN ('Frank', 'Todd', 'James')

您可以使用子查询而不是值数组。

    ::

        $builder->orWhereNotIn('id', function(BaseBuilder $builder) {
            return $builder->select('job_id')->from('users_jobs')->where('user_id', 3);
        });

        // 产生: OR "id" NOT IN (SELECT "job_id" FROM "users_jobs" WHERE "user_id" = 3)

************************
查找相似的数据
************************

**$builder->like()**

此方法使您可以生成 **LIKE** 子句，在进行搜索时非常有用。

.. note:: 传递给此方法的所有值都将自动转义。

.. note:: 通过将方法的第五个参数 ``true`` 传递给方法，可以强制所有 ``like*`` 方法变体执行不区分大小写的搜索。这将使用平台特有的功能（否则可用）将强制将值转换为小写，即 ``WHERE LOWER(column) LIKE '%search%'``。这可能需要建立索引 ``LOWER(column)`` 而不是有效的索引 ``column``。

#. **简单的 key/value 方法:**

	::

		$builder->like('title', 'match');
		// 产生: WHERE `title` LIKE '%match%' ESCAPE '!'

	如果您使用多个方法调用，则它们将用AND链接在一起::

		$builder->like('title', 'match');
		$builder->like('body', 'match');
		// WHERE `title` LIKE '%match%' ESCAPE '!' AND  `body` LIKE '%match% ESCAPE '!'

	如果要控制通配符（％）的放置位置，则可以使用可选的第三个参数。您的选择是“之前”，“之后”和“两者”（默认设置）。

	::

		$builder->like('title', 'match', 'before');	// 产生: WHERE `title` LIKE '%match' ESCAPE '!'
		$builder->like('title', 'match', 'after');	// 产生: WHERE `title` LIKE 'match%' ESCAPE '!'
		$builder->like('title', 'match', 'both');	// 产生: WHERE `title` LIKE '%match%' ESCAPE '!'

#. **关联数组方法:**

	::

		$array = ['title' => $match, 'page1' => $match, 'page2' => $match];
		$builder->like($array);
		// WHERE `title` LIKE '%match%' ESCAPE '!' AND  `page1` LIKE '%match%' ESCAPE '!' AND  `page2` LIKE '%match%' ESCAPE '!'

**$builder->orLike()**

此方法与上述方法相同，不同之处在于多个实例通过OR连接::

	$builder->like('title', 'match'); $builder->orLike('body', $match);
	// WHERE `title` LIKE '%match%' ESCAPE '!' OR  `body` LIKE '%match%' ESCAPE '!'

**$builder->notLike()**

此方法与 ``like()`` 相同，除了它生成 ``NOT LIKE`` 语句::

	$builder->notLike('title', 'match');	// WHERE `title` NOT LIKE '%match% ESCAPE '!'

**$builder->orNotLike()**

此方法与 ``notLike()`` 相同，不同之处在于多个实例通过OR连接::

	$builder->like('title', 'match');
	$builder->orNotLike('body', 'match');
	// WHERE `title` LIKE '%match% OR  `body` NOT LIKE '%match%' ESCAPE '!'

**$builder->groupBy()**

允许您编写查询的GROUP BY部分::

	$builder->groupBy("title"); // 产生: GROUP BY title

您还可以传递多个值的数组::

	$builder->groupBy(["title", "date"]);  // 产生: GROUP BY title, date

**$builder->distinct()**

将 ``DISTINCT`` 关键字添加到查询中

::

	$builder->distinct();
	$builder->get(); // 产生: SELECT DISTINCT * FROM mytable

**$builder->having()**

允许您编写查询的HAVING部分。有2种可能的语法，1个参数或2::

	$builder->having('user_id = 45');  // 产生: HAVING user_id = 45
	$builder->having('user_id',  45);  // 产生: HAVING user_id = 45

您还可以传递多个值的数组::

	$builder->having(['title =' => 'My Title', 'id <' => $id]);
	// 产生: HAVING title = 'My Title', id < 45

如果使用的是CodeIgniter对其进行转义查询的数据库，则可以通过传递可选的第三个参数并将其设置为FALSE来防止转义内容。

::

	$builder->having('user_id',  45);  // 产生: HAVING `user_id` = 45 in some databases such as MySQL
	$builder->having('user_id',  45, FALSE);  // 产生: HAVING user_id = 45

**$builder->orHaving()**

与having()相同，仅用“OR”分隔多个子句。

**$builder->havingIn()**

生成 ``HAVING field IN ('item', 'item')`` SQL查询，多个子句之间使用 AND 连接（如果适用）

    ::

        $groups = [1, 2, 3];
        $builder->havingIn('group_id', $groups);
        // 产生: HAVING group_id IN (1, 2, 3)

您可以使用子查询而不是值数组。

    ::

        $builder->havingIn('id', function(BaseBuilder $builder) {
            return $builder->select('user_id')->from('users_jobs')->where('group_id', 3);
        });
        // 产生: HAVING "id" IN (SELECT "user_id" FROM "users_jobs" WHERE "group_id" = 3)

**$builder->orHavingIn()**

生成 ``HAVING field IN ('item', 'item')`` SQL查询，多个子句之间使用 OR 连接（如果适用）

    ::

        $groups = [1, 2, 3];
        $builder->orHavingIn('group_id', $groups);
        // 产生: OR group_id IN (1, 2, 3)

您可以使用子查询而不是值数组。

    ::

        $builder->orHavingIn('id', function(BaseBuilder $builder) {
            return $builder->select('user_id')->from('users_jobs')->where('group_id', 3);
        });

        // 产生: OR "id" IN (SELECT "user_id" FROM "users_jobs" WHERE "group_id" = 3)

**$builder->havingNotIn()**

生成 ``HAVING field NOT IN ('item', 'item')`` SQL查询，多个子句之间使用 AND 连接（如果适用）

    ::

        $groups = [1, 2, 3];
        $builder->havingNotIn('group_id', $groups);
        // 产生: HAVING group_id NOT IN (1, 2, 3)

您可以使用子查询而不是值数组。

    ::

        $builder->havingNotIn('id', function(BaseBuilder $builder) {
            return $builder->select('user_id')->from('users_jobs')->where('group_id', 3);
        });

        // 产生: HAVING "id" NOT IN (SELECT "user_id" FROM "users_jobs" WHERE "group_id" = 3)


**$builder->orHavingNotIn()**

生成 ``HAVING field NOT IN ('item', 'item')`` SQL查询，多个子句之间使用 OR 连接（如果适用）

    ::

        $groups = [1, 2, 3];
        $builder->havingNotIn('group_id', $groups);
        // 产生: OR group_id NOT IN (1, 2, 3)

您可以使用子查询而不是值数组。

    ::

        $builder->orHavingNotIn('id', function(BaseBuilder $builder) {
            return $builder->select('user_id')->from('users_jobs')->where('group_id', 3);
        });

        // 产生: OR "id" NOT IN (SELECT "user_id" FROM "users_jobs" WHERE "group_id" = 3)

**$builder->havingLike()**

此方法使您可以为HAVING部分或查询生成 **LIKE** 子句，这对于进行搜索很有用。

.. note:: 传递给此方法的所有值都将自动转义。

.. note:: 通过将方法的第五个参数 ``true`` 传递给方法，可以强制所有 ``havingLike*`` 方法变体执行不区分大小写的搜索。这将使用平台特有的功能（否则可用）将强制将值转换为小写，即 ``HAVING LOWER(column) LIKE '%search%'``。这可能需要建立索引 ``LOWER(column)`` 而不是有效的索引 ``column``。

#. **简单的 key/value 方法:**

	::

		$builder->havingLike('title', 'match');
		// 产生: HAVING `title` LIKE '%match%' ESCAPE '!'

	如果您使用多个方法调用，则它们将用AND链接在一起::

		$builder->havingLike('title', 'match');
		$builder->havingLike('body', 'match');
		// HAVING `title` LIKE '%match%' ESCAPE '!' AND  `body` LIKE '%match% ESCAPE '!'

	如果要控制通配符（％）的放置位置，则可以使用可选的第三个参数。您的选择是“之前”，“之后”和“两者”（默认设置）。

	::

		$builder->havingLike('title', 'match', 'before');	// 产生: HAVING `title` LIKE '%match' ESCAPE '!'
		$builder->havingLike('title', 'match', 'after');	// 产生: HAVING `title` LIKE 'match%' ESCAPE '!'
		$builder->havingLike('title', 'match', 'both');	// 产生: HAVING `title` LIKE '%match%' ESCAPE '!'

#. **关联数组方法:**

	::

		$array = ['title' => $match, 'page1' => $match, 'page2' => $match];
		$builder->havingLike($array);
		// HAVING `title` LIKE '%match%' ESCAPE '!' AND  `page1` LIKE '%match%' ESCAPE '!' AND  `page2` LIKE '%match%' ESCAPE '!'

**$builder->orHavingLike()**

此方法与上述方法相同，不同之处在于多个实例通过OR连接::

	$builder->havingLike('title', 'match'); $builder->orHavingLike('body', $match);
	// HAVING `title` LIKE '%match%' ESCAPE '!' OR  `body` LIKE '%match%' ESCAPE '!'

**$builder->notHavingLike()**

此方法与 ``havingLike()`` 相同，除了它生成 ``NOT LIKE`` 语句::

	$builder->notHavingLike('title', 'match');	// HAVING `title` NOT LIKE '%match% ESCAPE '!'

**$builder->orNotHavingLike()**

此方法与 ``notHavingLike()`` 相同，不同之处在于多个实例通过OR连接::

	$builder->havingLike('title', 'match');
	$builder->orNotHavingLike('body', 'match');
	// HAVING `title` LIKE '%match% OR  `body` NOT LIKE '%match%' ESCAPE '!'

****************
排序结果
****************

**$builder->orderBy()**

使您可以设置ORDER BY子句。

第一个参数包含您要作为排序依据的列的名称。

第二个参数使您可以设置结果的方向。选项包括 **ASC**, **DESC** 和 **RANDOM**。

::

	$builder->orderBy('title', 'DESC');
	// 产生: ORDER BY `title` DESC

您还可以在第一个参数中传递自己的字符串::

	$builder->orderBy('title DESC, name ASC');
	// 产生: ORDER BY `title` DESC, `name` ASC

或者，如果需要多个字段，则可以进行多个函数调用。

::

	$builder->orderBy('title', 'DESC');
	$builder->orderBy('name', 'ASC');
	// 产生: ORDER BY `title` DESC, `name` ASC

如果选择 **RANDOM** 方向选项，除非指定数字种子值，否则第一个参数将被忽略。

::

	$builder->orderBy('title', 'RANDOM');
	// 产生: ORDER BY RAND()

	$builder->orderBy(42, 'RANDOM');
	// 产生: ORDER BY RAND(42)

.. note:: Oracle 暂时还不支持随机排序，会默认使用ASC。

****************************
限制或计数结果
****************************

**$builder->limit()**

让您限制查询希望返回的行数::

	$builder->limit(10);  // 产生: LIMIT 10

第二个参数使您可以设置结果偏移量。

::

	$builder->limit(10, 20);  // 产生: LIMIT 20, 10 (in MySQL. Other databases have slightly different syntax)


**$builder->countAllResults()**

允许您确定特定Query Builder查询中的行数。查询将接受Query Builder的限流器，例如 ``where()``, ``orWhere()``, ``like()``, ``orLike()``，等。示例::

	echo $builder->countAllResults();  // Produces an integer, like 25
	$builder->like('title', 'match');
	$builder->from('my_table');
	echo $builder->countAllResults(); // Produces an integer, like 17

但是，此方法还会重置您可能传递给 ``select()`` 的任何字段值。如果需要保留它们，可以将 ``FALSE`` 作为第一个参数传递。

	echo $builder->countAllResults(false); // Produces an integer, like 17

**$builder->countAll()**

允许您确定特定表中的行数。
示例::

	echo $builder->countAll();  // Produces an integer, like 25

与countAllResult方法一样，此方法也会重置您可能已经传递给 ``select()`` 的所有字段值。如果需要保留它们，可以将FALSE作为第一个参数传递。

**************
查询分组
**************

查询分组使您可以通过将WHERE子句括在括号中来创建它们。这将允许您使用复杂的WHERE子句创建查询。支持嵌套组。示例::

	$builder->select('*')->from('my_table')
		->groupStart()
			->where('a', 'a')
			->orGroupStart()
				->where('b', 'b')
				->where('c', 'c')
			->groupEnd()
		->groupEnd()
		->where('d', 'd')
	->get();

	// 生成:
	// SELECT * FROM (`my_table`) WHERE ( `a` = 'a' OR ( `b` = 'b' AND `c` = 'c' ) ) AND `d` = 'd'

.. note:: 条件组必须要配对，确保每个 groupStart() 方法都有一个 groupEnd() 方法与之配对。

**$builder->groupStart()**

通过在查询的WHERE子句中添加左括号来启动新组。

**$builder->orGroupStart()**

通过在查询的WHERE子句中添加左括号并以“OR”作为前缀来启动新组。

**$builder->notGroupStart()**

通过在查询的WHERE子句中添加开头括号并以'NOT'作为前缀来开始新组。

**$builder->orNotGroupStart()**

通过在查询的WHERE子句中添加开头括号并以'OR NOT'作为前缀来开始新组。

**$builder->groupEnd()**

通过在查询的WHERE子句中添加右括号来结束当前组。

**$builder->groupHavingStart()**

通过在查询的HAVING子句中添加左括号来启动新组。

**$builder->orGroupHavingStart()**

通过在查询的HAVING子句中添加左括号并以“OR”作为前缀来启动新组。

**$builder->notGroupHavingStart()**

通过在查询的HAVING子句中添加开头括号并以'NOT'作为前缀来开始新组。

**$builder->orNotGroupHavingStart()**

通过在查询的HAVING子句中添加开头括号并以'OR NOT'作为前缀来启动新组。

**$builder->groupHavingEnd()**

通过在查询的HAVING子句中添加右括号来结束当前组。

**************
插入数据
**************

**$builder->insert()**

根据您提供的数据生成插入字符串，然后运行查询。您可以将数组或对象传递给函数。这是使用数组的示例::

	$data = [
		'title' => 'My title',
		'name'  => 'My Name',
		'date'  => 'My date'
	];

	$builder->insert($data);
	// 产生: INSERT INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date')

第一个参数是值的关联数组。

这是使用对象的示例::

	/*
	class Myclass {
		public $title   = 'My Title';
		public $content = 'My Content';
		public $date    = 'My Date';
	}
	*/

	$object = new Myclass;
	$builder->insert($object);
	// 产生: INSERT INTO mytable (title, content, date) VALUES ('My Title', 'My Content', 'My Date')

第一个参数是一个对象。

.. note:: 所有值都会自动转义，从而产生更安全的查询。

**$builder->ignore()**

根据您提供的数据生成插入忽略字符串，然后运行查询。因此，如果已经存在具有相同主键的条目，则不会插入查询。您可以选择将布尔值传递给该函数。这是使用上述示例的数组的示例::

	$data = [
		'title' => 'My title',
		'name'  => 'My Name',
		'date'  => 'My date'
	];

	$builder->ignore(true)->insert($data);
	// 产生: INSERT OR IGNORE INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date')


**$builder->getCompiledInsert()**

像 ``$builder->insert()`` 一样编译插入查询，但 **不运行** 查询。此方法只是将SQL查询作为字符串返回。

示例::

	$data = [
		'title' => 'My title',
		'name'  => 'My Name',
		'date'  => 'My date'
	];

	$sql = $builder->set($data)->getCompiledInsert('mytable');
	echo $sql;

	// 产生字符串: INSERT INTO mytable (`title`, `name`, `date`) VALUES ('My title', 'My name', 'My date')

第二个参数使您可以设置是否重置query builder查询（默认情况下，它将像 ``$builder->insert()`` 一样）::

	echo $builder->set('title', 'My Title')->getCompiledInsert('mytable', FALSE);

	// 产生字符串: INSERT INTO mytable (`title`) VALUES ('My Title')

	echo $builder->set('content', 'My Content')->getCompiledInsert();

	// 产生字符串: INSERT INTO mytable (`title`, `content`) VALUES ('My Title', 'My Content')

在上面的示例中要注意的关键是第二个查询没有使用 `$builder->from()`，也没有将表名传递给第一个参数。之所以起作用，是因为尚未使用`$builder->insert()` 执行查询，该查询会重置值或直接使用 `$builder->resetQuery()` 进行重置 。

.. note:: 此方法不适用于批量插入。

**$builder->insertBatch()**

根据您提供的数据生成插入字符串，然后运行查询。您可以将 **数组** 或 **对象** 传递给函数。这是使用数组的示例::

	$data = [
		[
			'title' => 'My title',
			'name'  => 'My Name',
			'date'  => 'My date'
		],
		[
			'title' => 'Another title',
			'name'  => 'Another Name',
			'date'  => 'Another date'
		]
	];

	$builder->insertBatch($data);
	// 产生: INSERT INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date'),  ('Another title', 'Another name', 'Another date')

第一个参数是值的关联数组。

.. note:: 所有值都会自动转义，从而产生更安全的查询。

*************
更新数据
*************

**$builder->replace()**

此方法使用 *PRIMARY* 和 *UNIQUE* 键作为确定因素，执行REPLACE语句，该语句基本上是（可选）DELETE + INSERT的SQL标准。在我们的例子中，它可以使你免于需要实现与不同的组合复杂的逻辑 ``select()``, ``update()``, ``delete()`` 和 ``insert()`` 的调用。

示例::

	$data = [
		'title' => 'My title',
		'name'  => 'My Name',
		'date'  => 'My date'
	];

	$builder->replace($data);

	// 执行: REPLACE INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date')

在上面的示例中，如果假设 *title* 字段是我们的主键，则如果包含“ My title”作为 *title* 值的行，则该行将被删除，并用我们的新行数据替换。

``set()`` 也允许使用该方法，并且所有字段都会自动转义，就像使用 ``insert()`` 一样。

**$builder->set()**

此函数使您可以设置插入或更新的值。

**可以使用它代替将数据数组直接传递给插入或更新函数:**

::

	$builder->set('name', $name);
	$builder->insert();  // 产生: INSERT INTO mytable (`name`) VALUES ('{$name}')

如果多次调用该函数，它们将根据您执行插入还是更新而正确地进行组装::

	$builder->set('name', $name);
	$builder->set('title', $title);
	$builder->set('status', $status);
	$builder->insert();

**set()** 也将接受可选的第三个参数（``$escape``），如果设置为FALSE，它将防止数据被转义。为了说明两者之间的区别，在此 ``set()`` 使用带或不带转义参数的情况。

::

	$builder->set('field', 'field+1', FALSE);
	$builder->where('id', 2);
	$builder->update(); // 给出 UPDATE mytable SET field = field+1 WHERE `id` = 2

	$builder->set('field', 'field+1');
	$builder->where('id', 2);
	$builder->update(); // 给出 UPDATE `mytable` SET `field` = 'field+1' WHERE `id` = 2

您还可以将关联数组传递给此函数::

	$array = [
		'name'   => $name,
		'title'  => $title,
		'status' => $status
	];

	$builder->set($array);
	$builder->insert();

或对象::

	/*
	class Myclass {
		public $title   = 'My Title';
		public $content = 'My Content';
		public $date    = 'My Date';
	}
	*/

	$object = new Myclass;
	$builder->set($object);
	$builder->insert();

**$builder->update()**

生成更新字符串并根据您提供的数据运行查询。您可以将 **数组** 或 **对象** 传递给函数。这是使用数组的示例::

	$data = [
		'title' => $title,
		'name'  => $name,
		'date'  => $date
	];

	$builder->where('id', $id);
	$builder->update($data);
	// 产生:
	//
	//	UPDATE mytable
	//	SET title = '{$title}', name = '{$name}', date = '{$date}'
	//	WHERE id = $id

或者您可以提供一个对象::

	/*
	class Myclass {
		public $title   = 'My Title';
		public $content = 'My Content';
		public $date    = 'My Date';
	}
	*/

	$object = new Myclass;
	$builder->where('id', $id);
	$builder->update($object);
	// 产生:
	//
	// UPDATE `mytable`
	// SET `title` = '{$title}', `name` = '{$name}', `date` = '{$date}'
	// WHERE id = `$id`

.. note:: 所有值都会自动转义，从而产生更安全的查询。

您会注意到$builder->where()函数的使用，使您能够设置WHERE子句。您可以选择将此信息作为字符串直接传递给更新函数::

	$builder->update($data, "id = 4");

或作为数组::

	$builder->update($data, ['id' => $id]);

执行更新时，也可以使用上述$builder->set()函数。

**$builder->updateBatch()**

根据您提供的数据生成更新字符串，然后运行查询。您可以将 **数组** 或 **对象** 传递给函数。这是使用数组的示例::

	$data = [
	   [
	      'title' => 'My title' ,
	      'name'  => 'My Name 2' ,
	      'date'  => 'My date 2'
	   ],
	   [
	      'title' => 'Another title' ,
	      'name'  => 'Another Name 2' ,
	      'date'  => 'Another date 2'
	   ]
	];

	$builder->updateBatch($data, 'title');

	// 产生:
	// UPDATE `mytable` SET `name` = CASE
	// WHEN `title` = 'My title' THEN 'My Name 2'
	// WHEN `title` = 'Another title' THEN 'Another Name 2'
	// ELSE `name` END,
	// `date` = CASE
	// WHEN `title` = 'My title' THEN 'My date 2'
	// WHEN `title` = 'Another title' THEN 'Another date 2'
	// ELSE `date` END
	// WHERE `title` IN ('My title','Another title')

第一个参数是值的关联数组，第二个参数是where键。

.. note:: 所有值都会自动转义，从而产生更安全的查询。

.. note:: ``affectedRows()`` 由于这种方法的本质，因此无法为您提供正确的结果。而是 ``updateBatch()`` 返回受影响的行数。

**$builder->getCompiledUpdate()**

``$builder->getCompiledInsert()`` 除了产生UPDATE SQL字符串而不是INSERT SQL字符串外，此方法的工作原理与之完全相同。

有关更多信息，请查看`$builder->getCompiledInsert()`的文档。

.. note:: 此方法不适用于批量更新。

*************
删除数据
*************

**$builder->delete()**

生成删除SQL字符串并运行查询。

::

	$builder->delete(['id' => $id]);  // 产生: // DELETE FROM mytable  // WHERE id = $id

第一个参数是where子句。您也可以使用where()或or_where()函数，而不是将数据传递给该函数的第一个参数::

	$builder->where('id', $id);
	$builder->delete();

	// 产生:
	// DELETE FROM mytable
	// WHERE id = $id

如果要删除表中的所有数据，可以使用truncate()函数或emptyTable()。

**$builder->emptyTable()**

生成一个删除SQL字符串并运行查询::

	  $builder->emptyTable('mytable'); // 产生: DELETE FROM mytable

**$builder->truncate()**

生成截断SQL字符串并运行查询。

::

	$builder->truncate();

	// 产生:
	// TRUNCATE mytable

.. note:: 如果TRUNCATE命令不可用，则truncate()将作为“DELETE FROM table”执行。

**$builder->getCompiledDelete()**

``$builder->getCompiledInsert()`` 除了产生一个DELETE SQL字符串而不是一个INSERT SQL字符串外，它的工作方式与之完全相同。

有关更多信息，请参见$builder->getCompiledInsert()的文档。

***************
方法链
***************

方法链使您可以通过连接多个函数来简化语法。考虑以下示例::

	$query = $builder->select('title')
			 ->where('id', $id)
			 ->limit(10, 20)
			 ->get();

.. _ar-caching:

***********************
重置 Query Builder
***********************

**$builder->resetQuery()**

重置Query Builder使您可以重新开始查询，而无需先使用$builder->get() 或 $builder->insert()之类的方法执行查询。

在使用Query Builder生成SQL（例如 $builder->getCompiledSelect()）但随后选择运行查询的情况下，这很有用::

    // 注意get_compiled_select方法的第二个参数是FALSE
    $sql = $builder->select(['field1','field2'])
                   ->where('field3',5)
                   ->getCompiledSelect(false);

    // ...
    // 用 SQL 代码做一些疯狂的事情... 比如将它添加到 cron 脚本中
    // 以后执行还是什么...
    // ...

    $data = $builder->get()->getResultArray();

    // 会执行并返回以下查询的结果数组:
    // SELECT field1, field1 from mytable where field3 = 5;

***************
类参考
***************

.. php:class:: CodeIgniter\\Database\\BaseBuilder

	.. php:method:: resetQuery()

		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		重置当前的Query Builder状态。当您要构建可在某些条件下取消的查询时很有用。

	.. php:method:: countAllResults([$reset = TRUE])

		:param	bool	$reset: 是否重置SELECT的值
		:returns:	查询结果中的行数
		:rtype:	int

		生成特定于平台的查询字符串，该字符串对Query Builder查询返回的所有记录进行计数。

	.. php:method:: countAll([$reset = TRUE])

		:param	bool	$reset: 是否重置SELECT的值
		:returns:	查询结果中的行数
		:rtype:	int

		生成特定于平台的查询字符串，该字符串对Query Builder查询返回的所有记录进行计数。

	.. php:method:: get([$limit = NULL[, $offset = NULL[, $reset = TRUE]]]])

		:param	int	$limit: LIMIT子句
		:param	int	$offset: OFFSET子句
		:param 	bool $reset: 我们是否要清除query builder的值？
		:returns:	\CodeIgniter\Database\ResultInterface 实例 (方法链)
		:rtype:	\CodeIgniter\Database\ResultInterface

		根据已经调用的Query Builder方法编译并运行SELECT语句。

	.. php:method:: getWhere([$where = NULL[, $limit = NULL[, $offset = NULL[, $reset = TRUE]]]]])

		:param	string	$where: WHERE子句
		:param	int	$limit: LIMIT子句
		:param	int	$offset: OFFSET子句
		:param 	bool $reset: 我们是否要清除query builder的值？
		:returns:	\CodeIgniter\Database\ResultInterface 实例 (方法链)
		:rtype:	\CodeIgniter\Database\ResultInterface

		与 ``get()`` 相同，但也允许直接添加WHERE。

	.. php:method:: select([$select = '*'[, $escape = NULL]])

		:param	string	$select: 查询的SELECT部分
		:param	bool	$escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		将SELECT子句添加到查询中。

	.. php:method:: selectAvg([$select = ''[, $alias = '']])

		:param	string	$select: 计算平均值的字段
		:param	string	$alias: 结果值名称的别名
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		将SELECT AVG（field）子句添加到查询中。

	.. php:method:: selectMax([$select = ''[, $alias = '']])

		:param	string	$select: 计算最大值的字段
		:param	string	$alias: 结果值名称的别名
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		将SELECT MAX（field）子句添加到查询中。

	.. php:method:: selectMin([$select = ''[, $alias = '']])

		:param	string	$select: 计算最小值的字段
		:param	string	$alias: 结果值名称的别名
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		将SELECT MIN（field）子句添加到查询中。

	.. php:method:: selectSum([$select = ''[, $alias = '']])

		:param	string	$select: 计算总和的字段
		:param	string	$alias: 结果值名称的别名
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		将SELECT SUM（field）子句添加到查询中。

	.. php:method:: selectCount([$select = ''[, $alias = '']])

		:param	string	$select: 计算平均值的字段
		:param	string	$alias: 结果值名称的别名
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		将SELECT COUNT（field）子句添加到查询中。

	.. php:method:: distinct([$val = TRUE])

		:param	bool	$val: “distinct”标志的期望值
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		设置一个标志，该标志告诉查询生成器将DISTINCT子句添加到查询的SELECT部分​​。

	.. php:method:: from($from[, $overwrite = FALSE])

                :param	mixed	$from: 表名；字符串或数组
                :param	bool	$overwrite: 是否应该删除现有的第一个表？
                :returns:	BaseBuilder 实例 (方法链)
                :rtype:	BaseBuilder

		指定查询的FROM子句。

	.. php:method:: join($table, $cond[, $type = ''[, $escape = NULL]])

		:param	string	$table: join的表名
		:param	string	$cond: JOIN ON条件
		:param	string	$type: JOIN类型
		:param	bool	$escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		将JOIN子句添加到查询。

	.. php:method:: where($key[, $value = NULL[, $escape = NULL]])

		:param	mixed	$key: 要比较的字段名称或关联数组
		:param	mixed	$value: 如果是单个键，则与此值进行比较
		:param	bool	$escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例
		:rtype:	object

		生成查询的WHERE部分。使用“AND”分​​隔多个调用。

	.. php:method:: orWhere($key[, $value = NULL[, $escape = NULL]])

		:param	mixed	$key: 要比较的字段名称或关联数组
		:param	mixed	$value: 如果是单个键，则与此值进行比较
		:param	bool	$escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例
		:rtype:	object

		生成查询的WHERE部分。使用“OR”分​​隔多个调用。

	.. php:method:: orWhereIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要搜索的字段
		:param	array|Closure   $values: 目标值的数组，或用于子查询的匿名函数
		:param	bool	        $escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例
		:rtype:	object
        
		生成 ``WHERE field IN ('item', 'item')`` SQL查询，多个子句之间使用 OR 连接（如果适用）

	.. php:method:: orWhereNotIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要搜索的字段
		:param	array|Closure   $values: 目标值的数组，或用于子查询的匿名函数
		:param	bool	        $escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例
		:rtype:	object

		生成 ``WHERE field NOT IN ('item', 'item')`` SQL查询，多个子句之间使用 OR 连接（如果适用）

	.. php:method:: whereIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要检查的字段名称
		:param	array|Closure   $values: 目标值的数组，或用于子查询的匿名函数
		:param	bool            $escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例
		:rtype:	object

		生成 ``WHERE field IN ('item', 'item')`` SQL查询，多个子句之间使用 AND 连接（如果适用）

	.. php:method:: whereNotIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要检查的字段名称
		:param	array|Closure   $values: 目标值的数组，或用于子查询的匿名函数
		:param	bool	        $escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例
		:rtype:	object

		生成 ``WHERE field NOT IN ('item', 'item')`` SQL查询，多个子句之间使用 AND 连接（如果适用）

	.. php:method:: groupStart()

		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		使用组中的条件使用ANDs来启动组表达式。

	.. php:method:: orGroupStart()

		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		使用组中的条件使用ORs来启动组表达式。

	.. php:method:: notGroupStart()

		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		使用组中的条件使用AND NOTs来启动组表达式。

	.. php:method:: orNotGroupStart()

		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		使用组中的条件使用OR NOTs来启动组表达式。

	.. php:method:: groupEnd()

		:returns:	BaseBuilder 实例
		:rtype:	object

		结束组表达式。

	.. php:method:: like($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 字段名
		:param	string	$match: 要匹配的文本部分
		:param	string	$side: 表达式的哪一边放'％'通配符
		:param	bool	$escape: 是否转义值和标识符
		:param	bool    $insensitiveSearch: 是否强制不区分大小写的搜索
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		在查询中添加一个LIKE子句，使用AND分隔多个调用。

	.. php:method:: orLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 字段名
		:param	string	$match: 要匹配的文本部分
		:param	string	$side: 表达式的哪一边放'％'通配符
		:param	bool	$escape: 是否转义值和标识符
		:param	bool    $insensitiveSearch: 是否强制不区分大小写的搜索
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		在查询中添加一个LIKE子句，使用AND分隔多个调用。

	.. php:method:: notLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 字段名
		:param	string	$match: 要匹配的文本部分
		:param	string	$side: 表达式的哪一边放'％'通配符
		:param	bool	$escape: 是否转义值和标识符
		:param	bool    $insensitiveSearch: 是否强制不区分大小写的搜索
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		在查询中添加一个NOT LIKE子句，使用AND分隔多个调用。

	.. php:method:: orNotLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 字段名
		:param	string	$match: 要匹配的文本部分
		:param	string	$side: 表达式的哪一边放'％'通配符
		:param	bool	$escape: 是否转义值和标识符
		:param	bool    $insensitiveSearch: 是否强制不区分大小写的搜索
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		在查询中添加一个NOT LIKE子句，使用OR分隔多个调用。

	.. php:method:: having($key[, $value = NULL[, $escape = NULL]])

		:param	mixed	$key: 标识符（字符串）或字段/值对的关联数组
		:param	string	$value: 如果$key是标识符，则寻求值
		:param	string	$escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		向查询中添加HAVING子句，并使用AND分隔多个调用。

	.. php:method:: orHaving($key[, $value = NULL[, $escape = NULL]])

		:param	mixed	$key: 标识符（字符串）或字段/值对的关联数组
		:param	string	$value: 如果$key是标识符，则寻求值
		:param	string	$escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		向查询中添加一个HAVING子句，使用OR分隔多个调用。

	.. php:method:: orHavingIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要搜索的字段
		:param	array|Closure   $values: 目标值的数组，或用于子查询的匿名函数
		:param	bool	        $escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例
		:rtype:	object

		生成 ``HAVING field IN('item', 'item')`` SQL查询，多个子句之间使用 OR 连接（如果适用）

	.. php:method:: orHavingNotIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要搜索的字段
		:param	array|Closure   $values: 目标值的数组，或用于子查询的匿名函数
		:param	bool	        $escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例
		:rtype:	object

		生成 ``HAVING field NOT IN('item', 'item')`` SQL查询，多个子句之间使用 OR 连接（如果适用）

	.. php:method:: havingIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要检查的字段名称
		:param	array|Closure   $values: 目标值的数组，或用于子查询的匿名函数
		:param	bool            $escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例
		:rtype:	object

		生成 ``HAVING field IN('item', 'item')`` SQL查询，多个子句之间使用 AND 连接（如果适用）

	.. php:method:: havingNotIn([$key = NULL[, $values = NULL[, $escape = NULL]]])

		:param	string	        $key: 要检查的字段名称
		:param	array|Closure   $values: 目标值的数组，或用于子查询的匿名函数
		:param	bool	        $escape: 是否转义值和标识符
		:param	bool            $insensitiveSearch: 是否强制不区分大小写的搜索
		:returns:	BaseBuilder 实例
		:rtype:	object

		生成 ``HAVING field NOT IN('item', 'item')`` SQL查询，多个子句之间使用 AND 连接（如果适用）

	.. php:method:: havingLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 字段名
		:param	string	$match: 要匹配的文本部分
		:param	string	$side: 表达式的哪一边放'％'通配符
		:param	bool	$escape: 是否转义值和标识符
		:param	bool    $insensitiveSearch: 是否强制不区分大小写的搜索
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		将LIKE子句添加到查询的HAVING部分，以AND分隔多个调用。

	.. php:method:: orHavingLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 字段名
		:param	string	$match: 要匹配的文本部分
		:param	string	$side: 表达式的哪一边放'％'通配符
		:param	bool	$escape: 是否转义值和标识符
		:param	bool    $insensitiveSearch: 是否强制不区分大小写的搜索
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		将LIKE子句添加到查询的HAVING部分，以OR分隔多个调用。

	.. php:method:: notHavingLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 字段名
		:param	string	$match: 要匹配的文本部分
		:param	string	$side: 表达式的哪一边放'％'通配符
		:param	bool	$escape: 是否转义值和标识符
		:param	bool    $insensitiveSearch: 是否强制不区分大小写的搜索
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		将NOT LIKE子句添加到查询的HAVING部分，以AND分隔多个调用。

	.. php:method:: orNotHavingLike($field[, $match = ''[, $side = 'both'[, $escape = NULL[, $insensitiveSearch = FALSE]]]])

		:param	string	$field: 字段名
		:param	string	$match: 要匹配的文本部分
		:param	string	$side: 表达式的哪一边放'％'通配符
		:param	bool	$escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		将NOT LIKE子句添加到查询的HAVING部分，以OR分隔多个调用。

	.. php:method:: havingGroupStart()

		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		使用组中的内部条件ANDs来启动HAVING子句组表达式。

	.. php:method:: orHavingGroupStart()

		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		使用组中的内部条件ORs来启动HAVING子句组表达式。

	.. php:method:: notHavingGroupStart()

		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		使用组中的内部条件AND NOTs来启动HAVING子句组表达式。

	.. php:method:: orNotHavingGroupStart()

		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		使用组中的内部条件OR NOTs来启动HAVING子句组表达式。

	.. php:method:: havingGroupEnd()

		:returns:	BaseBuilder 实例
		:rtype:	object

		结束HAVING子句的组表达式。

	.. php:method:: groupBy($by[, $escape = NULL])

		:param	mixed	$by: 要分组的字段；字符串或数组
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		向查询中添加GROUP BY子句。

	.. php:method:: orderBy($orderby[, $direction = ''[, $escape = NULL]])

		:param	string	$orderby: 要排序的字段
		:param	string	$direction: 请求的顺序 - ASC, DESC 或 random
		:param	bool	$escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		向查询添加一个ORDER BY子句。

	.. php:method:: limit($value[, $offset = 0])

		:param	int	$value: 将结果限制为的行数
		:param	int	$offset: 要跳过的行数
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		将LIMIT和OFFSET子句添加到查询中。

	.. php:method:: offset($offset)

		:param	int	$offset: 要跳过的行数
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		将OFFSET子句添加到查询中。

	.. php:method:: set($key[, $value = ''[, $escape = NULL]])

		:param	mixed	$key: 字段名, 或字段/值对的数组
		:param	string	$value: 字段值，如果$key是单个字段
		:param	bool	$escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		添加字段/值对将在后面传递给 ``insert()``, ``update()`` 或 ``replace()``。

	.. php:method:: insert([$set = NULL[, $escape = NULL]])

		:param	array	$set: 字段/值对的关联数组
		:param	bool	$escape: 是否转义值和标识符
		:returns:	成功则为TRUE，失败则为FALSE
		:rtype:	bool

		编译并执行INSERT语句。

	.. php:method:: insertBatch([$set = NULL[, $escape = NULL[, $batch_size = 100]]])

		:param	array	$set: 要插入的数据
		:param	bool	$escape: 是否转义值和标识符
		:param	int	$batch_size: 一次插入的行数
		:returns:	插入的行数或失败时为FALSE
		:rtype:	mixed

		编译并执行批处理 ``INSERT`` 语句。

		.. note:: 如果 ``$batch_size`` 提供的行数多，则将执行多个 ``INSERT`` 查询，每个查询尝试插入最多 ``$batch_size`` 行。

	.. php:method:: setInsertBatch($key[, $value = ''[, $escape = NULL]])

		:param	mixed	$key: 字段名或字段/值对数组
		:param	string	$value: 字段值，如果$key是单个字段
		:param	bool	$escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		添加字段/值对，稍后通过 ``insertBatch()`` 插入到表中。

	.. php:method:: update([$set = NULL[, $where = NULL[, $limit = NULL]]])

		:param	array	$set: 字段/值对的关联数组
		:param	string	$where: WHERE子句
		:param	int	$limit: LIMIT子句
		:returns:	成功则为TRUE，失败则为FALSE
		:rtype:	bool

		编译并执行UPDATE语句。

	.. php:method:: updateBatch([$set = NULL[, $value = NULL[, $batch_size = 100]]])

		:param	array	$set: 字段名, 或字段/值对的关联数组
		:param	string	$value: 字段值，如果$set是单个字段
		:param	int	$batch_size: 在单个查询中分组的条件数
		:returns:	更新的行数或失败时为FALSE
		:rtype:	mixed

		编译并执行批处理 ``UPDATE`` 语句。

		.. note:: 如果 ``$batch_size`` 提供的字段/值对不止一个，则会执行多个查询，每个查询最多处理 ``$batch_size`` 字段/值对。

	.. php:method:: setUpdateBatch($key[, $value = ''[, $escape = NULL]])

		:param	mixed	$key: 字段名或字段/值对数组
		:param	string	$value: 字段值，如果$key是单个字段
		:param	bool	$escape: 是否转义值和标识符
		:returns:	BaseBuilder 实例 (方法链)
		:rtype:	BaseBuilder

		添加字段/值对，以便稍后通过 ``updateBatch()`` 在表中进行更新。

	.. php:method:: replace([$set = NULL])

		:param	array	$set: 字段/值对的关联数组
		:returns:	成功则为TRUE，失败则为FALSE
		:rtype:	bool

		编译并执行REPLACE语句。

	.. php:method:: delete([$where = ''[, $limit = NULL[, $reset_data = TRUE]]])

		:param	string	$where: WHERE子句
		:param	int	$limit: LIMIT子句
		:param	bool	$reset_data: TRUE以重置查询“write”子句
		:returns:	BaseBuilder 实例 (方法链) 或失败为FALSE
		:rtype:	mixed

		编译并执行DELETE查询。

    .. php:method:: increment($column[, $value = 1])

        :param string $column: 要递增的列的名称
        :param int    $value:  将列增加的数量

        将字段的值增加指定的数量。如果该字段不是数字字段（如VARCHAR），则很可能将其替换为$value。

    .. php:method:: decrement($column[, $value = 1])

        :param string $column: 要减少的列的名称
        :param int    $value:  减少列的数量

        将字段的值减指定的数量。如果该字段不是数字字段（如VARCHAR），则很可能将其替换为$value。

	.. php:method:: truncate()

		:returns:	成功则为TRUE，失败则为FALSE
		:rtype:	bool

		在表上执行TRUNCATE语句。

		.. note:: 如果使用的数据库平台不支持TRUNCATE，则将使用DELETE语句。

	.. php:method:: emptyTable()

		:returns:	成功则为TRUE，失败则为FALSE
		:rtype:	bool

		通过DELETE语句从表中删除所有记录。

	.. php:method:: getCompiledSelect([$reset = TRUE])

		:param	bool	$reset: 是否重置当前的QB值
		:returns:	编译后的SQL语句为字符串
		:rtype:	string

		编译SELECT语句，并将其作为字符串返回。

	.. php:method:: getCompiledInsert([$reset = TRUE])

		:param	bool	$reset: 是否重置当前的QB值
		:returns:	编译后的SQL语句为字符串
		:rtype:	string

		编译INSERT语句，并将其作为字符串返回。

	.. php:method:: getCompiledUpdate([$reset = TRUE])

		:param	bool	$reset: 是否重置当前的QB值
		:returns:	编译后的SQL语句为字符串
		:rtype:	string

		编译UPDATE语句，并将其作为字符串返回。

	.. php:method:: getCompiledDelete([$reset = TRUE])

		:param	bool	$reset: 是否重置当前的QB值
		:returns:	编译后的SQL语句为字符串
		:rtype:	string

		编译DELETE语句，并将其作为字符串返回。
