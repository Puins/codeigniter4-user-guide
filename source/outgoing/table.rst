################
HTML 表格 类
################

Table类提供了一些方法，使您可以从数组或数据库结果集中自动生成HTML表格。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

*********************
使用 Table 类
*********************

初始化类
======================

Table类未作为服务提供，应“正常”实例化，例如::

	$table = new \CodeIgniter\View\Table();

示例
========

这是显示如何从多维数组创建表格的示例。请注意，第一个数组索引将成为表头（或者您可以使用下面的函数参考中描述的 ``setHeading()`` 方法来设置自己的表头）。

::

	$table = new \CodeIgniter\View\Table();

	$data = array(
		array('Name', 'Color', 'Size'),
		array('Fred', 'Blue', 'Small'),
		array('Mary', 'Red', 'Large'),
		array('John', 'Green', 'Medium')	
	);

	echo $table->generate($data);

这是根据数据库查询结果创建的表格示例。table类将根据表名称自动生成表头（或者您可以使用下面的函数参考中描述的 ``setHeading()`` 方法来设置自己的表头）。

::

	$table = new \CodeIgniter\View\Table();

	$query = $db->query('SELECT * FROM my_table');

	echo $table->generate($query);

这是显示如何使用离散参数创建表格的示例::

	$table = new \CodeIgniter\View\Table();

	$table->setHeading('Name', 'Color', 'Size');

	$table->addRow('Fred', 'Blue', 'Small');
	$table->addRow('Mary', 'Red', 'Large');
	$table->addRow('John', 'Green', 'Medium');

	echo $table->generate();

这是相同的示例，除了使用单个数组代替单个参数外::

	$table = new \CodeIgniter\View\Table();

	$table->setHeading(array('Name', 'Color', 'Size'));

	$table->addRow(['Fred', 'Blue', 'Small']);
	$table->addRow(['Mary', 'Red', 'Large']);
	$table->addRow(['John', 'Green', 'Medium']);

	echo $table->generate();

改变表格外观
===============================

Table 类允许您设置表格模板，通过该模板可以指定布局的设计。这是模板原型::

	$template = [
		'table_open'		=> '<table border="0" cellpadding="4" cellspacing="0">',

		'thead_open'		=> '<thead>',
		'thead_close'		=> '</thead>',

		'heading_row_start'	=> '<tr>',
		'heading_row_end'	=> '</tr>',
		'heading_cell_start'	=> '<th>',
		'heading_cell_end'	=> '</th>',

		'tfoot_open'		 => '<tfoot>',
		'tfoot_close'		 => '</tfoot>',

		'footing_row_start'	 => '<tr>',
		'footing_row_end'	 => '</tr>',
		'footing_cell_start'     => '<td>',
		'footing_cell_end'	 => '</td>',

		'tbody_open'		=> '<tbody>',
		'tbody_close'		=> '</tbody>',

		'row_start'		=> '<tr>',
		'row_end'		=> '</tr>',
		'cell_start'		=> '<td>',
		'cell_end'		=> '</td>',

		'row_alt_start'		=> '<tr>',
		'row_alt_end'		=> '</tr>',
		'cell_alt_start'	=> '<td>',
		'cell_alt_end'		=> '</td>',

		'table_close'		=> '</table>'
	];

	$table->setTemplate($template);

.. note:: 您会注意到模板中有两组"row"块。这些允许您创建交替的行颜色或与行数据的每次迭代交替的设计元素。

您无需提交完整的模板。如果只需要更改布局的某些部分，则只需提交那些元素即可。在此示例中，仅更改了表格开始标记::

	$template = [
		'table_open' => '<table border="1" cellpadding="2" cellspacing="1" class="mytable">'
	];

	$table->setTemplate($template);
	
您还可以通过传递一系列模板设置来设置默认设置表构造器。::

	$customSettings = [
		'table_open' => '<table border="1" cellpadding="2" cellspacing="1" class="mytable">'
	];

	$table = new \CodeIgniter\View\Table($customSettings);


***************
类参考
***************

