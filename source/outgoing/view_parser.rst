###########
视图解析
###########

.. contents::
    :local:
    :depth: 3

视图解析可以对视图文件中包含的伪变量执行简单的文本替换。它可以解析简单变量或变量标签对。

伪变量名称或控制结构用大括号括起来，如下所示::

	<html>
	<head>
		<title>{blog_title}</title>
	</head>
	<body>
		<h3>{blog_heading}</h3>

		{blog_entries}
			<h5>{title}</h5>
			<p>{body}</p>
		{/blog_entries}

	</body>
	</html>

这些变量不是实际的PHP变量，而是纯文本表示形式，使您可以从模板（视图文件）中消除PHP。

.. note:: CodeIgniter也不会要求你使用这个类，因为在您的视图页面使用纯PHP（例如使用 :doc:`视图渲染器 </outgoing/view_renderer>`）让他们跑快一点。但是，如果开发人员与认为使用PHP会感到有些困惑的设计师合作，则他们更喜欢使用某种形式的模板引擎。

***************************
使用视图解析类
***************************

加载解析器类的最简单方法是通过其服务::

	$parser = \Config\Services::parser();

或者，如果您不使用 ``Parser`` 类作为默认渲染器，则可以直接实例化它::

	$parser = new \CodeIgniter\View\Parser();

然后，您可以使用它提供的三种标准渲染方法中的任何一种： **render(viewpath, options, save)**, **setVar(name, value, context)** 和 **setData(data, context)**。您还可以通过 **setDelimiters(left,right)** 方法直接指定定界符。

使用 ``Parser``，您的视图模板仅由解析器本身处理，而不像常规的视图PHP脚本那样处理。此类脚本中的PHP代码将被解析器忽略，并且仅执行替换操作。

这是有目的的：没有PHP的视图文件。

它能做什么
============

``Parser`` 类处理存储在应用程序视图路径中的“PHP/HTML脚本”。这些脚本不能包含任何PHP。

每个视图参数（我们称为伪变量）都会根据您为其提供的值的类型触发替换。伪变量不会提取到PHP变量中；相反，它们的值通过伪变量语法访问，其中其名称在花括号内引用。

Parser类在内部使用关联数组，以累积伪变量设置，直到调用它的 ``render()`` 为止。这意味着您的伪变量名称必须唯一，否则以后的参数设置将覆盖以前的参数。

这还会影响脚本中不同上下文的转义参数值。您将必须为每个转义的值赋予唯一的参数名称。

解析器模板
================

您可以使用 ``render()`` 方法来解析（或渲染）简单模板，如下所示::

	$data = [
		'blog_title'   => 'My Blog Title',
		'blog_heading' => 'My Blog Heading'
	];

	echo $parser->setData($data)
	             ->render('blog_template');

视图参数传递给 ``setData()`` 作为在模板要替换的数据的关联数组。在上面的示例中，模板将包含两个变量：{blog_title}和{blog_heading}第一个参数 ``render()`` 包含 :doc:`视图文件 </outgoing/views>` 的名称，其中 *blog_template* 是视图文件的名称。

.. important:: 如果省略文件扩展名，则视图应以 ``.php`` 扩展名结尾。

解析器配置选项
============================

可以将几个选项传递给 ``render()`` 或 ``renderString()`` 方法。

-   ``cache`` - 保存视图结果的时间（以秒为单位）；对renderString()忽略
-   ``cache_name`` - 用于保存/检索缓存的视图结果的ID；默认为viewpath；对renderString()忽略
-   ``saveData`` - 如果要保留视图数据参数以供后续调用，则为true;默认为 **false**
-	``cascadeData`` - 如果伪变量设置应传递给嵌套替换，则为true；默认值为**true**

::

	echo $parser->render('blog_template', [
		'cache'      => HOUR,
		'cache_name' => 'something_unique',
	]);

***********************
替换变量
***********************

支持三种类型的替换：简单，循环和嵌套。替换按照添加伪变量的相同顺序执行。

解析器执行的 **简单替换** 是伪变量的一对一替换，其中对应的数据参数具有标量或字符串值，如本例所示::

	$template = '<head><title>{blog_title}</title></head>';
	$data     = ['blog_title' => 'My ramblings'];

	echo $parser->setData($data)->renderString($template);

	// 结果: <head><title>My ramblings</title></head>

