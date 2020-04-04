**************************
调试应用程序
**************************

.. contents::
    :local:
    :depth: 2

================
更换的 var_dump
================

尽管使用XDebug和良好的IDE调试应用程序必不可少，但有时只需要 ``var_dump()`` 即可。CodeIgniter通过捆绑用于PHP的出色的 `Kint <https://kint-php.github.io/kint/>`_ 调试工具，使这一点变得更好。这远远超出了您通常使用的工具，它提供了许多备用数据，例如将时间戳格式化为可识别的日期，将十六进制代码显示为颜色，将数组数据显示为表格以便于阅读等等等。

启用 Kint
=============

默认情况下，仅在 **development** and **testing** 环境中启用Kint 。可以通过修改主文件 **index.php** 的“环境配置”部分中 ``$useKint`` 的值来更改此值::

    $useKint = true;

使用 Kint
==========

**d()**

``d()`` 方法转储它知道的有关作为唯一参数传递到屏幕的内容的所有数据，并允许脚本继续执行::

    d($_SERVER);

**dd()**

此方法与 ``d()`` 相同，不同之处在于，该方法也 ``dies()`` 不会再对此请求执行任何代码。

**trace()**

通过Kint自己的独特旋转，这可以追溯到当前执行点::

    trace();

有关更多信息，请参见 `Kint's 页面 <https://kint-php.github.io/kint//>`_ 。

=================
调试工具栏
=================

调试工具栏提供了有关当前页面请求的简要信息，包括基准测试结果，已运行的查询，请求和响应数据等。在开发过程中，这对帮助您进行调试和优化非常有用。

.. note:: 调试工具栏仍在构建中，尚未实现一些计划的功能。

启用工具栏
====================

默认情况下，工具栏在生产以外的任何环境中都是启用的。只要定义了常量CI_DEBUG并且其值为正，就会显示该信息。这是在启动文件（即 ``app/Config/Boot/development.php``）中定义的，可以在启动文件中进行修改以确定它在什么环境中显示。

工具栏本身显示为 :doc:`After Filter </incoming/filters>` 。您可以通过将其从 **app/Config/Filters.php** 中的 ``$globals`` 属性中删除来使其停止运行。

选择要显示的内容
---------------------

顾名思义，CodeIgniter附带了几个收集器，这些收集器收集数据以显示在工具栏上。您可以轻松地自定义工具栏。要确定显示哪些收集器，请再次转到 **app/Config/Toolbar.php** 配置文件::

	public $collectors = [
		\CodeIgniter\Debug\Toolbar\Collectors\Timers::class,
		\CodeIgniter\Debug\Toolbar\Collectors\Database::class,
		\CodeIgniter\Debug\Toolbar\Collectors\Logs::class,
		\CodeIgniter\Debug\Toolbar\Collectors\Views::class,
 		\CodeIgniter\Debug\Toolbar\Collectors\Cache::class,
		\CodeIgniter\Debug\Toolbar\Collectors\Files::class,
		\CodeIgniter\Debug\Toolbar\Collectors\Routes::class,
		\CodeIgniter\Debug\Toolbar\Collectors\Events::class,
	];

注释掉您不想显示的所有收集器。通过提供完全限定的类名称，在此处添加自定义收集器。此处显示的确切收集器将影响显示哪些选项卡以及时间轴上显示哪些信息。

.. note:: 某些选项卡（例如“数据库”和“日志”）仅在它们具有要显示的内容时显示。否则，将它们删除以在较小的显示器上提供帮助。

CodeIgniter附带的收集器是:

* **Timers** 通过系统和您的应用程序收集所有基准数据。
* **Database** 显示所有数据库连接已执行的查询及其执行时间的列表。
* **Logs** 任何已记录的信息将在此处显示。在长时间运行的系统或记录了许多项目的系统中，这可能会导致内存问题，应禁用它。
* **Views** 在时间轴上显示视图的渲染时间，并在单独的选项卡上显示传递给视图的所有数据。
* **Cache** 将显示有关缓存命中和未命中以及执行时间的信息。
* **Files** 显示此请求期间已加载的所有文件的列表。
* **Routes** 显示有关当前路由和系统中定义的所有路由的信息。
* **Events** 显示在此请求期间已加载的所有事件的列表。

