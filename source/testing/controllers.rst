###################
测试控制器
###################

有了两个新的辅助函数类和traits，可以方便地测试控制器。在测试控制器时，您可以在控制器内执行代码，而无需先执行整个应用程序引导过程。通常，使用 `功能测试工具 <feature.html>`_ 会更简单，但是可以在需要时使用此功能。

.. note:: 由于尚未引导整个框架，因此有时您无法以这种方式测试控制器。

辅助 Trait
================

您可以使用任何一个基础测试类，但是您在测试中确实需要使用 ``ControllerTester`` Trait::

    <?php namespace CodeIgniter;

    use CodeIgniter\Test\ControllerTester;

    class TestControllerA extends \CIDatabaseTestCase
    {
        use ControllerTester;
    }

包含 ``trait`` 后，就可以开始设置环境，包括请求和响应类，请求正文，URI等。您指定要与 ``controller()`` 方法一起使用的控制器，并传入控制器的标准类名。最后， ``execute()`` 使用要作为参数运行的方法名称来调用该方法::

    <?php namespace CodeIgniter;

    use CodeIgniter\Test\ControllerTester;

    class TestControllerA extends \CIDatabaseTestCase
    {
        use ControllerTester;

        public function testShowCategories()
        {
            $result = $this->withURI('http://example.com/categories')
			    ->controller(\App\Controllers\ForumController::class)
			    ->execute('showCategories');

            $this->assertTrue($result->isOK());
        }
    }

辅助方法
==============

**controller($class)**

指定要测试的控制器的类名。第一个参数必须是完全限定的类名称（即，包括命名空间）::

    $this->controller(\App\Controllers\ForumController::class);

**execute($method)**

在控制器内执行指定的方法。唯一的参数是要运行的方法的名称::

    $results = $this->controller(\App\Controllers\ForumController::class)
                     ->execute('showCategories');

这将返回一个新的辅助函数类，该类提供了许多用于检查响应本身的例程。有关详情，请参见下文。

**withConfig($config)**

允许您传入 **Config\App.php** 的修改版本以测试不同的设置::

    $config = new Config\App();
    $config->appTimezone = 'America/Chicago';

    $results = $this->withConfig($config)
                     ->controller(\App\Controllers\ForumController::class)
                     ->execute('showCategories');

如果您不提供，则将使用该应用程序的App配置文件。

**withRequest($request)**

允许您提供适合您的测试需求的 **IncomingRequest** 实例::

    $request = new CodeIgniter\HTTP\IncomingRequest(new Config\App(), new URI('http://example.com'));
    $request->setLocale($locale);

    $results = $this->withRequest($request)
                     ->controller(\App\Controllers\ForumController::class)
                     ->execute('showCategories');

如果不提供，则具有默认应用程序值的新 **IncomingRequest** 实例将传递到控制器中。

**withResponse($response)**

允许您提供一个 **Response** 实例::

    $response = new CodeIgniter\HTTP\Response(new Config\App());

    $results = $this->withResponse($response)
                     ->controller(\App\Controllers\ForumController::class)
                     ->execute('showCategories');

如果不提供，则具有默认应用程序值的新 **Response** 实例将传递到控制器中。

**withLogger($logger)**

允许您提供一个 **Logger** 实例::

    $logger = new CodeIgniter\Log\Handlers\FileHandler();

    $results = $this->withResponse($response)
                    -> withLogger($logger)
                     ->controller(\App\Controllers\ForumController::class)
                     ->execute('showCategories');

如果不提供，则具有默认配置值的新 **Logger** 实例将传递到控制器中。

**withURI($uri)**

允许您提供一个新的URI，该URI模拟运行该控制器时客户端正在访问的URL。如果您需要检查控制器中的URI段，这将很有帮助。唯一的参数是代表有效URI的字符串::

    $results = $this->withURI('http://example.com/forums/categories')
                     ->controller(\App\Controllers\ForumController::class)
                     ->execute('showCategories');

良好的做法是始终在测试过程中提供URI以避免意外。

**withBody($body)**