“Parser”进一步用“变量对”替换，用于嵌套替换或循环，并与一些高级条件替换的构造。

解析器执行时，通常

-	处理任何条件替换
-	处理任何嵌套/循环替换
-	处理剩余的单个替换

循环替换
==================

当伪变量的值是数组的顺序数组（如行设置的数组）时，将发生循环替换。

上面的示例代码允许替换简单变量。如果您希望重复整个变量块，而每次迭代都包含新值，该怎么办？考虑我们在页面顶部显示的模板示例::

	<html>
	<head>
		<title>{blog_title}</title>
	</head>
	<body>
		<h3>{blog_heading}</h3>

		{blog_entries}
			<h5>{title}</h5>
			<p>{body}</p>
		{/blog_entries}

	</body>
	</html>

在上面的代码中，您会注意到一对变量：{blog_entries}数据…{/blog_entries}。在这种情况下，这些对之间的整个数据块将被重复多次，这与参数数组的“ blog_entries”元素中的行数相对应。

使用上面显示的相同代码来解析单个变量，以解析变量对，但是您将添加一个与变量对数据相对应的多维数组。考虑以下示例::

	$data = [
		'blog_title'   => 'My Blog Title',
		'blog_heading' => 'My Blog Heading',
		'blog_entries' => [
			['title' => 'Title 1', 'body' => 'Body 1'],
			['title' => 'Title 2', 'body' => 'Body 2'],
			['title' => 'Title 3', 'body' => 'Body 3'],
			['title' => 'Title 4', 'body' => 'Body 4'],
			['title' => 'Title 5', 'body' => 'Body 5']
		]
	];

	echo $parser->setData($data)
	             ->render('blog_template');

伪变量 ``blog_entries`` 的值是关联数组的顺序数组。外层没有与每个嵌套“行”关联的键。

如果“对”数据来自数据库结果（已经是一个多维数组），则可以简单地使用数据库 ``getResultArray()`` 方法::

	$query = $db->query("SELECT * FROM blog");

	$data = [
		'blog_title'   => 'My Blog Title',
		'blog_heading' => 'My Blog Heading',
		'blog_entries' => $query->getResultArray()
	];

	echo $parser->setData($data)
	             ->render('blog_template');

如果您要循环的数组包含对象而不是数组，则解析器将首先在对象上查找 ``asArray`` 方法。如果存在，则将调用该方法，然后如上所述将结果数组循环。如果不存在 ``asArray`` 方法，则该对象将被强制转换为数组，并且其公共属性将对解析器可用。

这对于Entity类尤其有用，它具有一个asArray方法，该方法返回所有公共和受保护的属性（减去_options属性）并使它们对解析器可用。

嵌套替换
====================

当伪变量的值是值的关联数组（例如数据库中的记录）时，就会发生嵌套替换::

	$data = [
		'blog_title'   => 'My Blog Title',
		'blog_heading' => 'My Blog Heading',
		'blog_entry'   => [
			'title' => 'Title 1', 'body' => 'Body 1'
		]
	];

	echo $parser->setData($data)
	             ->render('blog_template');

伪变量 ``blog_entry`` 的值是一个关联数组。在其中定义的键/值对将在该变量的变量对循环内公开。

``blog_template`` 可能适用于上述情况::

	<h1>{blog_title} - {blog_heading}</h1>
	{blog_entry}
		<div>
			<h2>{title}</h2>
			<p>{body}</p>
		</div>
	{/blog_entry}

如果您希望在“blog_entry”范围内可以访问其他伪变量，请确保将“cascadeData”选项设置为true。

注释
========

