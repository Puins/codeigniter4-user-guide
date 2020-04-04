###########
CLI 库
###########

CodeIgniter4 的CLI库使创建交互式命令行脚本变得简单， 包含:

* 提示用户获取更多信息
* 在终端上写多色文本
* 哔哔声（很友好！）
* 在长任务期间显示进度条
* 将长文本行换行以适合窗口。

.. contents::
    :local:
    :depth: 2

初始化类
======================

您不需要创建CLI库的实例，因为它的所有方法都是静态的。相反，你只是需要确保控制器可以通过类上方的 ``use`` 语句找到它::

	<?php namespace App\Controllers;

	use CodeIgniter\CLI\CLI;

	class MyController extends \CodeIgniter\Controller
	{
		. . .
	}

类在第一次加载文件时自动初始化。

从用户获取输入
===========================

有时您需要询问用户更多信息。 他们可能未提供可选的命令行参数，否则脚本可能已遇到现有文件，需要在覆盖前进行确认。 这是用 ``prompt()`` 方法处理。

您可以通过将问题作为第一个参数传递来提供问题::

	$color = CLI::prompt('What is your favorite color?');

您可以提供默认答案，如果用户只是点击回车键，则可以通过在第二个参数默认::

	$color = CLI::prompt('What is your favorite color?', 'blue');

您可以通过传入允许的答案数组作为第二个参数来限制可接受的答案::

	$overwrite = CLI::prompt('File exists. Overwrite?', ['y','n']);

最后，您可以将验证规则作为第三个参数传递给答案输入::

	$email = CLI::prompt('What is your email?', null, 'required|valid_email');

提供反馈
==================

**write()**

提供了几种方法来向用户提供反馈。 这可以像单个状态更新一样简单或包装到用户终端窗口的复杂信息表。 其核心是 ``write()`` 将字符串作为第一个参数输出的方法::

	CLI::write('The rain in Spain falls mainly on the plains.');

您可以通过输入颜色名称作为第一个参数来更改文本的颜色::

	CLI::write('File created.', 'green');

这可用于按状态区分消息，或使用其他颜色创建“标题”。 您甚至可以通过传入颜色名称作为第三个参数来设置背景颜色::

	CLI::write('File overwritten.', 'light_red', 'dark_gray');

可以使用以下前景色:

* black
* dark_gray
* blue
* dark_blue
* light_blue
* green
* light_green
* cyan
* light_cyan
* red
* light_red
* purple
* light_purple
* light_yellow
* yellow
* light_gray
* white

一个较小的数字可以作为背景色:

* black
* blue
* green
* cyan
* red
* yellow
* light_gray
* magenta

**print()**

打印功能与 ``write（）`` 方法相同，除了它不会在换行符之前或之后强制换行。而是将其打印到当前光标所在的屏幕上。 这使您可以打印来自不同调用的多个项目在同一行。 当您想要显示状态，执行某项操作，然后在同一行上打印“完成”::

    for ($i = 0; $i <= 10; $i++)
    {
        CLI::print($i);
    }

**color()**

虽然 ``write（）`` 命令将单行写入终端，并以EOL字符结尾，但您可以使用 ``color（）`` 方法生成一个可以以相同方式使用的字符串片段，只是它不会强制打印后的EOL。 这使您可以在同一行上创建多个输出。 或者，更常见的是，您可以使用它在 ``write()`` 方法内部以在内部创建不同颜色的字符串::

	CLI::write("fileA \t". CLI::color('/path/to/file', 'white'), 'yellow');

此示例将在窗口中写一行，其中 ``fileA`` 为黄色，后跟一个制表符，然后 ``/path/to/file`` 以白色文本显示。

**error()**

如果需要输出错误，应该使用适当命名的 ``error()`` 方法。这些输入的淡红色的文字对于STDERR，而不是STDOUT，就像 ``write()`` 和 ``color()`` 一样。如果您有脚本监视，这将非常有用。对于错误，他们不必筛选所有的信息，只需要筛选实际的错误消息。你用它正如您所希望的 ``write()`` 方法::

	CLI::error('Cannot write to file: '. $file);