允许您为请求提供自定义主体。在测试需要将JSON值设置为主体的API控制器时，这可能会有所帮助。唯一的参数是代表请求正文的字符串::

    $body = json_encode(['foo' => 'bar']);

    $results = $this->withBody($body)
                     ->controller(\App\Controllers\ForumController::class)
                     ->execute('showCategories');

检查响应
=====================

执行控制器时，将返回一个新的 **ControllerResponse** 实例，该实例提供了许多有用的方法以及对生成的请求和响应的直接访问。

**isOK()**

这提供了简单的检查，以将响应视为“成功”响应。这主要检查HTTP状态代码是否在200或300范围内::

    $results = $this->withBody($body)
                     ->controller(\App\Controllers\ForumController::class)
                     ->execute('showCategories');

    if ($results->isOK())
    {
        . . .
    }

**isRedirect()**

检查最终响应是否为某种重定向::

    $results = $this->withBody($body)
                     ->controller(\App\Controllers\ForumController::class)
                     ->execute('showCategories');

    if ($results->isRedirect())
    {
        . . .
    }

**request()**

您可以访问使用此方法生成的Request对象::

    $results = $this->withBody($body)
                     ->controller(\App\Controllers\ForumController::class)
                     ->execute('showCategories');

    $request = $results->request();

**response()**

这使您可以访问所生成的响应对象（如果有）::

    $results = $this->withBody($body)
                     ->controller(\App\Controllers\ForumController::class)
                     ->execute('showCategories');

    $response = $results->response();

**getBody()**

您可以使用 **getBody()** 方法将访问发送给客户端的响应的主体。这可能是生成的HTML或JSON响应等::

    $results = $this->withBody($body)
                     ->controller(\App\Controllers\ForumController::class)
                     ->execute('showCategories');

    $body = $results->getBody();

响应辅助方法
-----------------------

您返回的响应包含许多帮助程序方法，用于检查响应中的HTML输出。这些对于在测试中的断言中使用非常有用。

**see()** 方法检查页面上的文本，看它本身是否存在，或者更具体地标记，如由类型，类，或id指定::

    // 检查 "Hello World" 在页面上
    $results->see('Hello World');
    // 检查 "Hello World" 在h1标记内
    $results->see('Hello World', 'h1');
    // 检查 "Hello World" 在class = "notice" 的元素内
    $results->see('Hello World', '.notice');
    // 检查 "Hello World" 在id = "title" 的元素内
    $results->see('Hellow World', '#title');

**dontSee()** 方法是完全相反的::

    // 检查 "Hello World" 不在页面上
    $results->dontSee('Hello World');
    // 检查 "Hellow World" 不在h1标记内
    $results->dontSee('Hello World', 'h1');

**seeElement()** and **dontSeeElement()** 非常类似于以前的方法，但不是查看元素的值。相反，他们只是检查页面上是否存在元素::

    // 检查 class = "notice" 的元素存在
    $results->seeElement('.notice');
    // 检查 id = "title" 的元素存在
    $results->seeElement('#title')
    // 确认 id = "title" 的元素不存在
    $results->dontSeeElement('#title');

您可以使用 **seeLink()** 确保链接显示在页面上，并带有指定的文本::

    // 检查存在带有'Upgrade Account'作为文本的链接
    $results->seeLink('Upgrade Account');
    // 检查存在带有'Upgrade Account'作为文本且 class = 'upsell' 的链接
    $results->seeLink('Upgrade Account', '.upsell');

**seeInField()** 对于任何输入标签方法检查具有名称和值存在::

    // 检查存在名为'user'且值为'John Snow'的输入
    $results->seeInField('user', 'John Snow');
    // 检查多维输入
    $results->seeInField('user[name]', 'John Snow');

最后，您可以检查复选框是否存在，并使用 **seeCheckboxIsChecked()** 方法进行检查::

    // 检查复选框是否存在 class = 'foo'
    $results->seeCheckboxIsChecked('.foo');
    // 检查复选框是否存在 id = 'bar'
    $results->seeCheckboxIsChecked('#bar');
