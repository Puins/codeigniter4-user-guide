##################
Throttler 类
##################

.. contents::
    :local:
    :depth: 2

Throttler类提供了一种非常简单的方法，可以将要执行的活动限制为在设定的时间段内进行一定次数的尝试。这最常用于对API进行速率限制，或限制用户针对表单进行的尝试次数，以帮助防止暴力攻击。该类本身可用于您需要根据设置的时间间隔内的操作进行限制的任何内容。

********
概述
********

Throttler实现了 `Token Bucket <https://en.wikipedia.org/wiki/Token_bucket>`_ 算法的简化版本。基本上，这会将您要执行的每个操作都视为一个存储桶。调用 ``check()`` 方法时，您会告诉它存储桶的大小，可以容纳多少令牌以及时间间隔。默认情况下，每个 ``check()`` 调用使用1个可用令牌。让我们通过一个例子来阐明这一点。

假设我们希望某动作每秒发生一次。对Throttler的第一次调用将如下所示。第一个参数是存储桶名称，第二个参数是存储桶持有的令牌数量，第三个参数是存储桶重新填充所需的时间::

    $throttler = \Config\Services::throttler();
    $throttler->check($name, 60, MINUTE);

在这里，我们暂时使用 :doc:`全局常量 </general/common_functions>` 之一，以使其更具可读性。也就是说，存储桶每分钟允许执行60次操作，或者每秒允许执行1次操作。

假设某个第三方脚本试图重复访问URL。最初，它能够在不到一秒钟的时间内使用所有60个令牌。但是，在那之后，Throttler每秒只允许执行一次操作，因此有可能减慢请求的速度，以至于攻击不再值得。

.. note:: 为了使Throttler类起作用，必须将Cache库设置为使用虚拟对象以外的处理程序。为了获得最佳性能，建议使用内存缓存，例如Redis或Memcached。

*************
限速
*************

Throttler类不做任何速率限制或单独请求节流，而是完成一项工作的关键。提供了一个 :doc:`Filter </incoming/filters>` 的示例，该过滤器以每个IP地址每秒一个请求的速度实现了非常简单的速率限制。在这里，我们将介绍它的工作原理，以及如何设置它并开始在应用程序中使用它。

代码
========

您可以在 **app/Filters/Throttle.php** 上创建自己的Throttler过滤器，大致如下:: 

    <?php namespace App\Filters;

    use CodeIgniter\Filters\FilterInterface;
    use CodeIgniter\HTTP\RequestInterface;
    use CodeIgniter\HTTP\ResponseInterface;
    use Config\Services;

    class Throttle implements FilterInterface
    {
            /**
             * 这是使用Throttler类实现应用程序速率限制的演示。
             *
             * @param RequestInterface|\CodeIgniter\HTTP\IncomingRequest $request
             *
             * @return mixed
             */
            public function before(RequestInterface $request)
            {
                    $throttler = Services::throttler();

                    // 将整个站点的IP地址限制为每秒不超过1个请求。
                    if ($throttler->check($request->getIPAddress(), 60, MINUTE) === false)
                    {
                            return Services::response()->setStatusCode(429);
                    }
            }

            //--------------------------------------------------------------------

            /**
             * 我们在这里没什么可做的。
             *
             * @param RequestInterface|\CodeIgniter\HTTP\IncomingRequest $request
             * @param ResponseInterface|\CodeIgniter\HTTP\Response       $response
             *
             * @return mixed
             */
            public function after(RequestInterface $request, ResponseInterface $response)
            {
            }
    }

运行时，此方法首先获取throttler的实例。接下来，它将IP地址用作存储桶名称，并进行设置以将其限制为每秒一个请求。如果throttler拒绝检查，返回false，然后我们返回状态代码设置为429-太多尝试的Response，并且脚本执行在命中​​控制器之前就结束了。本示例将基于对站点的所有请求（而不是每页）中的单个IP地址进行限制。

应用过滤器
===================

我们不一定需要限制网站上的每个页面。对于许多Web应用程序，这最有意义的是仅将其应用于POST请求，尽管API可能希望限制用户发出的每个请求。为了将此应用到传入请求，您需要编辑 **/app/Config/Filters.php** 并首先向过滤器添加别名::

	public $aliases = [
		...
		'throttle' => \App\Filters\Throttle::class
	];

接下来，我们将其分配给网站上的所有POST请求::

    public $methods = [
        'post' => ['throttle', 'CSRF']
    ];

这就是全部。现在，必须对网站上发出的所有POST请求进行速率限制。

***************
类参考
***************

.. php:method:: check(string $key, int $capacity, int $seconds[, int $cost = 1])

    :param string $key: 存储桶的名称
    :param int $capacity: 存储桶中持有的令牌数量
    :param int $seconds: 存储桶完全充满所需的秒数
    :param int $cost: 在此操作上花费的令牌数量
    :returns: 如果可以执行操作，则为TRUE，否则为FALSE
    :rtype: bool

    检查存储桶中是否还有任何令牌，或者是否在分配的时间限制内使用了太多令牌。在每次检查期间，如果成功，可用令牌数$cost将减少。

.. php:method:: getTokentime()

    :returns: 直到另一个令牌可用的秒数。
    :rtype: integer

    在 ``check()`` 运行并返回FALSE之后，可以使用此方法确定直到新令牌可用并可以再次尝试操作之前的时间。在这种情况下，最小强制等待时间为一秒。
