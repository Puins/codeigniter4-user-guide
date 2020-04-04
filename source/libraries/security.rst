##############
Security 类
##############

Security 类包含的方法可帮助保护您的站点免受跨站点请求伪造攻击。

.. contents::
    :local:
    :depth: 2

*******************
加载类库
*******************

如果你加载这个类，只是想进行 CSRF 的防护，那就没必要加载它，因为它作为过滤器运行并且没有手动交互。

如果发现确实需要直接访问的情况，则可以通过 ``Services`` 文件加载它::

	$security = \Config\Services::security();

*********************************
跨站点请求伪造 (CSRF)
*********************************

您可以通过更改 **app/Config/Filters.php** 并全局启用csrf过滤器来启用CSRF保护::

	public $globals = [
		'before' => [
			//'honeypot'
			'csrf'
		]
	];

可以从CSRF保护中将选择的URI列入白名单（例如，期望外部发布内容的API端点）。您可以通过将它们作为异常添加到过滤器中来添加这些URI::

	public $globals = [
		'before' => [
			'csrf' => ['except' => ['api/record/save']]
		]
	];

还支持正则表达式（不区分大小写）::

    public $globals = [
		'before' => [
			'csrf' => ['except' => ['api/record/[0-9]+']]
		]
	];

如果您使用 :doc:`form 辅助函数 <../helpers/form_helper>`，然后 :func:`form_open()` 会在表单中自动插入一个隐藏的csrf字段。如果没有，那么您可以使用始终可用 ``csrf_token()`` and ``csrf_hash()`` 函数
::

	<input type="hidden" name="<?= csrf_token() ?>" value="<?= csrf_hash() ?>" />

此外，您可以使用 ``csrf_field()`` 方法为您生成此隐藏的输入字段::

	// 生成: <input type="hidden" name="{csrf_token}" value="{csrf_hash}" />
	<?= csrf_field() ?>

发送JSON请求时，CSRF令牌也可以作为参数之一传递。传递CSRF令牌的另一种方法是特殊的Http标头，其名称可通过 ``csrf_header()`` 函数使用。

此外，您可以使用 ``csrf_meta()`` 方法为您生成此方便的元标记::

	// 生成: <meta name="{csrf_header}" content="{csrf_hash}" />
	<?= csrf_meta() ?>

检查CSRF令牌是否可用的顺序如下:

1. ``$_POST`` 数组
2. Http 标头
3. ``php://input`` (JSON 请求) - 请记住，这种方法是最慢的一种，因为我们必须先解码JSON然后再对其进行编码

令牌可以在每次提交时重新生成（默认），也可以在CSRF cookie的整个生命周期中保持不变。令牌的默认再生提供了更严格的安全性，但是可能会导致可用性问题，因为其他令牌会变得无效（后退/前进导航，多个选项卡/窗口，异步操作等）。您可以通过编辑以下config参数来更改此行为
::

	public $CSRFRegenerate  = true;

当请求未通过CSRF验证检查时，默认情况下它将重定向到上一页，并设置一条可以显示给最终用户的即显 ``error`` 消息。这提供了比简单崩溃更好的体验。可以通过编辑 **app/Config/App.php** 中的 ``$CSRFRedirect`` 值来关闭::

	public $CSRFRedirect = false;

即使重定向值是 **true**，AJAX调用也不会重定向，但会引发错误。

*********************
其他有用的方法
*********************

您将不需要直接使用Security类中的大多数方法。以下是您可能会发现有用的与CSRF保护无关的方法。

**sanitizeFilename()**

尝试清理文件名，以防止尝试遍历目录和其他安全威胁，这对于通过用户输入提供的文件特别有用。第一个参数是清理路径。

如果允许用户输入包含相对路径，例如 ``file/in/some/approved/folder.txt``，则可以将第二个可选参数 ``$relative_path`` 设置为true。
::

	$path = $security->sanitizeFilename($request->getVar('filepath'));
