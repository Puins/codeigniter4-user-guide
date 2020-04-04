###################
数据库迁移
###################

迁移是一种有条理、有组织的方式更改数据库的便捷方式。你可以手动编辑SQL的片段，然后你要负责告诉其他开发人员他们也需要去运行这段SQL。你还必须跟踪下次部署时需要对生产机器运行哪些更改。

数据库表 **migration** 会跟踪已经运行的迁移，因此您只需更新应用程序文件并调用 ``$migration->latest()`` 以确定应运行哪些迁移。您还可以使用 ``$migration->setNamespace(null)->progess()`` 来包括来自所有命名空间的迁移。


.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

********************
迁移文件名
********************

每个迁移都按数字顺序向前或向后运行，具体取决于所采用的方法。每次迁移均使用迁移时的时间戳进行编号
创建, 如 **YYYYMMDDHHIISS** 格式 (例如 **20121031100537** )。 这有助于防止在团队环境中工作时出现编号冲突。

在迁移文件之前加上迁移号，然后加上下划线，以及迁移的描述性名称。年、月和日期可以分开用破折号，下划线，或者根本不用。例如:

* 20121031100537_add_blog.php
* 2012-10-31-100538_alter_blog_track_views.php
* 2012_10_31_100539_alter_blog_add_translations.php


******************
创建迁移
******************

