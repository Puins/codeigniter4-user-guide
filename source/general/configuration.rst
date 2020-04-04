#############
配置
#############

每个框架都使用配置文件来定义许多参数和初始设置。CodeIgniter配置文件定义简单的类，其中所需的设置是公共属性。 

与许多其他框架不同，CodeIgniter可配置项不包含在单个文件中。相反，每个需要可配置项目的类都将具有一个与使用它的类同名的配置文件。您将在 **/app/Config** 文件夹中找到应用程序配置文件。

.. contents::
    :local:
    :depth: 2

使用配置文件
================================

您可以通过几种不同的方式访问类的配置文件。

- 通过使用 ``new`` 关键字创建实例::
 
	// 手动创建新配置对象
	$config = new \Config\Pager();

- 通过使用 ``config()`` 函数::

	// 使用config函数创建新对象
	$config = config('Pager', false);

	// 使用配置函数获取共享实例
	$config = config('Pager');

	// 使用config函数命名空间的存取配置类
	$config = config( 'Config\\Pager' );

所有配置对象属性都是公共的，因此您可以像访问其他任何属性一样访问设置::

        $config = config('Pager');
	// 作为对象属性访问设置
	$pageSize = $config->perPage;

如果没有提供命名空间，它将在所有定义的命名空间以及 **/app/Config/** 中查找文件。

CodeIgniter随附的所有配置文件都以 ``Config`` 为命名空间。在应用程序中使用此命名空间将提供最佳性能，因为它确切知道在哪里可以找到文件。

您可以使用其他命名空间以便将配置文件放在所需的任何文件夹中。这使您可以将配置文件放在生产服务器上的一个不可通过Web访问的文件夹中，同时将其保存在 **/app** 下，以便在开发过程中轻松访问。

创建配置文件
============================

当需要新配置时，首先要在所需位置创建一个新文件。默认文件位置（大多数情况下建议使用）是 **/app/Config**。该类应使用适当的命名空间，并且扩展 ``CodeIgniter\Config\BaseConfig`` 以确保可以接收特定环境的设置。

定义该类，并用代表您的设置的公共属性填充它::

    <?php namespace Config;

    use CodeIgniter\Config\BaseConfig;

    class CustomClass extends BaseConfig
    {
    	public $siteName  = 'My Great Site';
    	public $siteEmail = 'webmaster@example.com';

    }

环境变量
=====================

当今应用程序安装的最佳实践之一是使用环境变量。原因之一是环境变量很容易在部署之间进行更改，而无需更改任何代码。在部署之间，配置可能会发生很大变化，但是代码却不会。例如，多个环境，例如开发人员的本地计算机和生产服务器，通常需要针对每个特定设置使用不同的配置值。

环境变量也应用于任何私有内容，例如密码，API密钥或其他敏感数据。

环境变量 和 CodeIgniter
=====================================

CodeIgniter 使使用 ``dotenv`` 文件设置环境变量变得简单而轻松。该术语来自文件名，该文件名以“env”之前的点开头。

CodeIgniter希望 **.env** 与 ``system`` 和  ``app`` 目录一起位于项目的根目录。有一个与CodeIgniter一起分发的名为 **env** 的模板文件，该文件位于项目根目录下（注意开始时没有点（**.**）吗？）。它包含了您的项目可能使用的大量变量，这些变量已被分配为空，虚拟或默认值。您可以通过将该模板重命名为 **.env** 或为其创建一个名为 **.env** 的副本，来将该文件用作应用程序的起始位置。

.. important:: 确保版本控制系统未跟踪 **.env** 文件。对于git意味着将其添加到 **.gitignore** 。否则，可能导致敏感凭据公开。

设置以简单的名称/值对集合（以等号分隔）的形式存储在 **.env** 文件中。

::

	S3_BUCKET = dotenv
	SECRET_KEY = super_secret_key
        CI_ENVIRONMENT = development

当您的应用程序运行时，**.env** 将自动加载，并将变量放入环境中。如果环境中已经存在变量，则不会覆盖该变量。加载的环境变量使用 ``getenv()``, ``$_SERVER``, 或者 ``$_ENV`` 访问。

::

	$s3_bucket = getenv('S3_BUCKET');
	$s3_bucket = $_ENV['S3_BUCKET'];
	$s3_bucket = $_SERVER['S3_BUCKET'];

嵌套变量
=================

为节省输入时间，您可以通过将变量名称包装在文件中来重用文件中已指定的变量 ``${...}``

::

        BASE_DIR="/var/webroot/project-root"
        CACHE_DIR="${BASE_DIR}/cache"
        TMP_DIR="${BASE_DIR}/tmp"

命名空间变量
====================

