##################
API Response Trait
##################

许多现代PHP开发都需要构建API，无论是简单地为重载javascript的单页应用程序提供数据还是作为独立产品提供。CodeIgniter提供了一个API Response Trait，可以与任何控制器一起使用，以简化公共响应类型，而无需记住应该为哪种响应类型返回哪个HTTP状态码。

.. contents::
    :local:
    :depth: 2

*************
用法示例
*************

以下示例显示了控制器内的一种常见用法模式。

::

    <?php namespace App\Controllers;

    use CodeIgniter\API\ResponseTrait;

    class Users extends \CodeIgniter\Controller
    {
        use ResponseTrait;

        public function createUser()
        {
            $model = new UserModel();
            $user  = $model->save($this->request->getPost());

            // 响应201状态码
            return $this->respondCreated();
        }
    }

在此示例中，返回HTTP状态码201，以及通用状态消息“已创建”。存在用于最常见用例的方法::

    // 通用响应方法
    respond($data, 200);
    // 一般故障响应
    fail($errors, 400);
    // 项目创建响应
    respondCreated($data);
    // 项目成功删除
    respondDeleted($data);
    // 命令执行，无需响应
    respondNoContent($message);
    // 客户端无权
    failUnauthorized($description);
    // 禁止行动
    failForbidden($description);
    // 找不到资源
    failNotFound($description);
    // 数据未验证
    failValidationError($description);
    // 资源已经存在
    failResourceExists($description);
    // 资源先前已删除
    failResourceGone($description);
    // 客户端请求太多
    failTooManyRequests($description);

***********************
处理响应类型
***********************

当您通过以下任何一种方法传递数据时，它们将根据以下条件确定数据类型以格式化结果:

* 如果$data是字符串, 它将被当作HTML发送回客户端。
* 如果$data是数组, 它将尝试与客户端请求的内容协商内容类型，默认为JSON。如果尚未在 ``Config\API.php`` 中指定内容，则使用 ``$supportedResponseFormats`` 属性。

要定义使用的格式化程序，请编辑 **Config/Format.php**。``$supportedResponseFormats`` 包含MIME类型的列表，您的应用程序可以自动格式化响应。默认情况下，系统知道如何格式化XML和JSON响应::

        public $supportedResponseFormats = [
            'application/json',
            'application/xml'
        ];

这是在 :doc:`Content Negotiation </incoming/content_negotiation>` 期间用于确定要返回的响应类型的数组。如果找不到客户端请求的内容和您支持的内容之间的匹配项，则此数组中的第一种格式将是返回的格式。

接下来，您需要定义用于格式化数据数组的类。这必须是完全限定的类名，并且该类必须实现 **CodeIgniter\\Format\\FormatterInterface**。开箱即用的格式化程序支持JSON和XML::

    public $formatters = [
        'application/json' => \CodeIgniter\Format\JSONFormatter::class,
        'application/xml'  => \CodeIgniter\Format\XMLFormatter::class
    ];

因此，如果您的请求在 **Accept** 标头中要求JSON格式的数据，则您通过 ``respond*`` 或 ``fail*`` 方法传递的数据数组 将由 **CodeIgniter\\API\\JSONFormatter** 类格式化。生成的JSON数据将发送回客户端。

类参考
***************

.. php:method:: respond($data[, $statusCode=200[, $message='']])

    :param mixed  $data: 要返回给客户端的数据。字符串或数组。
    :param int    $statusCode: 要返回的HTTP状态码。默认为200
    :param string $message: 要返回的自定义“原因”消息。

    此 ``trait`` 中的所有其他方法使用的方法将响应返回给客户端。

    ``$data`` 元素可以是字符串或数组。默认情况下，字符串将以HTML形式返回，而数组将通过json_encode运行并以JSON形式返回，除非 :doc:`内容协商 </incoming/content_negotiation>` 确定应以其他格式返回。

    如果传递了 ``$message`` 字符串，它将代替响应状态的标准IANA原因码使用。但是，并非每个客户端都会尊重自定义代码，并且会使用与状态码匹配的IANA标准。

    .. note:: 因为它在活动的Response实例上设置状态码和主体，所以它应该始终是脚本执行中的最终方法。

.. php:method:: fail($messages[, int $status=400[, string $code=null[, string $message='']]])

    :param mixed $messages: A string or array of strings that contain error messages encountered.
    :param int   $status: 要返回的HTTP状态码。默认为400。
    :param string $code: 一个自定义的，特定于API的错误代码。
    :param string $message: 要返回的自定义“原因”消息。
    :returns: 客户端首选格式的多部分响应。

    这是通用方法，用于表示失败的响应，并由所有其他"fail"方法使用。

    ``$messages`` 元素可以是字符串或字符串的数组。

    ``$status`` 参数是应返回的HTTP状态码。

    由于使用自定义错误代码可以更好地服务许多API，因此可以在第三个参数中传递自定义错误代码。如果没有值，则与 ``$status`` 相同。

    如果传递了 ``$message`` 字符串，它将代替响应状态的标准IANA原因码使用。但是，并非每个客户端都会尊重自定义代码，并且会使用与状态码匹配的IANA标准。

    响应是一个包含两个元素的数组： ``error`` 和 ``messages``。``error`` 元素包含错误的状态码。``messages`` 元素包含错误消息的数组。它看起来像::

	    $response = [
	        'status'   => 400,
	        'code'     => '321a',
	        'messages' => [
	            'Error message 1',
	            'Error message 2'
	        ]
	    ];