这将是新博客站点的首次迁移。所有迁移都在 **app/Database/Migrations/** 目录中，并命名，如 **20121031100537_Add_blog.php** 。
::

	<?php namespace App\Database\Migrations;

	class AddBlog extends \CodeIgniter\Database\Migration {

		public function up()
		{
			$this->forge->addField([
				'blog_id'          => [
					'type'           => 'INT',
					'constraint'     => 5,
					'unsigned'       => TRUE,
					'auto_increment' => TRUE
				],
				'blog_title'       => [
					'type'           => 'VARCHAR',
					'constraint'     => '100',
				],
				'blog_description' => [
					'type'           => 'TEXT',
					'null'           => TRUE,
				],
			]);
			$this->forge->addKey('blog_id', TRUE);
			$this->forge->createTable('blog');
		}

		public function down()
		{
			$this->forge->dropTable('blog');
		}
	}

您可以通过 ``$this->db`` 和 ``$this->forge`` 分别使用数据库连接和数据库Forge类。

或者，你可以使用命令行调用来生成框架迁移文件。请参阅下面的更多细节。

外键
============

当表中包含外键时，在尝试删除表和列时，迁移通常会引起问题。可以使用 ``disableForeignKeyChecks()`` 和
``enableForeignKeyChecks()`` 方法在数据库连接时，以便运行迁移时暂时绕过外键检查。

::

    public function up()
    {
        $this->db->disableForeignKeyChecks();

        // Migration rules would go here...

        $this->db->enableForeignKeyChecks();
    }

数据库组
===============

只能针对单个数据库组运行迁移。如果在 **app/Config/Database.php** 中定义了多个组 ，则它将针对该 ``$defaultGroup`` 同一配置文件中指定的组运行。有时你可能需要为不同的数据库组使用不同的模式。也许你有一个用于所有常规站点信息的数据库，而另一个数据库用于关键任务数据。通过 ``$DBGroup`` 在迁移上设置属性，可以确保仅针对正确的组运行迁移。此名称必须与数据库组的名称完全匹配::

    <?php namespace App\Database\Migrations;

    class AddBlog extends \CodeIgniter\Database\Migration
    {
        protected $DBGroup = 'alternate_db_group';

        public function up() { . . . }

        public function down() { . . . }
    }

命名空间
==========

迁移库可以自动扫描你在 **app/Config/Autoload.php** 中定义的所有命名空间 及其 ``$psr4`` 属性以匹配目录名称。它将包括它在 **Database/Migrations** 中找到的所有迁移。

每个命名空间都有自己的版本序列，这将帮助您升级和降级每个模块（命名空间），而不会影响其他命名空间。

例如，假设我们在 ``Autoload`` 配置文件中定义了以下命名空间::

	$psr4 = [
		'App'       => APPPATH,
		'MyCompany' => ROOTPATH.'MyCompany'
	];

这将查找位于 **APPPATH/Database/Migrations** 和 **ROOTPATH/Database/Migrations** 的任何迁移。这使得在可重用的模块化代码套件中包含迁移变得简单。

*************
用法示例
*************

在此示例中，一些简单的代码放在 **app/Controllers/Migrate.php** 中以更新架构::

    <?php namespace App\Controllers;

	class Migrate extends \CodeIgniter\Controller
	{

		public function index()
		{
			$migrate = \Config\Services::migrations();

			try
			{
			  $migrate->latest();
			}
			catch (\Exception $e)
			{
			  // Do something with the error here...
			}
		}

	}

*******************
命令行工具
*******************

CodeIgniter附带了几个 :doc:`commands </cli/cli_commands>`，它们可以从命令行获得，以帮助你处理迁移。这些工具不需要使用迁移，但可能会使那些希望使用它们的人更容易。这些工具主要提供对 **MigrationRunner类** 中可用的相同方法的访问。

**migrate**

迁移具有所有可用迁移的数据库组::

    > php spark migrate

您可以使用（migrate）以下选项:

- (-g) 选择数据库组，否则将使用默认数据库组。
- (-n) 选择命名空间，否则将使用（App）命名空间。
- (-all) 将所有命名空间迁移到最新的迁移

此示例将 **Blog** 命名空间迁移到 **test** 数据库组上::

    > php spark migrate -g test -n Blog

当使用 “-all” 选项时，它将扫描所有命名空间，以尝试查找所有尚未运行的迁移。这些都将被收集，然后按创建日期按组进行排序。这应该有利于尽量减少主应用程序与任何模块之间的任何潜在冲突。

**rollback**

回滚所有迁移，将数据库组置于空白状态, 有效迁移0::

  > php spark migrate:rollback

你可以使用（rollback）以下选项:

- (-g) 选择数据库组，否则将使用默认数据库组。
- (-b) 选择批次：自然数指定批次，负数表示相对批次。
- (-f) 强制绕过确认问题，仅在生产环境中提出

**refresh**

首先回滚所有迁移，然后再迁移所有迁移，以刷新数据库状态::

  > php spark migrate:refresh

你可以使用（refresh）以下选项:

- (-g) 选择数据库组，否则将使用默认数据库组。
- (-n) 选择命名空间，否则将使用（App）命名空间。
- (-all) 选择刷新所有命名空间
- (-f) 强制绕过确认问题，仅在生产环境中提出

**status**

显示所有迁移及其运行的日期和时间的列表， 如果尚未运行，则显示'--'::

  > php spark migrate:status
  文件名                  迁移时间
  First_migration.php    2016-04-25 04:44:22

你可以使用（status）以下选项:

- (-g) 选择数据库组，否则将使用默认数据库组。

**create**

在 **app/Database/Migrations** 中创建框架迁移文件。它会自动添加当前时间戳。被创建的类名是以“Pascal命名法”命名的示例版本文件。

  > php spark migrate:create [filename]


你可以使用（create）以下选项:

- (-n) 选择命名空间，否则将使用（App）命名空间。

*********************
迁移参数
*********************

以下是 **app/Config/Migrations.php** 中提供的所有迁移配置选项的表。

========================== ====================== ========================== =============================================================
参数                        默认值                  可选项                     描述
========================== ====================== ========================== =============================================================
**enabled**                TRUE                   TRUE / FALSE               启用或者禁用迁移。
**table**                  migrations             None                       用于存储当前版本的数据库表名。
**timestampFormat**        Y-m-d-His\_                                       创建迁移时用于时间戳的格式。
========================== ====================== ========================== =============================================================

***************
类参考
***************

.. php:class:: CodeIgniter\\Database\\MigrationRunner

	.. php:method:: findMigrations()

		:returns:	迁移文件名数组
		:rtype:	array

		返回一个迁移文件名数组，该数组位于 **path** 属性中。

	.. php:method:: latest($group)

		:param	mixed	$group: 数据库组名称（如果为null），将使用默认数据库组。
		:returns:	TRUE 为成功, FALSE 为失败
		:rtype:	bool

        这将定位命名空间（或所有命名空间）的迁移，确定哪些迁移尚未运行，并按版本顺序运行它们（命名空间混合）。

	.. php:method:: regress($batch, $group)

		:param	mixed	$batch: 要向下迁移到的上一批；1+指定批次，0表示全部还原，负号表示相对批次（例如，-3表示“三批次退回”）。
		:param	mixed	$group: 数据库组名称（如果为null），将使用默认数据库组。
		:returns:	TRUE 为成功, FALSE 为失败 或者 找不到迁移
		:rtype:	bool

		回归可以用于将更改逐批回滚到以前的状态。
		::

			$migration->batch(5);
			$migration->batch(-1);

	.. php:method:: force($path, $namespace, $group)

		:param	mixed	$path:  有效迁移文件的路径。
		:param	mixed	$namespace: 提供的迁移的命名空间。
		:param	mixed	$group: 数据库组名称（如果为null），将使用默认数据库组。
		:returns:	TRUE 为成功, FALSE 为失败
		:rtype:	bool

		这将强制迁移单个文件，而不考虑顺序或批次。方法“up”或“down”是根据它是否已经被迁移来检测的。 **Note**: 仅建议将这种方法用于测试，并且可能导致数据一致性问题。

	.. php:method:: setNamespace($namespace)

		:param  string  $namespace: 应用命名空间。
		:returns:   当前的 MigrationRunner类的 实例
		:rtype:     CodeIgniter\Database\MigrationRunner

		设置库应查找迁移文件的路径::

			$migration->setNamespace($path)
					->latest();

	.. php:method:: setGroup($group)

		:param  string  $group: 数据库组名。
		:returns:   当前的 MigrationRunner类的 实例
		:rtype:     CodeIgniter\Database\MigrationRunner

		设置库应查找迁移文件的路径::

			$migration->setNamespace($path)
					->latest();
