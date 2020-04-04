####################
HTTP 功能测试
####################

功能测试使您可以查看对应用程序的单次调用的结果。这可能会返回单个Web表单的结果，命中API端点等等。这很方便，因为它允许您测试单个请求的整个生命周期，确保路由有效，响应是正确的格式，分析结果等等。

.. contents::
    :local:
    :depth: 2

测试类
==============

功能测试要求您所有的测试类都继承自 ``CodeIgniter\Test\FeatureTestCase`` 类。由于这继承了 `CIDatabaseTestCase <database.html>`_ ，因此您必须始终确保在执行操作之前调用 ``parent::setUp()`` 和 ``parent::tearDown()``。
::

    <?php namespace App;

    use CodeIgniter\Test\FeatureTestCase;

    class TestFoo extends FeatureTestCase
    {
        public function setUp()
        {
            parent::setUp();
        }

        public function tearDown()
        {
            parent::tearDown();
        }
    }

请求页面
=================

本质上，FeatureTestCase仅允许您在应用程序上调用终结点并返回结果。为此，您可以使用 ``call()`` 方法。第一个参数是要使用的HTTP方法（最常见的是GET或POST）。第二个参数是您网站上要测试的路径。第三个参数接受一个数组，该数组用于填充您正在使用的HTTP动词的超全局变量。因此，**GET** 方法将填充 **$_GET** 变量，而 **post** 请求将填充 **$_POST** 数组。
::

    // 获取一个简单的页面
    $result = $this->call('get', site_url());

    // 提交表单
    $result = $this->call('post', site_url('contact'), [
        'name' => 'Fred Flintstone',
        'email' => 'flintyfred@example.com'
    ]);

存在用于每个HTTP动词的简写方法，以简化键入并使内容更清晰::

    $this->get($path, $params);
    $this->post($path, $params);
    $this->put($path, $params);
    $this->patch($path, $params);
    $this->delete($path, $params);
    $this->options($path, $params);

.. note:: $params数组并不是对每个HTTP动词都有意义，但为了保持一致性而包含了该数组。

设置不同的路由
------------------------

您可以通过将 ``$routes`` 数组传递给 ``withRoutes()`` 方法来使用路线的自定义集合。这将覆盖系统中的所有现有路由::

    $routes = [
       [ 'get', 'users', 'UserController::list' ]
     ];

    $result = $this->withRoutes($routes)
        ->get('users');

每个 ``$routes`` 都是一个3元素的数组，包含HTTP动词（或全部为“add”），要匹配的URI和路由目的地。


设置Session值
----------------------

您可以使用 ``withSession()`` 方法设置自定义会话值，以便在单个测试中使用。发出此请求时，需要键/值对的数组，这些键/值对应存在于$_SESSION变量中。这对于测试身份验证等非常方便。
::

    $values = [
        'logged_in' => 123
    ];

    $result = $this->withSession($values)
        ->get('admin');

跳过事件
----------------

事件很容易在您的应用程序中使用，但在测试过程中可能会出现问题。尤其是用于发送电子邮件的事件。您可以使用以下 ``skipEvents()`` 方法告诉系统跳过任何事件处理::

    $result = $this->skipEvents()
        ->post('users', $userInfo);


测试响应
====================

执行 ``call()`` 并获得结果后，可以在测试中使用许多新的断言。

.. note:: 响应对象可从公开获得 ``$result->response``。如果需要，可以使用该实例对它执行其他声明。

检查响应状态
------------------------

**isOK()**

根据响应是否被认为是 ``ok``，返回布尔值true/false。这主要由200或300的响应状态代码确定。
::

    if ($result->isOK())
    {
        ...
    }

**assertOK()**

该断言仅使用 **isOK()** 方法来测试响应。
::

    $this->assertOK();

**isRedirect()**

根据响应是否为重定向响应，返回布尔值true/false。
::

    if ($result->isRedirect())
    {
        ...
    }

**assertRedirect()**

断言该响应是RedirectResponse的一个实例。
::

    $this->assertRedirect();

**assertStatus(int $code)**

断言返回的HTTP状态代码与$code相匹配。
::

    $this->assertStatus(403);


Session 断言
------------------

**assertSessionHas(string $key, $value = null)**

断言结果session中存在值。如果传递了$value，还将断言该变量的值与指定的值匹配。
::

    $this->assertSessionHas('logged_in', 123);

**assertSessionMissing(string $key)**

断言结果session不包含指定的$key。
::

    $this->assertSessionMissin('logged_in');