.. php:method:: respondCreated($data = null[, string $message = ''])

    :param mixed  $data: 要返回给客户端的数据。字符串或数组。
    :param string $message: 要返回的自定义“原因”消息。
    :returns: 响应对象的send()方法的值。

    设置创建新资源时要使用的适当状态码，通常为201 。::

	    $user = $userModel->insert($data);
	    return $this->respondCreated($user);

.. php:method:: respondDeleted($data = null[, string $message = ''])

    :param mixed  $data: 要返回给客户端的数据。字符串或数组。
    :param string $message: 要返回的自定义“原因”消息。
    :returns: 响应对象的send()方法的值。

    设置当此API调用删除新资源时要使用的适当状态码，通常为200。

    ::

	    $user = $userModel->delete($id);
	    return $this->respondDeleted(['id' => $id]);

.. php:method:: respondNoContent(string $message = 'No Content')

    :param string $message: 要返回的自定义“原因”消息。
    :returns: 响应对象的send()方法的值。

    设置适当的状态码，以在服务器成功执行命令但没有有意义的答复发送回客户端时使用，通常为204。

    ::

	    sleep(1);
	    return $this->respondNoContent();        

.. php:method:: failUnauthorized(string $description = 'Unauthorized'[, string $code=null[, string $message = '']])

    :param string  $description: 向用户显示的错误消息。
    :param string $code: 一个自定义的，特定于API的错误代码。
    :param string $message: 要返回的自定义“原因”消息。
    :returns: 响应对象的send()方法的值。

    设置适当的状态码，以在用户未被授权或授权不正确时使用。状态码是401。

    ::

	    return $this->failUnauthorized('Invalid Auth token');

.. php:method:: failForbidden(string $description = 'Forbidden'[, string $code=null[, string $message = '']])

    :param string  $description: 向用户显示的错误消息。
    :param string $code: 一个自定义的，特定于API的错误代码。
    :param string $message: 要返回的自定义“原因”消息。
    :returns: 响应对象的send()方法的值。

    不同于 ``failUnauthorized``，当从不允许使用请求的API端点时，应使用此方法。未经授权意味着鼓励客户端使用不同的凭据重试。禁止表示客户端不应再试一次，因为它无济于事。状态码是403。

    ::

    	return $this->failForbidden('Invalid API endpoint.');

.. php:method:: failNotFound(string $description = 'Not Found'[, string $code=null[, string $message = '']])

    :param string  $description: 向用户显示的错误消息。
    :param string $code: 一个自定义的，特定于API的错误代码。
    :param string $message: 要返回的自定义“原因”消息。
    :returns: 响应对象的send()方法的值。

    设置找不到请求的资源时要使用的适当状态码。状态码是404。

    ::

    	return $this->failNotFound('User 13 cannot be found.');

.. php:method:: failValidationError(string $description = 'Bad Request'[, string $code=null[, string $message = '']])

    :param string  $description: 向用户显示的错误消息。
    :param string $code: 一个自定义的，特定于API的错误代码。
    :param string $message: 要返回的自定义“原因”消息。
    :returns: 响应对象的send()方法的值。

    设置客户端发送的数据未通过验证规则时使用的适当状态码。状态码通常为400。

    ::

    	return $this->failValidationError($validation->getErrors());

.. php:method:: failResourceExists(string $description = 'Conflict'[, string $code=null[, string $message = '']])

    :param string  $description: 向用户显示的错误消息。
    :param string $code: 一个自定义的，特定于API的错误代码。
    :param string $message: 要返回的自定义“原因”消息。
    :returns: 响应对象的send()方法的值。

    设置客户端尝试创建的资源已存在时要使用的适当状态码。状态码通常是409。

    ::

    	return $this->failResourceExists('A user already exists with that email.');

.. php:method:: failResourceGone(string $description = 'Gone'[, string $code=null[, string $message = '']])

    :param string  $description: 向用户显示的错误消息。
    :param string $code: 一个自定义的，特定于API的错误代码。
    :param string $message: 要返回的自定义“原因”消息。
    :returns: 响应对象的send()方法的值。

    设置适当的状态码，以在先前删除了所请求的资源并且不再可用时使用。状态码通常是410。

    ::

    	return $this->failResourceGone('That user has been previously deleted.');

.. php:method:: failTooManyRequests(string $description = 'Too Many Requests'[, string $code=null[, string $message = '']])

    :param string  $description: 向用户显示的错误消息。
    :param string $code: 一个自定义的，特定于API的错误代码。
    :param string $message: 要返回的自定义“原因”消息。
    :returns: 响应对象的send()方法的值。

    设置客户端调用API端点太多次时要使用的适当状态码。这可能是由于某种形式的节流或速率限制。状态码通常为400。

    ::

    	return $this->failTooManyRequests('You must wait 15 seconds before making another request.');

.. php:method:: failServerError(string $description = 'Internal Server Error'[, string $code = null[, string $message = '']])

    :param string $description: 向用户显示的错误消息。
    :param string $code: 一个自定义的，特定于API的错误代码。
    :param string $message: 要返回的自定义“原因”消息。
    :returns: 响应对象的send()方法的值。

    设置服务器错误时使用的适当状态码。

    ::

    	return $this->failServerError('Server error.');
