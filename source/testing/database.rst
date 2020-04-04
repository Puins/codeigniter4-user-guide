=====================
测试数据库
=====================

.. contents::
    :local:
    :depth: 2

测试类
==============

为了利用CodeIgniter提供的用于测试的内置数据库工具，您的测试必须继承 ``CIDatabaseTestCase`` ::

    <?php namespace App\Database;

    use CodeIgniter\Test\CIDatabaseTestCase;

    class MyTests extends CIDatabaseTestCase
    {
        . . .
    }

因为在 ``setUp()`` 和 ``tearDown()`` 阶段执行了特殊功能，所以如果需要使用父方法，则必须确保调用它们的方法，否则您将失去此处描述的许多功能::

    <?php namespace App\Database;

    use CodeIgniter\Test\CIDatabaseTestCase;

    class MyTests extends CIDatabaseTestCase
    {
        public function setUp()
        {
            parent::setUp();

            // Do something here....
        }

        public function tearDown()
        {
            parent::tearDown();

            // Do something here....
        }
    }

设置一个测试数据库
==========================

在运行数据库测试时，您需要提供一个可以在测试期间使用的数据库。该框架提供了特定于CodeIgniter的工具，而不是使用PHPUnit内置的数据库功能。第一步是确保您已在 **app/Config/Database.php** 中设置了一个数据库组 ``tests`` 。这指定了仅在运行测试时使用的数据库连接，以确保其他数据的安全。

如果团队中有多个开发人员，则可能需要将凭据存储在 **.env** 文件中。为此，请编辑文件以确保存在以下各行并具有正确的信息::

    database.tests.dbdriver = 'MySQLi';
    database.tests.username = 'root';
    database.tests.password = '';
    database.tests.database = '';

迁移和种子
--------------------

运行测试时，您需要确保数据库具有正确的架构设置，并且每个测试都处于已知状态。通过向测试中添加几个类属性，可以使用迁移和种子设置数据库。
::

    <?php namespace App\Database;

    use CodeIgniter\Test\CIDatabaseTestCase;

    class MyTests extends\CIDatabaseTestCase
    {
        protected $refresh  = true;
        protected $seed     = 'TestSeeder';
        protected $basePath = 'path/to/database/files';
    }

**$refresh**

此布尔值确定在每次测试之前是否完全刷新数据库。如果为true，则将所有迁移回滚到版本0，然后将数据库迁移到最新的可用迁移。

**$seed**

如果存在且不为空，则此参数指定种子文件的名称，该种子文件用于在每次运行测试之前用测试数据填充数据库。

**$basePath**

默认情况下，CodeIgniter将在 **tests/_support/Database/Seeds** 中查找以在测试期间应运行的种子。您可以通过指定 ``$basePath`` 属性来更改此目录。该目录不应包括 **seeds** 目录，而应包含指向子目录的单个目录的路径。

**$namespace**

默认情况下，CodeIgniter将在 **tests/_support/DatabaseTestMigrations/Database/Migrations** 中查找以定位在测试期间应运行的迁移。您可以通过在 ``$namespace`` 属性中指定新的命名空间来更改此位置。这不应包括数据库/迁移路径，而应仅包括基本命名空间。

辅助方法
==============

 **CIDatabaseTestCase** 类提供了一些辅助方法以帮助测试你的数据库。

**seed($name)**

允许您将“种子”手动加载到数据库中。唯一的参数是要运行的种子的名称。种子必须存在于中指定的 ``$basePath`` 路径中。

**dontSeeInDatabase($table, $criteria)**

确定数据库中不存在具有与DOES中 ``$criteria`` 的键/值对匹配的条件的行。
::

    $criteria = [
        'email'  => 'joe@example.com',
        'active' => 1
    ];
    $this->dontSeeInDatabase('users', $criteria);

**seeInDatabase($table, $criteria)**

确定数据库中存在具有与DOES中 ``$criteria`` 的键/值对匹配的条件的行。
::

    $criteria = [
        'email'  => 'joe@example.com',
        'active' => 1
    ];
    $this->seeInDatabase('users', $criteria);

**grabFromDatabase($table, $column, $criteria)**

从与 ``$criteria`` 行匹配的指定表中返回 ``$column`` 的值。如果找到多个行，它将仅针对第一行进行测试。
::

    $username = $this->grabFromDatabase('users', 'username', ['email' => 'joe@example.com']);

**hasInDatabase($table, $data)**

在数据库中插入新行。当前测试运行后，该行将被删除。``$data`` 是具有要插入表中的数据的关联数组。
::

    $data = [
        'email' => 'joe@example.com',
        'name'  => 'Joe Cool'
    ];
    $this->hasInDatabase('users', $data);

**seeNumRecords($expected, $table, $criteria)**

确定在数据库中找到了许多匹配 ``$criteria`` 的匹配行。
::

    $criteria = [
        'active' => 1
    ];
    $this->seeNumRecords(2, 'users', $criteria);

