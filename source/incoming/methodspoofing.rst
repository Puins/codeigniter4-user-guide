====================
HTTP 方法伪造
====================

使用HTML表单时，只能使用HTTP动词(GET或POST)。在大多数情况下，这很好。但是，要支持基于REST-ful的路由，您需要支持其他更正确的动词，例如DELETE或PUT。由于浏览器不支持此功能，因此CodeIgniter为您提供了一种欺骗所使用方法的方法。这允许您发出POST请求，但告诉应用程序应将其视为其他请求类型。

为了伪造该方法，将一个名称为 ``_method`` 的隐藏的输入添加到表单中。它的值是您希望请求成为的HTTP动词::

    <form action="" method="post">
        <input type="hidden" name="_method" value="PUT" />

    </form>

就路由和IncomingRequest类而言，此形式将转换为PUT请求，并且是真正的PUT请求。

您使用的表单必须是POST请求。GET请求不能被伪造。

.. note:: 请确保检查Web服务器的配置，因为某些服务器不支持默认配置的所有HTTP动词，并且必须启用其他程序包才能正常工作。
