************************
扩展控制器
************************

CodeIgniter的核心Controller不应更改，但在 **app/Controllers/BaseController.php**。任何新的控制器都应该扩展 ``BaseController`` 以获取预加载组件的优势和您提供的任何附加功能::

	<?php namespace App\Controllers;
	
	use CodeIgniter\Controller;
	
	class Home extends BaseController {
	
	}

预加载组件
=====================

基本控制器是加载您打算加载的任何辅助函数，模型，库，服务等的好地方，每次您的项目运行时使用。 应该将辅助函数添加到预定义的 ``$helpers`` 数组中。 例如，如果您想要通用的 ``html`` 和 ``text`` 辅助函数::

	protected $helpers = ['html', 'text'];

要加载的任何其他组件或要处理的数据都应添加到构造函数 ``initController()`` 中。例如，如果您的项目大量使用会话库，您可能希望在初始化它::

	public function initController(...)
	{
		// 不要编辑此行
		parent::initController($request, $response, $logger);
		
		$this->session = \Config\Services::session();
	}

附加方法
==================

基本控制器不可路由（系统配置会将其路由到 ``404页面未找到`` ）。 作为增加的安全性考虑您创建的所有新方法都应声明为 ``protected`` 或者 ``private`` ，并且只能通过您创建的扩展 ``BaseController`` 的控制器被访问。

其他选择
=============

您可能会发现您需要多个基本控制器。 您可以创建新的基本控制器，只要您制造的任何其他控制器扩展了正确的基本控制器即可。 例如，如果您的项目有一个涉及的公共界面和一个简单的管理门户，您可能想将 ``BaseController`` 扩展到公共控制器，并为所有管理控制器设置 ``AdminController`` 。

如果不想使用 **BaseController** ，可以通过让控制器扩展系统控制器来绕过它::

	class Home extends Controller
	{
	
	}