您可以通过将注释包装在 ``{#  #}`` 符号中，将注释放置在模板中，这些注释在解析过程中将被忽略和删除。

::

	{# 注释在解析过程中被删除。 #}
	{blog_entry}
		<div>
			<h2>{title}</h2>
			<p>{body}</p>
		</div>
	{/blog_entry}

级联数据
==============

通过嵌套和循环替换，您可以选择将数据对级联到内部替换中。

以下示例不受级联的影响::

	$template = '{name} lives in {location}{city} on {planet}{/location}.';

	$data = [
		'name'     => 'George',
		'location' => [ 'city' => 'Red City', 'planet' => 'Mars' ]
	];

	echo $parser->setData($data)->renderString($template);
	// 结果: George lives in Red City on Mars.

此示例根据级联给出不同的结果::

	$template = '{location}{name} lives in {city} on {planet}{/location}.';

	$data = [
		'name'     => 'George',
		'location' => [ 'city' => 'Red City', 'planet' => 'Mars' ]
	];

	echo $parser->setData($data)->renderString($template, ['cascadeData'=>false]);
	// 结果: {name} lives in Red City on Mars.

	echo $parser->setData($data)->renderString($template, ['cascadeData'=>true]);
	// 结果: George lives in Red City on Mars.

防止解析
==================

您可以使用 ``{noparse}{/noparse}`` 标记对指定页面的某些部分不进行解析。本节中的所有内容将保持原样，并且括号之间的标记不会发生任何变量替换，循环等。

::

	{noparse}
		<h1>Untouched Code</h1>
	{/noparse}

条件逻辑
=================

解析器类支持一些基本的条件语句来处理 ``if``, ``else``, 和 ``elseif`` 语法。所有 ``if`` 块必须使用 ``endif`` 标签关闭::

	{if $role=='admin'}
		<h1>Welcome, Admin!</h1>
	{endif}

在解析过程中，此简单块将转换为以下内容::

	<?php if ($role=='admin'): ?>
		<h1>Welcome, Admin!</h1>
	<?php endif ?>

if语句中使用的所有变量必须事先设置为相同的名称。除此之外，它被完全视为标准PHP条件，所有标准PHP规则都将在此处适用。你可以使用你通常会用任何比较运算符，像 ``==``, ``===``, ``!==``, ``<``, ``>``, 等。

::

	{if $role=='admin'}
		<h1>Welcome, Admin</h1>
	{elseif $role=='moderator'}
		<h1>Welcome, Moderator</h1>
	{else}
		<h1>Welcome, User</h1>
	{endif}

.. note:: 在后台，使用 **eval()** 解析条件语句，因此您必须确保注意在条件语句中使用的用户数据，否则可能会使应用程序面临安全风险。

转义数据
=============

默认情况下，所有变量替换均被转义，以帮助防止对页面进行XSS攻击。CodeIgniter的 ``esc`` 方法支持几种不同的上下文，例如常规 **html**，如果它位于HTML  **attr** ，**css** 等中。如果未指定其他内容，则将假定数据位于HTML上下文中。您可以使用 **esc** 过滤器指定使用的上下文::

	{ user_styles | esc(css) }
	<a href="{ user_link | esc(attr) }">{ title }</a>

有时候，您绝对需要使用某些东西而又不能转义。您可以通过在左括号和右括号中添加感叹号(!)来做到这一点::

	{! unescaped_var !}

过滤器
=======

任何单个变量替换都可以应用一个或多个过滤器以修改其表示方式。这些并不是要大幅度改变输出，而是提供重用相同变量数据但使用不同表示的方法。上面讨论的 **esc** 过滤器是一个示例。日期是另一个常见用例，您可能需要在同一页面上的多个部分中以不同的方式设置相同数据的格式。

过滤器是伪变量名称之后的命令，并用竖线(``|``)符号分隔::

	// -55 被显示为 55
	{ value|abs  }

如果参数接受任何参数，则必须用逗号(,)分隔并用括号()括起来::

	{ created_at|date(Y-m-d) }

通过将多个过滤器管道连接在一起，可以将多个过滤器应用于该值。它们从左到右依次处理::

	{ created_at|date_modify(+5 days)|date(Y-m-d) }

提供过滤器
----------------

使用解析器时，可以使用以下过滤器:

+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ **Filter**    + **参数**            + **描述**                                                     + **示例**                            +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ abs           +                     + 显示数字的绝对值。                                           + { v|abs }                           +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ capitalize    +                     + 在以下情况下显示字符串：首字母大写的所有小写字母。           + { v|capitalize}                     +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ date          + format (Y-m-d)      + PHP **date**-兼容的格式字符串。                              + { v|date(Y-m-d) }                   +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ date_modify   + value to add        + 与 **strtotime** 兼容的字符串，用于修改日期，例如            + { v|date_modify(+1 day) }           +
+               + / subtract          +  ``+5 天`` 或 ``-1 周``.                                     +                                     +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ default       + default value       + 如果变量为空或未定义，则显示默认值。                         + { v|default(just in case) }         +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ esc           + html, attr, css, js + 指定用于转义数据的上下文。                                   + { v|esc(attr) }                     +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ excerpt       + phrase, radius      + 返回给定短语中单词半径内的文本。与 **excerpt** 辅助函数相同。+ { v|excerpt(green giant, 20) }      +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ highlight     + phrase              + 使用'<mark></mark>'标签突出显示文本中的给定短语。            + { v|highlight(view parser) }        +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ highlight_code+                     + 用HTML / CSS突出显示代码示例。                               + { v|highlight_code }                +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ limit_chars   + limit               + 将字符数限制为$limit。                                       + { v|limit_chars(100) }              +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ limit_words   + limit               + 将字数限制为$limit。                                         + { v|limit_words(20) }               +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ local_currency+ currency, locale    +显示货币的本地化版本。“货币”值是任何3个字母的ISO4217货币代码。+ { v|local_currency(EUR,en_US) }     +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ local_number  + type, precision,    + 显示数字的本地化版本。“类型”可以是以下之一 : decimal,        + { v|local_number(decimal,2,en_US) } +
+               + locale              +  currency, percent, scientific, spellout, ordinal, duration。+                                     +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ lower         +                     + 将字符串转换为小写。                                         + { v|lower }                         +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ nl2br         +                     + 将所有换行符(\n)替换为HTML <br/>标记。                       + { v|nl2br }                         +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ number_format + places              + 包装PHP **number_format** 函数以在解析器中使用。             + { v|number_format(3) }              +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ prose         +                     + 接收一段文本，并使用 **auto_typography()** 方法将其转换为    + { v|prose }                         +
+               +                     + 更漂亮，更易于阅读的散文。                                   +                                     +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ round         + places, type        + 将数字四舍五入到指定位置。可以传递 **ceil** 和 **floor**     + { v|round(3) } { v|round(ceil) }    +
+               +                     + 类型来代替使用这些功能。                                     +                                     +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ strip_tags    + allowed chars       + 包装PHP **strip_tags**。可以接受一串允许的标签。             + { v|strip_tags(<br>) }              +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ title         +                     +显示字符串的“标题大小写”版本，所有字母均小写，每个单词均大写。+ { v|title }                         +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+
+ upper         +                     + 以大写形式显示字符串。                                       + { v|upper }                         +
+---------------+---------------------+--------------------------------------------------------------+-------------------------------------+

有关"local_number"过滤器的详细信息，请参见PHP的 `PHP's NumberFormatter <https://www.php.net/manual/en/numberformatter.create.php>`_。

自定义过滤器
--------------

您可以通过编辑 **app/Config/View.php** 并将新条目添加到 ``$filters`` 数组中来轻松创建自己的过滤器。每个键都是在视图中被调用的过滤器的名称，其值是任何可调用的有效PHP::

	public $filters = [
		'abs'        => '\CodeIgniter\View\Filters::abs',
		'capitalize' => '\CodeIgniter\View\Filters::capitalize',
	];

PHP本机函数用作过滤器
-------------------------------

您可以通过编辑 **app/Config/View.php** 并将新条目添加到 ``$filters`` 数组中来使用本机php函数作为过滤器，每个键是在视图中被调用的本机PHP函数的名称，其值是任何有效的本机PHP函数前缀::

	public $filters = [
		'str_repeat' => '\str_repeat',
	];

解析器插件
==============

插件允许您扩展解析器，为每个项目添加自定义功能。它们可以是任何可调用的PHP，从而使它们的实现非常简单。在模板内，插件由 ``{+ +}`` 标签指定::

	{+ foo +} inner content {+ /foo +}

此示例显示了一个名为 **foo** 的插件。它可以在其开始标记和结束标记之间操纵任何内容。在此示例中，它可以使用文本“内部内容”。在进行任何伪变量替换之前，将先处理插件。

尽管插件通常由标签对组成，如上所示，但它们也可以是单个标签，没有结束标签::

	{+ foo +}

开头标签还可以包含可以自定义插件工作方式的参数。参数表示为键/值对::

	{+ foo bar=2 baz="x y" }

参数也可以是单个值::

	{+ include somefile.php +}

提供插件
----------------

使用解析器时，可以使用以下插件:

==================== ========================== ==================================================== ========================================================================
Plugin               参数                        描述                                                  示例
==================== ========================== ==================================================== ========================================================================
current_url                                     current_url辅助函数的别名。                             {+ current_url +}
previous_url                                    previous_url辅助函数的别名。                            {+ previous_url +}
siteURL                                         site_url辅助函数的别名。                                {+ siteURL "login" +}
mailto               email, title, attributes   mailto辅助函数的别名。                                  {+ mailto email=foo@example.com title="Stranger Things" +}
safe_mailto          email, title, attributes   safe_mailto辅助函数的别名。                             {+ safe_mailto email=foo@example.com title="Stranger Things" +}
lang                 language string            lang辅助函数的别名。                                    {+ lang number.terabyteAbbr +}
validation_errors    fieldname(optional)        返回该字段的错误字符串（如果已指定）或所有验证错误。          {+ validation_errors +} , {+ validation_errors field="email" +}
route                route name                 route_to辅助函数的别名。                                {+ route "login" +}
==================== ========================== ==================================================== ========================================================================

注册插件
--------------------

最简单的方法是，注册一个新插件并准备就绪可以使用，只需将其添加到 **app/Config/View.php** 下的 **$plugins** 数组中。key是在模板文件中使用的插件的名称。该值是任何可调用的有效PHP，包括静态类方法和闭包::

	public $plugins = [
		'foo'	=> '\Some\Class::methodName',
		'bar'	=> function($str, array $params=[]) {
			return $str;
		},
	];

必须在配置文件的构造函数中定义正在使用的所有闭包::

    class View extends \CodeIgniter\Config\View
    {
        public $plugins = [];

        public function __construct()
        {
            $this->plugins['bar'] = function(array $params=[]) {
                return $params[0] ?? '';
            };

            parent::__construct();
        }
    }

如果可调用项是单独的，则将其视为单个标签，而不是打开/关闭标签。它将由插件的返回值替换::

	public $plugins = [
		'foo'	=> '\Some\Class::methodName'
	];

	// 标签由Some\Class::methodName静态函数的返回值替换。
	{+ foo +}

如果可调用对象包装在数组中，则将其视为可以在其标签之间的任何内容上进行操作的打开/关闭标签对::

	public $plugins = [
		'foo' => ['\Some\Class::methodName']
	];

	{+ foo +} inner content {+ /foo +}

***********
用法说明
***********

如果您包含模板中未引用的替换参数，则将忽略它们::

	$template = 'Hello, {firstname} {lastname}';
	$data = [
		'title' => 'Mr',
		'firstname' => 'John',
		'lastname' => 'Doe'
	];
	echo $parser->setData($data)
	             ->renderString($template);

	// 结果: Hello, John Doe

如果您不包括模板中引用的替换参数，则原始伪变量将显示在结果中::

	$template = 'Hello, {firstname} {initials} {lastname}';
	$data = [
		'title'     => 'Mr',
		'firstname' => 'John',
		'lastname'  => 'Doe'
	];
	echo $parser->setData($data)
	             ->renderString($template);

	// 结果: Hello, John {initials} Doe

如果在需要数组时（例如，对变量对）提供字符串替换参数，则将对开头变量对标记进行替换，但结尾变量对标记将无法正确呈现::

	$template = 'Hello, {firstname} {lastname} ({degrees}{degree} {/degrees})';
	$data = [
		'degrees'   => 'Mr',
		'firstname' => 'John',
		'lastname'  => 'Doe',
		'titles'    => [
			['degree' => 'BSc'],
			['degree' => 'PhD']
		]
	];
	echo $parser->setData($data)
	             ->renderString($template);

	// 结果: Hello, John Doe (Mr{degree} {/degrees})

视图参数
==============

您不必使用变量对就可以在视图中获得迭代的效果。可以将视图片段用于变量对内部，并在控制器中而不是视图中控制迭代。

在视图中控制迭代的示例::

	$template = '<ul>{menuitems}
		<li><a href="{link}">{title}</a></li>
	{/menuitems}</ul>';

	$data = [
		'menuitems' => [
			['title' => 'First Link', 'link' => '/first'],
			['title' => 'Second Link', 'link' => '/second'],
		]
	];
	echo $parser->setData($data)
	             ->renderString($template);

结果::

	<ul>
		<li><a href="/first">First Link</a></li>
		<li><a href="/second">Second Link</a></li>
	</ul>

一个使用视图片段在控制器中控制迭代的示例::

	$temp = '';
	$template1 = '<li><a href="{link}">{title}</a></li>';
	$data1 = [
		['title' => 'First Link', 'link' => '/first'],
		['title' => 'Second Link', 'link' => '/second'],
	];

	foreach ($data1 as $menuItem)
	{
		$temp .= $parser->setData($menuItem)->renderString($template1);
	}

	$template2 = '<ul>{menuitems}</ul>';
	$data = [
		'menuitems' => $temp
	];
	echo $parser->setData($data)
	             ->renderString($template2);

结果::

	<ul>
		<li><a href="/first">First Link</a></li>
		<li><a href="/second">Second Link</a></li>
	</ul>

***************
类参考
***************

.. php:class:: CodeIgniter\\View\\Parser

	.. php:method:: render($view[, $options[, $saveData=false]]])

		:param  string  $view: 视图源的文件名
		:param  array   $options: 键/值对选项数组
		:param  boolean $saveData: 如果为true，则将保存数据以供任何其他调用使用；如果为false，则在呈现视图后将清理数据。
		:returns: 所选视图的渲染文本
		:rtype: string

    	根据文件名和已设置的任何数据构建输出::

			echo $parser->render('myview');

        支持的选项:

	        -   ``cache`` - 以秒为单位的时间，用于保存视图的结果
	        -   ``cache_name`` - 用于保存/检索缓存的视图结果的ID；默认为viewpath
	        -   ``cascadeData`` - 如果要发生嵌套或循环替换时有效的数据对，则为true
	        -   ``saveData`` - 如果要保留视图数据参数以供后续调用，则为true
	        -   ``leftDelimiter`` - 在伪变量语法中使用的左定界符
	        -   ``rightDelimiter`` - 在伪变量语法中使用的右定界符

	首先执行任何条件替换，然后对每个数据对执行其余替换。

	.. php:method:: renderString($template[, $options[, $saveData=false]]])

		:param  string  $template: 以字符串形式提供的视图源
		:param  array   $options: 键/值对选项数组
		:param  boolean $saveData: 如果为true，则将保存数据以供任何其他调用使用；如果为false，则在呈现视图后将清理数据。
		:returns: 所选视图的渲染文本
		:rtype: string

    	根据提供的模板源和任何已设置的数据来构建输出::

			echo $parser->render('myview');

        支持的选项和行为如上所述。

	.. php:method:: setData([$data[, $context=null]])

		:param  array   $data: 视图数据字符串的数组，作为键/值对
		:param  string  $context: 用于数据转义的上下文。
		:returns: 渲染器，用于方法链
		:rtype: CodeIgniter\\View\\RendererInterface.

    	一次设置几条视图数据::

			$renderer->setData(['name'=>'George', 'position'=>'Boss']);

        支持的转义上下文: html, css, js, url, 或 attr 或 raw。如果'raw'，将不会发生转义。		

	.. php:method:: setVar($name[, $value=null[, $context=null]])

		:param  string  $name: 视图数据变量的名称
		:param  mixed   $value: 视图数据的值
		:param  string  $context: 用于数据转义的上下文。
		:returns: 渲染器，用于方法链
		:rtype: CodeIgniter\\View\\RendererInterface.

    	设置单个视图数据::

			$renderer->setVar('name','Joe','html');

        支持的转义上下文: html, css, js, url, 或 attr 或 raw。如果'raw'，将不会发生转义。

	.. php:method:: setDelimiters($leftDelimiter = '{', $rightDelimiter = '}')

		:param  string  $leftDelimiter: 替换字段的左定界符
		:param  string  $rightDelimiter: 替换字段的右定界符
		:returns: 渲染器，用于方法链
		:rtype: CodeIgniter\\View\\RendererInterface.

    	重写替换字段右定界符::

			$renderer->setDelimiters('[',']');