**wrap()**

此命令将获取一个字符串，开始在当前行上打印它，并在新行上将其包装为设置的长度。当显示一个选项列表时，这可能很有用，其中包含了要在当前选项中换行的描述窗口不离开屏幕::

	CLI::color("task1\t", 'yellow');
	CLI::wrap("Some long description goes here that might be longer than the current window.");

默认情况下，字符串将在终端宽度处换行。Windows当前无法确定窗口大小，所以我们默认为80个字符。如果你想把宽度限制为庚短，您可以非常确定是否适合窗口，将最大行长度作为第二个参数传递。这个将在最近的单词屏障处断开字符串，以便单词不会断开。
::

	// 文本最大宽度为20个字符
	CLI::wrap($description, 20);

您可能会发现标题，文件或任务的左侧需要一列，而文本则需要一列在右侧及其说明。 默认情况下，这将切换到窗口的左边缘，即不允许事物按列排列。 在这种情况下，您可以传递多个空格来填充第一行之后的每一行，以便您在左侧具有清晰的列边缘::

	// 确定所有标题的最大长度
	// 确定左栏的宽度
	$maxlen = max(array_map('strlen', $titles));

	for ($i=0; $i <= count($titles); $i++)
	{
		CLI::write(
			// 在行的左边显示标题
			$title[$i].'   '.
			// 将描述包装在右边的列中
			// 左边3个字符宽于
			// 左边最长的项目。
			CLI::wrap($descriptions[$i], 40, $maxlen+3)
		);
	}

会得到如下这样:

.. code-block:: none

    task1a     Lorem Ipsum is simply dummy
               text of the printing and typesetting
               industry.
    task1abc   Lorem Ipsum has been the industry's
               standard dummy text ever since the

**newLine()**

 ``newLine()`` 方法向用户显示一个空行。它不需要任何参数::

	CLI::newLine();

**clearScreen()**

可以使用``clearScreen()`` 方法清除当前终端窗口。在大多数Windows版本中，这将只需插入40个空行，因为Windows不支持此功能。Windows 10 bash集成应该改变它::

	CLI::clearScreen();

**showProgress()**

如果您有一个长期运行的任务，想让用户了解进度，请使用 ``showProgress()`` 方法显示如下内容:

.. code-block:: none

	[####......] 40% Complete

这个方块的动画效果非常好。

要使用它，请传入当前步骤作为第一个参数，传入步骤总数作为第二个参数。完成百分比和显示长度将根据该数字确定。完成后，将 ``false`` 作为第一个参数传递，将删除进度条。
::

	$totalSteps = count($tasks);
	$currStep   = 1;

	foreach ($tasks as $task)
	{
		CLI::showProgress($currStep++, $totalSteps);
		$task->run();
	}

	// Done, so erase it...
	CLI::showProgress(false);

**table()**

::

	$thead = ['ID', 'Title', 'Updated At', 'Active'];
	$tbody = [
		[7, 'A great item title', '2017-11-15 10:35:02', 1],
		[8, 'Another great item title', '2017-11-16 13:46:54', 0]
	];

	CLI::table($tbody, $thead);

.. code-block:: none

	+----+--------------------------+---------------------+--------+
	| ID | Title                    | Updated At          | Active |
	+----+--------------------------+---------------------+--------+
	| 7  | A great item title       | 2017-11-16 10:35:02 | 1      |
	| 8  | Another great item title | 2017-11-16 13:46:54 | 0      |
	+----+--------------------------+---------------------+--------+

**wait()**

等待一定的秒数，可选地显示等待消息和等待按键。

::

        // 等待指定的时间间隔，并显示倒计时
        CLI::wait($seconds, true);

        // 显示继续消息并等待输入
        CLI::wait(0, false);

        // 等待指定的间隔
        CLI::wait($seconds, false);
