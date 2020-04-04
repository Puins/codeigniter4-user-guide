#####
视图
#####

.. contents::
    :local:
    :depth: 2

视图只是一个网页或页面片段，例如页眉，页脚，侧边栏等。实际上，如果您需要这种类型的层次结构，则视图可以灵活地嵌入其他视图中（在其他视图等内部）。

视图从不直接调用，它们必须由控制器加载。请记住，在MVC框架中，Controller充当交通警察，因此它负责获取特定视图。如果您尚未阅读 :doc:`Controllers </incoming/controllers>` 页面，则应先阅读该内容，然后再继续。

使用您在控制器页面中创建的示例控制器，让我们为其添加一个视图。

创建视图
===============

使用文本编辑器，创建一个名为的文件 ``BlogView.php``，代码如下::

	<html>
        <head>
            <title>My Blog</title>
        </head>
        <body>
            <h1>Welcome to my Blog!</h1>
        </body>
	</html>

然后将文件保存在您的 **app/Views** 目录中。

显示视图
=================

要加载并显示特定的视图文件，您将使用以下函数::

	echo view('name');

其中 *name* 是您的视图文件的名称。

.. important:: 如果省略文件扩展名，则视图应以.php扩展名结尾。

现在，打开您之前创建的控制器文件 ``Blog.php``，并将echo语句替换为view函数::

	<?php namespace App\Controllers;

        class Blog extends \CodeIgniter\Controller
	{
		public function index()
		{
			echo view('BlogView');
		}
	}

如果您使用以前访问过的URL访问站点，则应该会看到新视图。该URL类似于以下内容::

	example.com/index.php/blog/

.. note:: 尽管所有示例均直接回显视图，但是您也可以从视图返回输出，并将其附加到任何捕获的输出。

加载多个视图
======================

CodeIgniter将智能地处理 ``view()`` 来自控制器内的多个调用。如果发生多个调用，它们将被附加在一起。例如，您可能希望具有页眉视图，菜单视图，内容视图和页脚视图。可能看起来像这样::

	<?php namespace App\Controllers;

	class Page extends \CodeIgniter\Controller
	{
		public function index()
		{
			$data = [
				'page_title' => 'Your title'
			];

			echo view('header');
			echo view('menu');
			echo view('content', $data);
			echo view('footer');
		}
	}

在上面的示例中，我们使用的是“动态添加的数据”，您将在下面看到。

在子目录中存储视图
====================================

如果您喜欢这种类型的组织，则视图文件也可以存储在子目录中。这样做时，您将需要包括加载视图的目录名称。示例::

	echo view('directory_name/file_name');

命名空间视图
================

您可以将视图存储在已命名空间的 **View** 目录下，并像加载命名空间一样加载该视图。虽然PHP不支持从命名空间加载非类文件，但CodeIgniter提供了此功能，使您可以将它们以类似于模块的方式打包在一起，以便于重用或分发。

如果您的 ``Blog`` 目录在 :doc:`Autoloader </concepts/autoloader>` 中位于命名空间下，并且有在其中设置了PSR-4映射的目录，则 ``Example\Blog`` 可以检索视图文件，就像它们也被命名为命名空间一样。在此示例之后，可以通过在命名空间前添加视图名称来从 **/blog/views** 加载 **BlogView** 文件::

    echo view('Example\Blog\Views\BlogView');

缓存视图
=============

您可以通过 ``view`` 命令传递cache选项用于缓存视图的秒数来缓存视图::

    // 将视图缓存60秒
    echo view('file_name', $data, ['cache' => 60]);

默认情况下，将使用与视图文件本身相同的名称来缓存视图。您可以通过传递 ``cache_name`` 和希望使用的缓​​存ID来对此进行自定义::

    // 将视图缓存60秒
    echo view('file_name', $data, ['cache' => 60, 'cache_name' => 'my_cached_view']);

向视图添加动态数据
===============================

数据通过视图函数第二个参数中的数组从控制器传递到视图。这是一个示例::

	$data = [
		'title'   => 'My title',
		'heading' => 'My Heading',
		'message' => 'My Message'
	];

	echo view('blogview', $data);

让我们用您的控制器文件尝试一下。打开它并添加以下代码::

	<?php namespace App\Controllers;

	class Blog extends \CodeIgniter\Controller
	{
		public function index()
		{
			$data['title']   = "My Real Title";
			$data['heading'] = "My Real Heading";

			echo view('blogview', $data);
		}
	}

现在打开视图文件，并将文本更改为与数据中的数组键对应的变量::

	<html>
        <head>
            <title><?= $title ?></title>
        </head>
        <body>
            <h1><?= $heading ?></h1>
        </body>
	</html>

然后将页面加载到您一直在使用的URL，您应该会看到变量已替换。

传入的数据仅在当次调用 `view` 期间可用。如果在单个请求中多次调用该函数，则必须将所需的数据传递给每个视图。这样可以防止任何数据“渗入”其他视图，从而可能导致问题。如果希望保留数据，则可以将 `saveData` 选项传递到第三个参数的 `$option` 数组中。

::

	$data = [
		'title'   => 'My title',
		'heading' => 'My Heading',
		'message' => 'My Message'
	];

	echo view('blogview', $data, ['saveData' => true]);

另外，如果您希望view方法的默认功能是在调用之间保存数据，则可以在 **app/Config/Views.php** 中将 ``$saveData`` 设置为 **true**。

创建循环
==============

传递给视图文件的数据数组不仅限于简单变量。您可以传递多维数组，可以循环生成多个行。例如，如果您从数据库中提取数据，则数据通常将采用多维数组的形式。

这是一个简单的示例。将此添加到您的控制器::

	<?php namespace App\Controllers;

	class Blog extends \CodeIgniter\Controller
	{
		public function index()
		{
			$data = [
				'todo_list' => ['Clean House', 'Call Mom', 'Run Errands'],
				'title'     => "My Real Title",
				'heading'   => "My Real Heading"
			];

			echo view('blogview', $data);
		}
	}

现在打开视图文件并创建一个循环::

	<html>
	<head>
		<title><?= $title ?></title>
	</head>
	<body>
		<h1><?= $heading ?></h1>

		<h3>My Todo List</h3>

		<ul>
		<?php foreach ($todo_list as $item):?>

			<li><?= $item ?></li>

		<?php endforeach;?>
		</ul>

	</body>
	</html>
