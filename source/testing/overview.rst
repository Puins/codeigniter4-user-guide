#######
测试
#######

CodeIgniter的构建旨在使对框架和应用程序的测试尽可能简单。``phpunit`` 内置对它的支持，并且该框架提供了许多方便的辅助方法，以使您的应用程序的各个方面的测试尽可能轻松。

.. contents::
    :local:
    :depth: 2

*************
系统设置
*************

安装 phpUnit
==================

CodeIgniter使用 `phpUnit <https://phpunit.de/>`__ 作为所有测试的基础。有两种方法可以安装phpUnit以便在系统中使用。

Composer
----------

推荐的方法是使用 `Composer <https://getcomposer.org/>`_ 将其安装在项目中。尽管可以在全局范围内安装它，但我们不建议您这样做，因为随着时间的流逝，它可能导致与系统上其他项目的兼容性问题。

确保系统上已安装Composer。在项目根目录（包含应用程序和系统目录的目录）中，从命令行键入以下内容::

    > composer require --dev phpunit/phpunit

这将为您当前的PHP版本安装正确的版本。完成后，您可以通过键入以下内容运行该项目的所有测试::

    > ./vendor/bin/phpunit

Phar
-------

另一个选择是从 `phpUnit <https://phpunit.de/getting-started/phpunit-7.html>`_ 网站下载 `.phar` 文件。这是一个独立文件，应放置在项目根目录下。


************************
测试应用程序
************************

PHPUnit 配置
=====================

框架在项目根目录中有一个 ``phpunit.xml.dist`` 文件。这控制了框架本身的单元测试。如果您提供自己的 ``phpunit.xml``，它将超越此。

如果你是单元测试您的应用程序，``phpunit.xml`` 应该排除在 ``system`` 文件夹，以及任何 ``vendor`` 或 ``ThirdParty`` 文件夹外。

测试类
==============

为了利用所提供的其他工具，您的测试必须继承 ``CIUnitTestCase``。默认情况下，所有测试均应位于 **tests/app** 目录中。

要测试新库 **Foo**，您可以在 **tests/app/Libraries/FooTest.php** 中创建一个新文件::

    <?php namespace App\Libraries;

    use CodeIgniter\Test\CIUnitTestCase;

    class FooTest extends CIUnitTestCase
    {
        public function testFooNotBar()
        {
            . . .
        }
    }

要测试您的模型之一，您可能最终会在 ``tests/app/Models/OneOfMyModelsTest.php`` 中看到以下内容::

    <?php namespace App\Models;

    use CodeIgniter\Test\CIUnitTestCase;

    class OneOfMyModelsTest extends CIUnitTestCase
    {
        public function testFooNotBar()
        {
            . . .
        }
    }


您可以创建适合您的测试样式/需求的任何目录结构。为测试类命名时，请记住 **app** 目录是 ``App`` 命名空间的根，因此您使用的任何类都必须具有相对于的正确命名空间 ``App``。

.. note:: 测试类并非严格要求使用命名空间，但是它们有助于确保没有类名冲突。

测试数据库结果时，必须使用 `CIDatabaseTestClass <database.html>`_ 类。

附加断言
---------------------

``CIUnitTestCase`` 提供了可能有用的其他单元测试断言。

**assertLogged($level, $expectedMessage)**

确保您希望实际记录的内容是::

        $config = new LoggerConfig();
        $logger = new Logger($config);

        ... 做一些您希望从中获得日志条目的操作
        $logger->log('error', "That's no moon");

        $this->assertLogged('error', "That's no moon");

**assertEventTriggered($eventName)**

确保您希望触发的事件实际上是::

    Events::on('foo', function($arg) use(&$result) {
        $result = $arg;
    });

    Events::trigger('foo', 'bar');

    $this->assertEventTriggered('foo');

**assertHeaderEmitted($header, $ignoreCase=false)**

确保实际上发出了标头或cookie::

    $response->setCookie('foo', 'bar');

    ob_start();
    $this->response->send();
    $output = ob_get_clean(); // 如果你想检查副体

    $this->assertHeaderEmitted("Set-Cookie: foo=bar");

Note: 与此相关的测试用例应在 `PHPunit中作为单独的进程运行 <https://phpunit.readthedocs.io/en/7.4/annotations.html#runinseparateprocess>`_ 。

**assertHeaderNotEmitted($header, $ignoreCase=false)**

