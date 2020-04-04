############
事务
############

CodeIgniter 的数据库抽象类，允许你将事务和支持事务安全表类型的数据库一起使用。在 MySQL 中，你需要将表设置为 InnoDB 或者 BDB 类型， 而不是更常见的 MyISAM。大多数的数据库本身支持事务。

如果你不熟悉事务，我们建议你找到一个很好的在线资源，以了解你使用的数据库。以下的信息假定你对事务有基本的了解。

CodeIgniter 的事务方法
======================================

CodeIgniter使用一种与通用数据库类ADODB所使用的过程非常相似的事务处理方法。我们选择这种方法是因为它极大地简化了运行事务的过程。在大多数情况下，所需要做的只是两行代码。

传统的事务需要执行大量工作，因为它们要求您跟踪查询并根据查询的成功或失败来确定是提交还是回退。对于嵌套查询，这尤其麻烦。相反，我们实施了一个智能事务系统，该系统可以自动为您完成所有操作（如果您愿意，也可以手动管理交易，但实际上没有任何好处）。

运行事务
====================

要使用事务运行查询，您将使用 ``$this->db->transStart()`` 和 ``$this->db->transComplete()`` 函数，如下所示::

	$this->db->transStart();
	$this->db->query('AN SQL QUERY...');
	$this->db->query('ANOTHER QUERY...');
	$this->db->query('AND YET ANOTHER QUERY...');
	$this->db->transComplete();

您可以在启动/完成功能之间运行任意数量的查询，这些查询将根据任何给定查询的成功或失败而全部提交或回退。

严格模式
===========

默认情况下，CodeIgniter以严格模式运行所有事务。启用严格模式后，如果您正在运行多个事务组，则如果一个组失败，则所有组都将回滚。如果禁用严格模式，则每个组将被独立对待，这意味着一个组的故障不会影响其他任何组。

严格模式可以按以下方式禁用::

	$this->db->transStrict(false);

管理错误
===============

如果您在 ``Config/Database.php`` 文件中启用了错误报告，则如果提交失败，您将看到标准错误消息。如果关闭调试，则可以按以下方式管理自己的错误::

	$this->db->transStart();
	$this->db->query('AN SQL QUERY...');
	$this->db->query('ANOTHER QUERY...');
	$this->db->transComplete();

	if ($this->db->transStatus() === FALSE)
	{
		// 生成错误...或使用 log_message() 函数记录错误
	}

禁用事务处理
======================

默认情况下启用事务。如果您想禁用事务，可以使用 ``$this->db->transOff()`` 进行操作::

	$this->db->transOff();

	$this->db->transStart();
	$this->db->query('AN SQL QUERY...');
	$this->db->transComplete();

禁用事务后，您的查询将自动提交，就像在运行没有事务的查询时一样。

测试模式
=========

您可以选择将事务系统置于“测试模式”，这将使您的查询回滚-即使查询产生有效的结果。要使用测试模式，只需将 ``$this->db->transStart()`` 函数中的第一个参数设置为TRUE::

	$this->db->transStart(true); // 查询将回滚
	$this->db->query('AN SQL QUERY...');
	$this->db->transComplete();

手动运行事务
=============================

如果你想手动运行事务，可以按如下方式执行::

	$this->db->transBegin();

	$this->db->query('AN SQL QUERY...');
	$this->db->query('ANOTHER QUERY...');
	$this->db->query('AND YET ANOTHER QUERY...');

	if ($this->db->transStatus() === FALSE)
	{
		$this->db->transRollback();
	}
	else
	{
		$this->db->transCommit();
	}

.. note:: 运行手动事务时，请确保使用 ``$this->db->transBegin()``，而 **不** 使用 ``$this->db->transStart()``。
