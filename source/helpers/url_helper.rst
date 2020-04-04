###############
URL 辅助函数
###############

URL 辅助函数文件包含协助使用URL的功能。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

加载此辅助函数
===================

框架会在每次请求时自动加载该辅助函数。

可用功能
===================

以下功能可用：

.. php:function:: site_url([$uri = ''[, $protocol = NULL[, $altConfig = NULL]]])

	:param	mixed	$uri: URI字符串或URI段数组
	:param	string	$protocol: 协议, 例如， 'http' 或 'https'
	:param	\\Config\\App	$altConfig: 要使用的备用配置
	:returns:	站点 URL
	:rtype:	string

	返回配置文件中指定的站点URL。index.php文件（或您在配置文件中设置为站点 **indexPage** 的任何文件）将被添加到URL，传递给该函数的所有URI段也将被添加到URL。

	我们鼓励您在需要生成本地URL的任何时候使用此功能，以使您的页面在URL发生更改时变得更加可移植。

	段可以选择作为字符串或数组传递给函数。这是一个字符串示例::

		echo site_url('news/local/123');

	上面的示例将返回以下内容:
	*http://example.com/index.php/news/local/123*

	这是作为数组传递的段的示例::

		$segments = ['news', 'local', '123'];
		echo site_url($segments);

        如果为不同于您的站点的站点生成URL（其中包含不同的配置首选项），则备用配置可能会很有用。我们使用它来对框架本身进行单元测试。

.. php:function:: base_url([$uri = ''[, $protocol = NULL]])

	:param	mixed	$uri: URI字符串或URI段数组
	:param	string	$protocol: 协议, 例如， 'http' 或 'https'
	:returns:	基本 URL
	:rtype:	string

	返回配置文件中指定的站点基本URL。 示例::

		echo base_url();

	此函数返回与 :php:func:`site_url()` 相同的东西，但未附加indexPage。

	与 :php:func:`site_url()` 一样，你能提供程序段为字符串或数组。这是一个字符串示例::

		echo base_url("blog/post/123");

	上面的示例将返回如下内容:
	*http://example.com/blog/post/123*

	这很有用，因为与 :php:func:`site_url()` 不同，您可以向文件（例如图像或样式表）提供字符串。对于示例::

		echo base_url("images/icons/edit.png");

	这将为您提供类似的信息:
	*http://example.com/images/icons/edit.png*

.. php:function:: current_url([$returnObject = false])

	:param	boolean	$returnObject: 如果您想返回一个URI实例而不是字符串，则为True。
	:returns:	当前 URL
	:rtype:	string|URI

	返回当前正在查看的页面的完整URL（包含程序段）。

	.. note:: 调用此函数与执行此操作相同::
		base_url(uri_string());

.. php:function:: previous_url([$returnObject = false])

	:param boolean $returnObject: 如果希望返回一个URI实例而不是字符串，则为True。
	:returns: 用户以前使用的URL
	:rtype: string|URI

	返回用户先前所在页面的完整URL（包含程序段）。

	由于盲目地信任HTTP_REFERER系统变量的安全性问题，CodeIgniter将在会话中存储以前访问的页面（如果可用）。这样可以确保我们始终使用已知且受信任的来源。如果尚未加载会话，或者会话不可用，则将使用HTTP_REFERER的清理版本。

.. php:function:: uri_string()

	:returns:	URI字符串
	:rtype:	string

	返回当前URL的路径部分。例如，如果您的URL是这样的::

		http://some-site.com/blog/comments/123

	该函数将返回::

		blog/comments/123

.. php:function:: index_page([$altConfig = NULL])

	:param	\Config\App	$altConfig: 要使用的备用配置
	:returns:	'index_page' 值
	:rtype:	mixed

	返回配置文件中指定的站点 **indexPage**。示例::

		echo index_page();

	与 :php:func:`site_url()` 一样，您可以指定备用配置。如果为不同于您的站点的站点生成URL（其中包含不同的配置首选项），则备用配置可能会很有用。我们使用它来对框架本身进行单元测试。

.. php:function:: anchor([$uri = ''[, $title = ''[, $attributes = ''[, $altConfig = NULL]]]])

	:param	mixed	$uri: URI字符串或URI段数组
	:param	string	$title: 锚标题
	:param	mixed	$attributes: HTML属性
	:param	\Config\App	$altConfig: 要使用的备用配置
	:returns:	HTML 超链接 (anchor 标记)
	:rtype:	string

	根据您的本地站点URL创建标准的HTML锚链接。

	第一个参数可以包含您希望附加到URL的任何段。与上面的 :php:func:`site_url()` 函数一样，段可以是字符串或数组。

	.. note:: 如果要构建应用程序内部的链接，请不要包含基本URL(`http://...`)。这将从配置文件中指定的信息自动添加。仅包括您希望附加到URL的URI段。

	第二部分是您想要链接的文本。如果将其保留为空白，则将使用URL。

	第三个参数可以包含您想要添加到链接的属性列表。这些属性可以是简单字符串或关联数组。

	这里有些例子::

		echo anchor('news/local/123', 'My News', 'title="News title"');
		// 打印: <a href="http://example.com/index.php/news/local/123" title="News title">My News</a>

		echo anchor('news/local/123', 'My News', ['title' => 'The best news!']);
		// 打印: <a href="http://example.com/index.php/news/local/123" title="The best news!">My News</a>

		echo anchor('', 'Click here');
		// 打印: <a href="http://example.com/index.php">Click here</a>

	如上所述，您可以指定替代配置。如果为不同于您的站点生成的链接（包含不同的配置首选项），则可能会发现备用配置很有用。我们使用它来对框架本身进行单元测试。

	.. note:: 传递到 ``anchor`` 函数中的属性会自动转义，以免受XSS攻击。