有时候，您会有多个具有相同名称的变量。系统需要一种方法来知道正确的设置。这个问题可以通过“ *命名空间* ”变量来解决。

命名空间变量使用点符号来限定变量名称，因此在合并到环境中时它们将是唯一的。这是通过包括一个可区分的前缀，后跟一个点（.），然后是变量名本身来完成的。
::

    // 非命名空间变量
    name = "George"
    db=my_db

    // 命名空间变量
    address.city = "Berlin"
    address.country = "Germany"
    frontend.db = sales
    backend.db = admin
    BackEnd.db = admin

配置类和环境变量
===============================================

实例化配置类时，将考虑将任何 *命名空间* 的环境变量合并到配置对象的属性中。

如果命名空间变量的前缀与配置类的命名空间完全匹配，则设置的结尾部分（点后）将被视为配置属性。如果与现有配置属性匹配，则环境变量的值将替换配置文件中的相应值。如果不匹配，则配置类属性保持不变。在这种用法中，前缀必须是该类的完整（区分大小写）命名空间。
::

    Config\App.CSRFProtection  = true    
    Config\App.CSRFCookieName = csrf_cookie
    Config\App.CSPEnabled = true

.. note:: 命名空间前缀和属性名称均区分大小写。它们必须与配置类文件中定义的完整命名空间和属性名称完全匹配。

这同样适用于一个短的前缀，这是仅使用配置类名的小写版本命名空间。如果短前缀与类名匹配，则 **.env** 中的值将替换配置文件值。
::

    app.CSRFProtection  = true    
    app.CSRFCookieName = csrf_cookie
    app.CSPEnabled = true

.. note:: 使用 **短前缀** 时，属性名称仍必须与类定义的名称完全匹配。

将环境变量视为数组
========================================

命名空间的环境变量可以进一步视为数组。如果前缀与配置类匹配，那么如果环境变量名称的其余部分还包含一个点，则将其视为数组引用。
::

    // 常规的命名空间变量
    Config\SimpleConfig.name = George

    // 数组化的命名空间变量
    Config\SimpleConfig.address.city = "Berlin"
    Config\SimpleConfig.address.country = "Germany"

如果这是指SimpleConfig配置对象，则以上示例将被视为::

    $address['city']    = "Berlin";
    $address['country'] = "Germany";

该 ``$address`` 属性的任何其他元素将保持不变。

您也可以使用数组属性名称作为前缀。如果环境文件包含以下内容，则结果将与上面相同。
::

    // 数组化的命名空间变量
    Config\SimpleConfig.address.city = "Berlin"
    address.country = "Germany"


处理不同的环境
===============================

通过使用单独的 **.env** 文件，并修改其值以满足该环境的需要，可以轻松完成配置多个环境的操作。

该文件不应包含应用程序使用的每个配置类的所有可能的设置。实际上，它应仅包括特定于环境的项目或敏感细节（例如密码和API密钥）以及不应公开的其他信息。但是在部署之间进行任何更改都是公平的。

在每种环境中，请将 **.env** 文件放置在项目的根文件夹中。对于大多数设置，与 ``system`` 和 ``app`` 同级目录。

不要使用版本控制系统跟踪 **.env** 文件。如果您这样做了，并且该存储库已公开，则您会将敏感信息放在每个人都可以找到的地方。

.. _registrars:

注册器
==========

配置文件还可以指定任意数量的 “注册商”，即可以提供其他配置属性的任何其他类。这是通过 ``registrars`` 在配置文件中添加属性，并保存候选注册商名称数组来完成的。

::

    protected $registrars = [
        SupportingPackageRegistrar::class
    ];

为了充当 “注册商”，如此标识的类必须具有与配置类相同的静态函数，并且应返回属性设置的关联数组。

实例化您的配置对象时，它将遍历指定类中的 ``$registrars``。对于每个包含与配置类匹配的方法名称的类，它将调用该方法，并以对命名空间变量所述的相同方式合并所有返回的属性。

为此的示例配置类设置::

    <?php namespace App\Config;

    use CodeIgniter\Config\BaseConfig;

    class MySalesConfig extends BaseConfig
    {
        public $target        = 100;
        public $campaign      = "Winter Wonderland";
        protected $registrars = [
            '\App\Models\RegionalSales';
        ];
    }

…以及相关的区域销售模式可能类似于::

    <?php namespace App\Models;

    class RegionalSales
    {
        public static function MySalesConfig()
        {
            return ['target' => 45, 'actual' => 72];
        }
    }

在上面的示例中，当实例化 `MySalesConfig` 时，它将以声明的两个属性结束，但是通过将 `RegionalSalesModel` 视为 “注册商” ，`$target` 属性的值将被覆盖。产生的配置属性::

    $target   = 45;
    $campaign = "Winter Wonderland";