确保没有发出标题或cookie::

    $response->setCookie('foo', 'bar');

    ob_start();
    $this->response->send();
    $output = ob_get_clean(); // 如果你想检查副体

    $this->assertHeaderNotEmitted("Set-Cookie: banana");

Note: 与此相关的测试用例应在 `PHPunit中作为单独的进程运行 <https://phpunit.readthedocs.io/en/7.4/annotations.html#runinseparateprocess>`_ 。

**assertCloseEnough($expected, $actual, $message='', $tolerance=1)**

对于延长的执行时间测试，请测试预期时间与实际时间之间的绝对差是否在规定的公差内::

    $timer = new Timer();
    $timer->start('longjohn', strtotime('-11 minutes'));
    $this->assertCloseEnough(11 * 60, $timer->getElapsedTime('longjohn'));

上述测试将允许实际时间为660或661秒。

**assertCloseEnoughString($expected, $actual, $message='', $tolerance=1)**

对于延长的执行时间测试，测试预期时间与实际时间之间的绝对差（格式为字符串）在规定的公差范围内::

    $timer = new Timer();
    $timer->start('longjohn', strtotime('-11 minutes'));
    $this->assertCloseEnoughString(11 * 60, $timer->getElapsedTime('longjohn'));

上述测试将允许实际时间为660或661秒。


访问受保护的/私有的属性
--------------------------------------

测试时，可以使用以下setter和getter方法访问要测试的类中的保护方法和私有方法以及属性。

**getPrivateMethodInvoker($instance, $method)**

使您可以从类外部调用私有方法。这将返回一个可以调用的函数。第一个参数是要测试的类的实例。第二个参数是您要调用的方法的名称。

::

    // 创建测试类的是实例
    $obj = new Foo();

    // 获取'privateMethod'方法的调用者。
	$method = $this->getPrivateMethodInvoker($obj, 'privateMethod');

    // 测试结果
	$this->assertEquals('bar', $method('param1', 'param2'));

**getPrivateProperty($instance, $property)**

从类的实例中检索私有/受保护的类属性的值。第一个参数是要测试的类的实例。第二个参数是属性的名称。

::

    // 创建测试类的是实例
    $obj = new Foo();

    // 测试值
    $this->assertEquals('bar', $this->getPrivateProperty($obj, 'baz'));

**setPrivateProperty($instance, $property, $value)**

在类实例内设置一个受保护的值。第一个参数是要测试的类的实例。第二个参数是要为其设置值的属性的名称。第三个参数是将其设置为的值::

    // 创建测试类的是实例
    $obj = new Foo();

    // 设置值
    $this->setPrivateProperty($obj, 'baz', 'oops!');

    // Do normal testing...

模拟服务
================

您通常会发现您需要模拟 **app/Config/Services.php** 中定义的服务之一，以将测试限制为仅涉及代码，同时模拟服务的各种响应。在测试控制器和其他集成测试时尤其如此。该服务类提供了两个方法，使这个简单的： ``injectMock()``, 和 ``reset()``。

**injectMock()**

此方法允许您定义Services类将返回的确切实例。您可以使用它来设置服务的属性，使其以某种方式运行，或者将服务替换为模拟类。
::

    public function testSomething()
    {
        $curlrequest = $this->getMockBuilder('CodeIgniter\HTTP\CURLRequest')
                            ->setMethods(['request'])
                            ->getMock();
        Services::injectMock('curlrequest', $curlrequest);

        // Do normal testing here....
    }

第一个参数是您要替换的服务。该名称必须与Services类中的函数名称完全匹配。第二个参数是要替换为的实例。

**reset()**

从Services类中删除所有模拟的类，使其恢复到原始状态。


流过滤器
==============

**CITestStreamFilter** 提供了这些辅助方法的替代方法。

您可能需要测试难以测试的内容。有时，捕获流（例如PHP自己的STDOUT或STDERR）可能会有所帮助。 ``CITestStreamFilter`` 将帮助您从您所选择的流捕获输出。

在一个测试用例中演示此示例::

    public function setUp()
    {
        CITestStreamFilter::$buffer = '';
        $this->stream_filter = stream_filter_append(STDOUT, 'CITestStreamFilter');
    }

    public function tearDown()
    {
        stream_filter_remove($this->stream_filter);
    }

    public function testSomeOutput()
    {
        CLI::write('first.');
        $expected = "first.\n";
        $this->assertEquals($expected, CITestStreamFilter::$buffer);
    }
