##############
AJAX 请求
##############

``IncomingRequest::isAJAX()`` 方法使用 ``X-Requested-With`` 标题定义是XHR还是正常请求。但是，最新的JavaScript实现（即抓取）不再随请求一起发送此标头，因此使用的 ``IncomingRequest::isAJAX()`` 可靠性降低，因为没有此标头就无法定义请求是否为XHR。

为了解决这个问题，到目前为止，最有效的解决方案是手动定义请求标头，强制将信息发送到服务器，服务器随后将能够识别该请求为XHR。

以下就是如何迫使在Fetch API和其他JavaScript库中发送 ``X-Requested-With`` 请求头。

Fetch API
=========

    fetch(url, {
        method: "get",
        headers: {

          "Content-Type": "application/json",

          "X-Requested-With": "XMLHttpRequest"

        }

    });


jQuery
======

例如，对于像jQuery这样的库，不必显式发送此标头，因为根据 `官方文档 <https://api.jquery.com/jquery.ajax/>`_ ，对于所有 ``$.ajax()`` 请求来说，这都是一个标准头。但是，如果你还是不想担风险并强制发送这个头，请按照以下步骤操作:

    $.ajax({
        url: "your url",

        headers: {'X-Requested-With': 'XMLHttpRequest'}

    });  


VueJS
=====

在VueJS中你只需要在 ``created`` 函数中增加以下代码，只要你在这类请求时使用 ``Axios`` ::

    axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';


React
=====

    axios.get("your url", {headers: {'Content-Type': 'application/json'}})
