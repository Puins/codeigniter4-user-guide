#############
视图渲染
#############

.. contents::
    :local:
    :depth: 2

使用视图渲染器
***************************

``view()`` 函数是一种便捷功能，可获取 ``renderer`` 服务的实例 ，设置数据并渲染视图。尽管这通常正是您想要的，但是您可能发现有需要更直接地使用它的时候。在这种情况下，您可以直接访问View服务::

	$view = \Config\Services::renderer();

或者，如果您不使用 ``View`` 类作为默认渲染器，则可以直接实例化它::

	$view = new \CodeIgniter\View\View();

.. important:: 您应该仅在控制器内创建服务。如果需要从库访问View类，则应在库的构造函数中将其设置为依赖项。

然后，您可以使用它提供的三种标准方法中的任何一种:**render(viewpath, options, save)**, **setVar(name, value, context)** and **setData(data, context)**.

它能做什么
============

``View`` 类处理存储在应用程序视图路径中的常规HTML/PHP脚本，将视图参数提取到PHP变量中后，可以在脚本中访问。这意味着视图参数名必须是合法的PHP变量名。

View类在内部使用关联数组来累积视图参数直到你调用它的 ``render()``。这意味着您的参数（或变量）名称必须是唯一的，否则以后的变量设置将覆盖以前的设置。

这还会影响脚本中不同上下文的转义参数值。您将必须为每个转义的值赋予唯一的参数名称。

值是数组的参数没有特殊含义。您可以根据自己的PHP代码适当地处理数组。

方法链
===============

`setVar()` 和 `setData()` 方法可链接，让您的一些不同的调用一起在一个链结合::

	$view->setVar('one', $one)
	     ->setVar('two', $two)
	     ->render('myView');

转义数据
=============

将数据传递给 ``setVar()`` 和 ``setData()`` 函数时，您可以选择转义数据以防止跨站点脚本攻击。作为这两种方法中的最后一个参数，您可以传递所需的上下文以转义其数据。请参阅下面的上下文描述。

如果您不想转义数据，则可以将 `null` 或 `raw` 作为最终参数传递给每个函数::

	$view->setVar('one', $one, 'raw');

如果选择不转义数据，或者要传递对象实例，则可以使用 ``esc()`` 函数在视图内手动转义数据。第一个参数是要转义的字符串。第二个参数是用于转义数据的上下文（请参见下文）::

	<?= \esc($object->getStat()) ?>

转义上下文
-----------------

默认情况下， ``esc()`` 和 ``setVar()`` 和 ``setData()`` 函数假定你想要转义的数据是为了在标准的HTML中使用。但是，如果数据打算用于Javascript，CSS或href属性中，则需要使用不同的转义规则才能生效。您可以传入上下文名称作为第二个参数。有效上下文是 'html', 'js', 'css', 'url', and 'attr'::

	<a href="<?= esc($url, 'url') ?>" data-foo="<?= esc($bar, 'attr') ?>">Some Link</a>

	<script>
		var siteName = '<?= esc($siteName, 'js') ?>';
	</script>

	<style>
		body {
			background-color: <?= esc('bgColor', 'css') ?>
		}
	</style>

视图渲染选项
=====================

可以将几个选项传递给 ``render()`` 或 ``renderString()`` 方法:

-   ``cache`` - 保存视图结果的时间（以秒为单位）；对renderString()忽略
-   ``cache_name`` - 用于保存/检索缓存的视图结果的ID；默认为viewpath；对renderString()忽略
-   ``saveData`` - 如果要保留视图数据参数以供后续调用，则为true

类参考
***************

.. php:class:: CodeIgniter\\View\\View

	.. php:method:: render($view[, $options[, $saveData=false]]])
                :noindex:

		:param  string  $view: 视图源的文件名
		:param  array   $options: 键/值对选项数组
		:param  boolean $saveData: 如果为true，则将保存数据以供任何其他调用使用；如果为false，则在渲染视图后将数据清理。 
		:returns: 所选视图的渲染文本
		:rtype: string

		根据文件名和已设置的任何数据构建输出::

			echo $view->render('myview');

	.. php:method:: renderString($view[, $options[, $saveData=false]]])
                :noindex:

		:param  string  $view: 要渲染的视图内容，例如从数据库检索的内容
		:param  array   $options: 键/值对选项数组
		:param  boolean $saveData: 如果为true，则将保存数据以供任何其他调用使用；如果为false，则在渲染视图后将数据清理。 
		:returns: 所选视图的渲染文本
		:rtype: string

		根据视图片段和任何已设置的数据构建输出::

			echo $view->renderString('<div>My Sharona</div>');

		这可以用于显示可能已经存储在数据库中的内容，但是您需要意识到这是一个潜在的安全漏洞，并且必须验证任何此类数据，并可能适当地对其进行转义！

	.. php:method:: setData([$data[, $context=null]])
                :noindex:

		:param  array   $data: 键/值视图数据字符串的数组
		:param  string  $context: 用于数据转义的上下文。
		:returns: 渲染器，用于方法
		:rtype: CodeIgniter\\View\\RendererInterface.

		一次设置几条视图数据::

			$view->setData(['name'=>'George', 'position'=>'Boss']);

		支持的转义上下文：html，css，js，url或attr或raw。如果是'raw'，将不会发生转义。

		每次调用都会将对象累积的数据添加到数组中，直到渲染视图为止。

	.. php:method:: setVar($name[, $value=null[, $context=null]])
                :noindex:

		:param  string  $name: 视图数据变量的名称
		:param  mixed   $value: 此视图数据的值
		:param  string  $context: 用于数据转义的上下文。
		:returns: 渲染器，用于方法
		:rtype: CodeIgniter\\View\\RendererInterface.

		设置单个视图数据::

			$view->setVar('name','Joe','html');

		支持的转义上下文：html，css，js，url或attr或raw。如果是'raw'，将不会发生转义。

		如果使用以前用于该对象的视图数据变量，则新值将替换现有值。