.. php:function:: anchor_popup([$uri = ''[, $title = ''[, $attributes = FALSE[, $altConfig = NULL]]]])

	:param	string	$uri: URI 字符串
	:param	string	$title: 锚标题
	:param	mixed	$attributes: HTML属性
	:param	\Config\App	$altConfig: 要使用的备用配置
	:returns:	Pop-up 超链接
	:rtype:	string

	与 :php:func:`anchor()` 函数几乎相同，除了它在新窗口中打开URL。您可以在第三个参数中指定JavaScript窗口属性，以控制窗口的打开方式。如果未设置第三个参数，它将仅使用您自己的浏览器设置打开一个新窗口。

	这是带有属性的示例::

		$atts = [
			'width'       => 800,
			'height'      => 600,
			'scrollbars'  => 'yes',
			'status'      => 'yes',
			'resizable'   => 'yes',
			'screenx'     => 0,
			'screeny'     => 0,
			'window_name' => '_blank'
		];

		echo anchor_popup('news/local/123', 'Click Me!', $atts);

	.. note:: 上面的属性是函数的默认值，因此您只需要设置与所需属性不同的属性即可。如果要让函数使用其所有默认值，只需在第三个参数中传递一个空数组即可:
                    ``echo anchor_popup('news/local/123', 'Click Me!', [])``;

	.. note:: **window_name** 不是真实的属性，但是对于 JavaScript `window.open() <https://www.w3schools.com/jsref/met_win_open.asp>`_ 方法， 它接受任何一方的窗口名或者窗口目标。
 
	.. note:: 除上面列出的属性外，任何其他属性都将被解析为锚标记的HTML属性。

		如上所述，您可以指定替代配置。如果为不同于您的站点生成的链接（包含不同的配置首选项），则可能会发现备用配置很有用。我们使用它来对框架本身进行单元测试。

	.. note:: 传递给 ``anchor_popup`` 函数的属性会自动转义，以免受XSS攻击。

.. php:function:: mailto($email[, $title = ''[, $attributes = '']])

	:param	string	$email: E-mail 地址
	:param	string	$title: 锚标题
	:param	mixed	$attributes: HTML属性
	:returns:	"mail to" 超链接
	:rtype:	string

	创建一个标准的HTML电子邮件链接。用法示例::

		echo mailto('me@my-site.com', 'Click Here to Contact Me');

	与 :php:func:`anchor()` 上面的选项卡一样，您可以使用第三个参数设置属性:: 

		$attributes = ['title' => 'Mail me'];
		echo mailto('me@my-site.com', 'Contact Me', $attributes);

	.. note:: 传递到mailto函数的属性会自动转义以防止受到XSS攻击。

.. php:function:: safe_mailto($email[, $title = ''[, $attributes = '']])

	:param	string	$email: E-mail 地址
	:param	string	$title: 锚标题
	:param	mixed	$attributes: HTML属性
	:returns:	垃圾邮件安全的 "mail to" 超链接
	:rtype:	string

	该功能与 :php:func:`mailto()` 功能相同，不同之处在于它使用JavaScript编写的序号来编写 *mailto* 标签的混淆版本，以帮助防止垃圾邮件机器人获取电子邮件地址。

.. php:function:: auto_link($str[, $type = 'both'[, $popup = FALSE]])

	:param	string	$str: 输入字符串
	:param	string	$type: 链接类型 ('email', 'url' 或 'both')
	:param	bool	$popup: 是否创建弹出链接
	:returns:	链接字符串
	:rtype:	string

	自动将字符串中包含的URL和email转换为链接。示例::

		$string = auto_link($string);

	第二个参数确定URL和email是被转换还是仅被转换。如果未指定该参数，则默认行为是两个。电子邮件链接的编码与 :php:func:`safe_mailto()` 如上所述。

	仅转换URL::

		$string = auto_link($string, 'url');

	仅转换电子email地址::

		$string = auto_link($string, 'email');

	第三个参数确定链接是否在新窗口中显示。该值可以为TRUE或FALSE（布尔值）::

		$string = auto_link($string, 'both', TRUE);

	.. note:: 唯一可识别的URL是以“www.”开头的URL或带有“://”。	

.. php:function:: url_title($str[, $separator = '-'[, $lowercase = FALSE]])

	:param	string	$str: 输入字符串
	:param	string	$separator: 单词分隔符
	:param	bool	$lowercase: 是否将输出字符串转换为 `小写`
	:returns:	格式化的URL字符串
	:rtype:	string

	将字符串作为输入，并创建易于使用的URL字符串。例如，如果您有一个博客，您想在其中使用URL中条目的标题，则此功能很有用。 示例::

		$title     = "What's wrong with CSS?";
		$url_title = url_title($title);
		// 产生： Whats-wrong-with-CSS

	第二个参数确定单词定界符。默认情况下，使用破折号。首选的选项有：- **-** (破折号) or **_** (下划线)。

	示例::

		$title     = "What's wrong with CSS?";
		$url_title = url_title($title, 'underscore');
		// 产生： Whats_wrong_with_CSS

	第三个参数确定是否强制使用小写字符。默认情况下不是。选项为布尔值TRUE/FALSE。

	示例::

		$title     = "What's wrong with CSS?";
		$url_title = url_title($title, 'underscore', TRUE);
		// 产生： whats_wrong_with_css

.. php:function:: prep_url($str = '')

	:param	string	$str: URL字符串
	:returns:	带协议前缀的URL字符串
	:rtype:	string

	如果URL中缺少协议前缀，此函数将添加 *http://*。

	将URL字符串传递给以下函数::

		$url = prep_url('example.com');