Header 断言
-----------------

**assertHeader(string $key, $value = null)**

断言响应中存在一个名为 **$key** 的header 。如果 **$value** 不为空，则还将断言这些值匹配。
::

    $this->assertHeader('Content-Type', 'text/html');

**assertHeaderMissing(string $key)**

断言响应中不存在标头名称 **$key**。
::

    $this->assertHeader('Accepts');



Cookie 断言
-----------------

**assertCookie(string $key, $value = null, string $prefix = '')**

断言响应中存在一个名为 **$key** 的cookie 。如果 **$value** 不为空，则还将断言这些值匹配。您可以根据需要通过将cookie前缀作为第三个参数传入来设置它。
::

    $this->assertCookie('foo', 'bar');

**assertCookieMissing(string $key)**

断言响应中不存在名为 **$key** 的cookie 。
::

    $this->assertCookieMissing('ci_session');

**assertCookieExpired(string $key, string $prefix = '')**

断言存在一个名为 **$key** 的cookie ，但已过期。您可以根据需要通过将cookie前缀作为第二个参数传递来设置它。
::

    $this->assertCookieExpired('foo');


DOM 断言
--------------

您可以执行测试，以查看带有以下声明的响应的正文中是否存在特定的元素/文本/等。

**assertSee(string $search = null, string $element = null)**

断言文本/HTML是否在页面上，无论是其自身还是（更具体而言）是在标签中（由类型，类或ID指定）::

    // 检查 "Hello World" 在页面上
    $this->assertSee('Hello World');
    // 检查 "Hello World" 在h1标记内
    $this->assertSee('Hello World', 'h1');
    // 检查 "Hello World" 在class = "notice" 的元素内
    $this->assertSee('Hello World', '.notice');
    // 检查 "Hello World" 在id = "title" 的元素内
    $this->assertSee('Hellow World', '#title');


**assertDontSee(string $search = null, string $element = null)**

断言与 **assertSee()** 方法完全相反的方法::

    // 检查 "Hello World" 不在页面上
    $results->dontSee('Hello World');
    // 检查 "Hellow World" 不在h1标记内
    $results->dontSee('Hello World', 'h1');

**assertSeeElement(string $search)**

与 **assertSee()** 类似，但是这仅检查现有元素。它不检查特定的文本::

    // 检查 class = "notice" 的元素存在
    $results->seeElement('.notice');
    // 检查 id = "title" 的元素存在
    $results->seeElement('#title')

**assertDontSeeElement(string $search)**

与 **assertSee()** 类似，但是这仅检查缺少的现有元素。它不检查特定的文本::

    // 确认 id = "title" 的元素不存在
    $results->dontSeeElement('#title');

**assertSeeLink(string $text, string $details=null)**

断言找到一个匹配标记为 **$text** 的锚标记::

    // 检查存在带有'Upgrade Account'作为文本的链接
    $results->seeLink('Upgrade Account');
    // 检查存在带有'Upgrade Account'作为文本且 class = 'upsell' 的链接
    $results->seeLink('Upgrade Account', '.upsell');

**assertSeeInField(string $field, string $value=null)**

断言输入标签具有名称和值::

    // 检查是否存在名为'user'且值为'John Snow'的输入
    $results->seeInField('user', 'John Snow');
    // 检查多维输入
    $results->seeInField('user[name]', 'John Snow');



使用JSON
-----------------

响应通常包含JSON响应，尤其是在使用API​​方法时。以下方法可以帮助测试响应。

**getJSON()**

此方法将以JSON字符串的形式返回响应的主体::

    // 这是响应主体:
    ['foo' => 'bar']

    $json = $result->getJSON();

    // 这是 $json:
    {
        "foo": "bar"
    }

.. note:: 请注意，JSON字符串将漂亮地打印在结果中。

**assertJSONFragment(array $fragment)**

断言$fragment在JSON响应中找到。它不需要匹配整个JSON值。

::

    // 这是响应主体:
    [
        'config' => ['key-a', 'key-b']
    ]

    // 是 true
    $this->assertJSONFragment(['config' => ['key-a']);

.. note:: 这只是使用phpUnit自己的 `assertArraySubset() <https://phpunit.readthedocs.io/en/7.2/assertions.html#assertarraysubset>`_ 方法进行比较。

**assertJSONExact($test)**

与 **assertJSONFragment()** 相似，但是会检查整个JSON响应以确保完全匹配。


使用XML
----------------

**getXML()**

如果您的应用程序返回XML，则可以通过此方法检索它。