.. php:class:: Table

	.. attribute:: $function = NULL

		允许您指定要应用于所有单元格数据的本机PHP函数或有效函数数组对象。
		::

			$table = new \CodeIgniter\View\Table();

			$table->setHeading('Name', 'Color', 'Size');
			$table->addRow('Fred', '<strong>Blue</strong>', 'Small');

			$table->function = 'htmlspecialchars';
			echo $table->generate();

		在上面的示例中，所有单元格数据都将通过PHP的 :php:func:`htmlspecialchars()` 函数运行，结果是::

			<td>Fred</td><td>&lt;strong&gt;Blue&lt;/strong&gt;</td><td>Small</td>

	.. php:method:: generate([$tableData = NULL])

		:param	mixed	$tableData: 用来填充表行的数据
		:returns:	HTML 表格
		:rtype:	string

		返回包含生成的表格的字符串。接受一个可选参数，该参数可以是数组或数据库结果对象。

	.. php:method:: setCaption($caption)

		:param	string	$caption: 表格标题
		:returns:	Table 实例 (方法链)
		:rtype:	Table

		允许您在表格中添加标题。
		::

			$table->setCaption('Colors');

	.. php:method:: setHeading([$args = [] [, ...]])

		:param	mixed	$args: 包含表格列标题的数组或多个字符串
		:returns:	Table 实例 (方法链)
		:rtype:	Table

		允许您设置表格标头。您可以提交数组或离散参数::

			$table->setHeading('Name', 'Color', 'Size'); // 或

			$table->setHeading(['Name', 'Color', 'Size']);

	.. php:method:: setFooting([$args = [] [, ...]])

		:param	mixed	$args: 包含表格基础值的数组或多个字符串
		:returns:	Table 实例 (方法链)
		:rtype:	Table

		允许您设置表格基础。您可以提交数组或离散参数::

			$table->setFooting('Subtotal', $subtotal, $notes); // 或

			$table->setFooting(['Subtotal', $subtotal, $notes]);

	.. php:method:: addRow([$args = array()[, ...]])

		:param	mixed	$args: –包含行值的数组或多个字符串
		:returns:	Table 实例 (方法链)
		:rtype:	Table

		允许您在表格中添加一行。您可以提交数组或离散参数::

			$table->addRow('Blue', 'Red', 'Green'); // 或

			$table->addRow(['Blue', 'Red', 'Green']);

		如果要设置单个单元格的标记属性，则可以对该单元格使用关联数组。关联键 **data** 定义单元格的数据。其他任何 **key => val** 对将作为 ``key ='val'`` 属性添加到标签::

			$cell = ['data' => 'Blue', 'class' => 'highlight', 'colspan' => 2];
			$table->addRow($cell, 'Red', 'Green');

			// 生成
			// <td class='highlight' colspan='2'>Blue</td><td>Red</td><td>Green</td>

	.. php:method:: makeColumns([$array = [] [, $columnLimit = 0]])

		:param	array	$array: 包含多行数据的数组
		:param	int	$columnLimit: 表中的列数
		:returns:	HTML表格列的数组
		:rtype:	array

		此方法将一维数组作为输入，并创建深度等于所需列数的多维数组。这允许在具有固定列数的表格中显示包含多个元素的单个数组。考虑以下示例::

			$list = ['one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine', 'ten', 'eleven', 'twelve'];

			$newList = $table->makeColumns($list, 3);

			$table->generate($newList);

			// 生成有如下属性的表格

			<table border="0" cellpadding="4" cellspacing="0">
			<tr>
			<td>one</td><td>two</td><td>three</td>
			</tr><tr>
			<td>four</td><td>five</td><td>six</td>
			</tr><tr>
			<td>seven</td><td>eight</td><td>nine</td>
			</tr><tr>
			<td>ten</td><td>eleven</td><td>twelve</td></tr>
			</table>


	.. php:method:: setTemplate($template)

		:param	array	$template: 包含模板值的关联数组
		:returns:	成功则为TRUE，失败则为FALSE
		:rtype:	bool

		允许您设置模板。您可以提交完整或部分模板。
		::

			$template = [
				'table_open'  => '<table border="1" cellpadding="2" cellspacing="1" class="mytable">'
			];
		
			$table->setTemplate($template);

	.. php:method:: setEmpty($value)

		:param	mixed	$value: 放入空单元格的值
		:returns:	Table 实例 (方法链)
		:rtype:	Table

		使您可以设置一个默认值，以用于任何空的表单元格。例如，您可以设置一个不间断的空格::

			$table->setEmpty("&nbsp;");

	.. php:method:: clear()

		:returns:	Table 实例 (方法链)
		:rtype:	Table

		让您清除表头，行数据和标题。如果需要显示具有不同数据的多个表格，则应在生成每个表格后清除该表的先前信息，然后调用此方法。

		示例 ::

			$table = new \CodeIgniter\View\Table();


			$table->setCaption('Preferences')
                            ->setHeading('Name', 'Color', 'Size')
                            ->addRow('Fred', 'Blue', 'Small')
                            ->addRow('Mary', 'Red', 'Large')
                            ->addRow('John', 'Green', 'Medium');

			echo $table->generate();

			$table->clear();

			$table->setCaption('Shipping')
                            ->setHeading('Name', 'Day', 'Delivery')
                            ->addRow('Fred', 'Wednesday', 'Express')
                            ->addRow('Mary', 'Monday', 'Air')
                            ->addRow('John', 'Saturday', 'Overnight');

			echo $table->generate();