设定基准点
========================

为了让Profiler编译并显示基准数据，您必须使用特定的语法来命名标记点。

请阅读 :doc:`Benchmark Library </testing/benchmark>` 页面中有关设置基准点的信息。

创建自定义收集器
==========================

创建自定义收集器是一项简单的任务。您创建一个新的类，该类具有全命名空间，以便自动加载器可以找到它，并扩展 ``CodeIgniter\Debug\Toolbar\Collectors\BaseCollector`` 。这提供了许多可以覆盖的方法，并具有四个必需的类属性，必须根据收集器的工作方式正确设置这些属性
::

	<?php namespace MyNamespace;

	use CodeIgniter\Debug\Toolbar\Collectors\BaseCollector;

	class MyCollector extends BaseCollector
	{
		protected $hasTimeline   = false;

		protected $hasTabContent = false;

		protected $hasVarData    = false;

		protected $title         = '';
	}

对于要在工具栏的时间轴中显示信息的任何收集器，应将 **$hasTimeline** 设置为 ``true``。如果是这样，您将需要实现 ``formatTimelineData()`` 格式化和返回数据以供显示的方法。

收集器要显示其带有自定义内容的选项卡时 **$hasTabContent** 应该是 ``true``。如果是这样，您将需要提供一个 ``$title`` 实现 ``display()`` 以呈现选项卡内容的方法，并且如果您想在选项卡内容标题的右侧显示其他信息，则可能需要实现该方法  ``getTitleDetails()``。

如果此收集器要将其他数据添加到 ``Vars`` 选项卡 **$hasVarData** 应该是 ``true``。如果是这样，则需要实现 ``getVarData()`` 方法。

**$title** 显示在打开的选项卡上。

显示工具栏选项卡
------------------------

要显示工具栏选项卡，您必须:

1. 填写 ``$title`` 显示为工具栏标题和选项卡标题的文本。
2. 设置 ``$hasTabContent`` 为 ``true``。
3. 实现 ``display()`` 方法。
4. （可选）实现 ``getTitleDetails()`` 方法。

``display()`` 创建一个标签本身中显示的HTML。无需担心选项卡的标题，因为它是由工具栏自动处理的。它应该返回一个HTML字符串。

``getTitleDetails()`` 方法应该返回一个字符串，该字符串显示在选项卡标题的右侧。它可用于提供其他概述信息。例如，“数据库”选项卡显示所有连接中的查询总数，而“文件”选项卡显示文件总数。

提供时间表数据
-----------------------

要提供在时间轴中显示的信息，您必须:

1. 设置 ``$hasTimeline`` 为 ``true``。
2. 实现 ``formatTimelineData()`` 方法。

``formatTimelineData()`` 方法必须返回以时间轴可以对其进行正确排序和显示正确信息的方式格式化数组的数组。内部数组必须包含以下信息::

	$data[] = [
		'name'      => '',     // 名称显示在时间轴的左侧
		'component' => '',     // 在时间轴中间列出的组件名称
		'start'     => 0.00,   // 开始时间, 像 microtime(true)
		'duration'  => 0.00    // 持续时间, 像 mircrotime(true) - microtime(true)
	];

提供 Vars
--------------

要将数据添加到 ``Vars`` 选项卡中，您必须:

1. 设置 ``$hasVarData`` 为 ``true``。
2. 实现 ``getVarData()`` 方法。

``getVarData()`` 方法应返回一个包含要显示的键/值对数组的数组。外部数组键的名称是 ``Vars`` 选项卡上部分的名称::

	$data = [
		'section 1' => [
		    'foo' => 'bar',
		    'bar' => 'baz'
		],
		'section 2' => [
		    'foo' => 'bar',
		    'bar' => 'baz'
		]
	 ];
