##################
控制器过滤
##################

.. contents::
    :local:
    :depth: 2

控制器过滤器使您可以在控制器执行之前或之后执行操作。与 :doc:`事件 </extending/events>` 不同，您可以非常简单地选择应用程序中已对其应用过滤器的URI。传入的过滤器可以修改请求，然后过滤器甚至可以修改响应，从而具有很大的灵活性和功能性。可以使用过滤器执行的一些常见任务示例是:

* 对传入请求执行CSRF保护
* 根据其角色来限制网站的区域
* 在某些端点上执行速率限制
* 显示 "Down for Maintenance" 页面
* 自动执行内容协商
* 更多..

*****************
创建过滤器
*****************

过滤器是实现 ``CodeIgniter\Filters\FilterInterface`` 的简单类。它们包含两个方法：``before()`` 和 ``after()`` 分别保存将在控制器之前和之后运行的代码。您的类必须包含两个方法，但是如果不需要，可以将这些方法留空。框架过滤器类如下所示::

    <?php namespace App\Filters;

    use CodeIgniter\HTTP\RequestInterface;
    use CodeIgniter\HTTP\ResponseInterface;
    use CodeIgniter\Filters\FilterInterface;

    class MyFilter implements FilterInterface
    {
        public function before(RequestInterface $request)
        {
            // 在这里做点什么
        }

        //--------------------------------------------------------------------

        public function after(RequestInterface $request, ResponseInterface $response)
        {
            // 在这里做点什么
        }
    }

前置过滤器
==============

从任何过滤器中，您都可以返回 ``$request`` 对象，它将替换当前的Request，使您可以进行更改，这些更改在控制器执行时仍然存在。


由于前置过滤器是在执行控制器之前执行，您有时可能希望停止控制器正在发生的行为。您可以通过传回不是请求对象的任何东西来做到这一点。通常用于执行重定向，如以下示例所示::

    public function before(RequestInterface $request)
    {
        $auth = service('auth');

        if (! $auth->isLoggedIn())
        {
            return redirect('login');
        }
    }

如果返回Response实例，则Response将被发送回客户端，并且脚本执行将停止。这对于实现API的速率限制很有用。有关示例，请参见 :doc:`Throttler </libraries/throttler>`。

后置过滤器
=============

后置过滤器与前置过滤器几乎相同，除了只能返回 ``$response`` 对象，并且不能停止脚本执行。这确实允许您修改最终输出，或仅对最终输出执行某些操作。这可以用于确保以正确的方式设置了某些安全标头，或者用于缓存最终输出，甚至可以使用错误单词过滤器过滤最终输出。

*******************
配置过滤器
*******************

创建过滤器后，您需要配置它们的运行时间。这是在 ``app/Config/Filters.php`` 中完成的。该文件包含四个属性，可让您准确配置过滤器的运行时间。

$aliases
========

``$aliases`` 数组用于将一个简单名称与一个或多个要运行的过滤器的完全限定的类名相关联::

    public $aliases = [
        'csrf' => \CodeIgniter\Filters\CSRF::class
    ];

别名是强制性的，如果以后尝试使用完整的类名，则系统将引发错误。以这种方式定义它们可以很容易地切换出所使用的类。非常适合当您决定需要更改为其他身份验证系统时，仅更改过滤器的类就完成了。

您可以将多个过滤器组合为一个别名，从而使复杂的过滤器集易于应用::

    public $aliases = [
        'apiPrep' => [
            \App\Filters\Negotiate::class,
            \App\Filters\ApiAuth::class
        ]
    ];

您应该根据需要定义任意多个别名。

$globals
========

第二部分允许您定义应用于框架的每个请求的所有过滤器。您应该注意在这里使用的数量，因为它可能会影响性能，以至于每个请求都运行太多。可以通过将过滤器的别名添加到before或after数组来指定过滤器::

	public $globals = [
		'before' => [
			'csrf'
		],
		'after'  => []
	];

有时候，您希望对几乎每个请求都应用过滤器，但是有一些应该单独使用。一个常见的示例是，如果您需要从CSRF保护过滤器中排除一些URI，以允许来自第三方网站的请求命中一个或两个特定的URI，而其余的则受到保护。为此，请添加一个带有'except'键和一个uri的数组，以匹配别名旁边的值::

	public $globals = [
		'before' => [
			'csrf' => ['except' => 'api/*']
		],
		'after'  => []
	];

在过滤器设置中可以随意使用URI，可以使用正则表达式，或者像本示例一样，对通配符使用星号，以匹配之后的所有字符。在此示例中，以URL开头的任何URL ``api/`` 都将不受CSRF保护，但该站点的表单将全部受到保护。如果需要指定多个URI，则可以使用URI模式数组::

	public $globals = [
		'before' => [
			'csrf' => ['except' => ['foo/*', 'bar/*']]
		],
		'after'  => []
	];

$methods
========

您可以将过滤器应用于某个HTTP方法的所有请求，例如POST，GET，PUT等。在此数组中，您可以使用小写形式指定方法名称。它的值将是要运行的过滤器数组。与 ``$globals`` 或 ``$filters`` 属性不同，它们只会像以前的过滤器一样运行::

    public $methods = [
        'post' => ['foo', 'bar'],
        'get'  => ['baz']
    ]

除了标准的HTTP方法外，它还支持两种特殊情况：“cli”和“ajax”。名称在这里是不言自明的，但是'cli'将适用于从命令行运行的所有请求，而'ajax'将适用于每个AJAX请求。

.. note:: AJAX请求取决于 ``X-Requested-With`` 标头，在某些情况下，默认情况下，``X-Requested-With`` 标头不会通过JavaScript在XHR请求中发送（即，fetch）。有关如何避免此问题的信息，请参见 :doc:`AJAX 请求 </general/ajax>` 部分。

$filters
========

此属性是过滤器别名的数组。对于每个别名，您可以在包含过滤器应应用于的URI模式列表的数组之前和之后指定::

    public filters = [
        'foo' => ['before' => ['admin/*'], 'after' => ['users/*']],
        'bar' => ['before' => ['api/*', 'admin/*']]
    ];

****************
提供的过滤器
****************

CodeIgniter4捆绑了三个过滤器：Honeypot，Security和DebugToolbar。
