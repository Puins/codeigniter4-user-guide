############
代码模块
############

CodeIgniter支持某种形式的代码模块化，以帮助您创建可重用的代码。模块通常以特定主题为中心，可以将其视为大型应用程序中的微型应用程序。支持框架内的任何标准文件类型，例如控制器，模型，视图，配置文件，辅助函数，语言文件等。模块可能包含着或多或少的你所需要的以上这些类型中。

.. contents::
    :local:
    :depth: 2

==========
命名空间
==========

模块功能的核心元素来自 CodeIgniter使用的 :doc:`PSR4-兼容的 自动加载 </concepts/autoloader>`。尽管任何代码都可以使用PSR4自动加载器和命名空间，但充分利用模块的主要方法是为代码添加命名空间，并将其添加到本节的 **app/Config/Autoload.php** 中 ``psr4``。

例如，假设我们要保留一个简单的博客模块，以便在应用程序之间重复使用。我们可能会使用公司名称 ``Acme`` 创建文件夹，以在其中存储所有模块。我们将其放置 在主项目根目录中的 **app** 目录旁边::

    /acme        // 新的模块目录
    /app
    /public
    /system
    /tests
    /writable

打开 **app/Config/Autoload.php** 并将 **Acme** 命名空间添加到psr4数组属性中::

    $psr4 = [
        'Config'        => APPPATH . 'Config',
        APP_NAMESPACE   => APPPATH,                // 自定义命名空间
        'App'           => APPPATH,                // 确保 ``filters`` 等组件可找到,
        'Acme'          => ROOTPATH.'acme'
    ];

现在已经设置好了，我们可以通过 ``Acme`` 命名空间访问 **acme** 文件夹中的任何文件。仅此一项就可以满足模块工作所需的80％的内容，因此您应确保熟悉命名空间并熟悉它们的使用。将通过所有定义的命名空间自动扫描几种文件类型，这是使用模块的关键要素。

模块中的通用目录结构将模仿主应用程序文件夹::

    /acme
        /Blog
            /Config
            /Controllers
            /Database
                /Migrations
                /Seeds
            /Helpers
            /Language
                /en
            /Libraries
            /Models
            /Views

当然，没有什么能强迫您使用这种确切的结构，您应该以最适合您的模块的方式来组织它，去掉不需要的目录，创建新目录为实体，接口或存储库等。

==============
自动发现
==============

很多时候，您需要为要包括的文件指定完整的命名空间，但是可以通过自动发现许多不同的文件类型来配置CodeIgniter，以使将模块集成到应用程序中变得更加简单。包含:

- :doc:`Events </extending/events>`
- :doc:`Registrars </general/configuration>`
- :doc:`Route files </incoming/routing>`
- :doc:`Services </concepts/services>`

这些是在 **app/Config/Modules.php** 文件中配置的。

自动发现系统通过扫描 **Config/Autoload.php** 中定义的psr4命名空间内的特定目录和文件来工作。

要使自动发现适用于Blog命名空间，我们需要进行一些小的调整。 **Acme** 需要更改为 **Acme\\Blog**，因为命名空间中的每个“模块”都需要完全定义。 定义模块文件夹路径后，发现进程将在该路径上寻找可发现的项目，例如，应在 **/acme/Blog/Config/Routes.php** 中找到路由文件。

开启/关闭 发现
=======================

您可以使用 **$enabled** 类变量打开或关闭系统中的所有自动发现。False将禁用所有发现，优化性能，但会否决模块的特殊功能。

指定发现项目
=======================

使用 **$activeExplorers** 选项，您可以指定自动发现哪些项目。如果不存在该项目，则不会对该项目进行自动发现，但仍会发现数组中的其他项目。

发现 和 Composer
======================

默认情况下还将发现通过Composer安装的软件包。这仅要求Composer知道的命名空间是PSR4命名空间。PSR0命名空间将不会被检测到。

如果在查找文件时不希望扫描Composer的所有已知目录，可以通过在 ``Config\Modules.php`` 中的 ``$discoverInComposer`` 变量关闭此功能::

    public $discoverInComposer = false;

==================
使用文件
==================

本节将介绍每种文件类型（控制器，视图，语言文件等），以及如何在模块中使用它们。其中一些信息在用户指南的相关位置进行了更详细的描述，但在此处进行了复制，以便您更轻松地掌握所有这些部分如何组合在一起。

路由
======

默认情况下，将自动在模块内扫描 :doc:`路由 </incoming/routing>`。可以在上述 **Modules** 配置文件中将其关闭。

.. note:: 由于文件已包含在当前作用域中，因此已经为您定义了 ``$routes`` 实例。如果您尝试重新定义该类，它将导致错误。

控制器
===========

在主 **app/Controllers** 目录下定义的控制器不会自动被URI路由自动检测，所以必须在Routes文件中指定::

    // Routes.php
    $routes->get('blog', 'Acme\Blog\Controllers\Blog::index');

为了减少此处所需的键入量，**group** 路由功能非常有用::

    $routes->group('blog', ['namespace' => 'Acme\Blog\Controllers'], function($routes)
    {
        $routes->get('/', 'Blog::index');
    });

配置文件
============

使用配置文件时，无需进行特殊更改。这些仍然是命名空间类，并使用以下 ``new`` 命令加载::

    $config = new \Acme\Blog\Config\Blog();

只要使用始终可用的 **config()** 函数，就会自动发现配置文件。

迁移
==========

迁移文件将通过定义的命名空间自动发现。所有命名空间里找到的迁移每次都会被自动运行。

种子
=====

只要提供完整的命名空间，种子文件就可以在CLI中使用，也可以在其他种子文件中调用。如果在CLI上调用，则需要提供双反斜杠::

    > php public/index.php migrations seed Acme\\Blog\\Database\\Seeds\\TestPostSeeder

辅助函数
==========

使用 ``helper()`` 方法时，只要位于命名空间 **Helpers** 目录中，就会自动从定义的命名空间中找到它们::

    helper('blog');

语言文件
==============

使用 ``lang()`` 方法时，只要文件遵循与主应用程序目录相同的目录结构，就会从定义的命名空间自动找到语言文件。

库
=========

库总是通过完全命名空间化的类名进行实例化，所以不需要额外的操作::

    $lib = new \Acme\Blog\Libraries\BlogLib();

模型
======

模型总是通过完全命名空间化的类名进行实例化，所以不需要额外的操作::

    $model = new \Acme\Blog\Models\PostModel();

视图
=====

可以使用类命名空间来加载视图，如 :doc:`视图 </outgoing/views>` 文档中所述::

    echo view('Acme\Blog\Views\index');
