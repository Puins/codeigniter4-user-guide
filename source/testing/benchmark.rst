############
基准测试类
############

CodeIgniter提供了两个单独的工具来帮助您对代码进行基准测试和测试不同的选项：Timer和Iterator。计时器使您可以轻松地计算脚本执行过程中两点之间的时间。通过Iterator，您可以设置多个版本并运行这些测试，记录性能和内存统计信息，以帮助您确定最佳版本。

Timer类始终处于活动状态，从调用框架开始一直到将输出发送给用户之前一直处于活动状态，从而为整个系统执行提供了非常精确的计时。

.. contents::
    :local:
    :depth: 2

===============
使用Timer
===============

使用计时器，您可以测量应用程序执行两分钟之间的时间。这使测量应用程序不同方面的性能变得简单。所有测量均使用 ``start()`` 和 ``stop()`` 方法完成。

``start()`` 方法采用单个参数：此计时器的名称。您可以使用任何字符串作为计时器的名称。它仅供您以后参考以了解哪个度量是哪个::

	$benchmark = \Config\Services::timer();
	$benchmark->start('render view');

该 ``stop()`` 方法将要停止的计时器名称作为唯一参数，并且::

	$benchmark->stop('render view');

该名称不区分大小写，但是必须与启动计时器时输入的名称匹配。

另外，您可以使用 :doc:`global function </general/common_functions>` 中的 ``timer()`` 来启动和停止计时器::

	// 开始时间
	timer('render view');
	// 如果已启动此名称之一，则停止正在运行的计时器
	timer('render view');

查看您的基准点
=============================

当您的应用程序运行时，您设置的所有计时器均由Timer类收集。但是，它不会自动显示它们。您可以通过调用 ``getTimers()`` 方法来检索所有计时器。这将返回一系列基准信息，包括开始，结束和持续时间::

	$timers = $benchmark->getTimers();

	// Timers =
	[
		'render view'  => [
			'start'    => 1234567890,
			'end'      => 1345678920,
			'duration' => 15.4315      // number of seconds
		]
	]

您可以通过传递要显示为唯一参数的小数位数来更改计算持续时间的精度。默认值为小数点后4个数字::

	$timers = $benchmark->getTimers(6);

计时器会自动显示在 :doc:`Debub Toolbar </testing/debugging>` 中。

显示执行时间
=========================

尽管 ``getTimers()`` 方法将为您提供项目中所有计时器的原始数据，但您可以使用 `getElapsedTime()` 方法以秒为单位检索单个计时器的持续时间。第一个参数是要显示的计时器的名称。第二个是要显示的小数位数。默认为4::

	echo timer()->getElapsedTime('render view');
	// 显示: 0.0234

======================
使用迭代（Iterator）
======================

Iterator是一个简单的工具，旨在让您尝试解决方案的多种变体，以查看速度差异和不同的内存使用模式。您可以添加任意数量的“任务”以使其运行，该类将运行该任务数百次或数千次，以获得更清晰的性能图。然后，您的脚本可以检索并使用结果，也可以将其显示为HTML表。

创建要运行的任务
=====================

任务在闭包中定义。任务创建的任何输出将被自动丢弃。它们通过 `add()` 方法添加到Iterator类。第一个参数是您要用来引用此测试的名称。第二个参数是Closure本身::

	$iterator = new \CodeIgniter\Benchmark\Iterator();

	// 添加新任务
	$iterator->add('single_concat', function()
		{
			$str = 'Some basic'.'little'.'string concatenation test.';
		}
	);

	// 添加其他任务
	$iterator->add('double', function($a='little')
		{
			$str = "Some basic {$little} string test.";
		}
	);

运行的任务
=================

一旦添加了要运行的任务，就可以使用该 ``run()`` 方法多次遍历任务。默认情况下，它将运行每个任务1000次。对于大多数简单的测试，这可能就足够了。如果您需要运行测试多次，则可以将数字作为第一个参数传递::

	// 运行测试 3000 次.
	$iterator->run(3000);

一旦运行，它将返回带有测试结果的HTML表。如果您不希望显示结果，则可以传入 `false` 作为第二个参数::

	// 不显示结果。
	$iterator->run(1000, false);
