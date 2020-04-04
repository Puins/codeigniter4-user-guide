############
本土化
############

.. contents::
    :local:
    :depth: 2

********************
使用语言环境设置
********************

CodeIgniter提供了多种工具来帮助您将应用程序本地化为不同的语言。尽管应用程序的完全本地化是一个复杂的主题，但使用不同支持的语言替换应用程序中的字符串很简单。

语言字符串存储在 **app/Language** 目录中，每种支持的语言都有一个子目录::

    /app
        /Language
            /en
                app.php
            /fr
                app.php

.. important:: 语言环境设置检测仅适用于使用IncomingRequest类的基于Web的请求。命令行请求将不具有这些功能。

语言环境配置
======================

每个站点都将使用其默认语言/区域设置。可以在 **Config/App.php** 中进行设置::

    public $defaultLocale = 'en';

该值可以是您的应用程序用来管理文本字符串和其他格式的任何字符串。建议使用 `BCP 47 <http://www.rfc-editor.org/rfc/bcp/bcp47.txt>`_ 语言代码。这将导致语言代码，例如美国英语为en-US，法语/法国为fr-FR。在 `W3C'的站点 <https://www.w3.org/International/articles/language-tags/>`_ 上可以找到对此更具可读性的介绍。

如果找不到完全匹配的系统，该系统足够智能，可以使用更多通用语言代码。如果将语言环境代码设置为 **en-US** ，而我们仅为 **en** 设置了语言文件， 则将使用这些语言文件，因为更具体的 **en-US** 不存在任何语言文件。但是，如果在 **app/Language/en-US** 中存在一个语言目录，则将首先使用该目录。

语言环境检测
================

支持两种方法来在请求期间检测正确的语言环境。第一种是“设置并忘记”方法，该方法将自动执行 :doc:`内容协商 </incoming/content_negotiation>`，以供您确定要使用的正确语言环境。第二种方法允许您在路由中指定一个用于设置语言环境的片段。

内容协商
-------------------

您可以通过在 ``Config/App`` 中设置两个附加设置来将内容协商设置为自动进行。第一个值告诉Request类，我们确实希望协商语言环境，因此只需将其设置为true即可::

    public $negotiateLocale = true;

启用此功能后，系统将根据您在 ``$supportLocales`` 中定义的语言环境数组自动协商正确的语言。如果在您支持的语言和请求的语言之间找不到匹配项，则将使用$supportedLocales中的第一项。在以下示例中，如果未找到匹配项，则将使用 **en** 语言环境::

    public $supportedLocales = ['en', 'es', 'fr-FR'];

在路由中
---------

第二种方法使用自定义占位符来检测所需的语言环境并在“请求”上进行设置。占位符 ``{locale}`` 可以作为片段放置在您的路由中。如果存在，则匹配片段内容的将是您的语言环境::

    $routes->get('{locale}/books', 'App\Books::index');

在此示例中，如果用户尝试访问 ``http://example.com/fr/books``，则假定该语言环境被配置为有效语言环境，则该语言环境将被设置为 ``fr``。

.. note:: 如果该值与App配置文件中定义的有效语言环境不匹配，则会在其位置使用默认语言环境。

检索当前语言环境
=============================

通过 ``getLocale()`` 方法，始终可以从IncomingRequest对象中检索当前语言环境。如果您的控制器正在扩展 ``CodeIgniter\Controller``，则可以通过 ``$this->request`` 使用::

    <?php namespace App\Controllers;

    class UserController extends \CodeIgniter\Controller
    {
        public function index()
        {
            $locale = $this->request->getLocale();
        }
    }

或者，您可以使用 :doc:`Services 类 </concepts/services>` 来检索当前请求::

    $locale = service('request')->getLocale();

*********************
语言本地化
*********************

创建语言文件
=======================

语言没有必需的任何特定命名约定。该文件应在逻辑上进行命名，以描述其保存的内容的类型。例如，假设您要创建一个包含错误消息的文件。您可能会简单地命名为：**Errors.php**。

在文件内，您将返回一个数组，其中数组中的每个元素都有一个语言键和要返回的字符串::

        'language_key' => 'The actual message to be shown.'

.. note:: 优良作法是对给定文件中的所有消息使用通用前缀，以避免与其他文件中名称相似的项目发生冲突。例如，如果要创建错误消息，则可以为它们加上 ``error_`` 前缀   

::

    return [
        'errorEmailMissing'    => 'You must submit an email address',
        'errorURLMissing'      => 'You must submit a URL',
        'errorUsernameMissing' => 'You must submit a username',
    ];

基本用法
===========

通过将文件名和语言键作为 ``lang()`` 的第一个参数（以句点（.）分隔），可以使用辅助函数从任何语言文件中检索文本。例如，要从 ``Errors`` 语言文件加载 ``errorEmailMissing`` 字符串，请执行以下操作::

    echo lang('Errors.errorEmailMissing');

如果当前语言环境的文件中不存在所请求的语言键，则该字符串将原封不动地传递回。在此示例中，如果不存在，它将返回“Errors.errorEmailMissing”。

更换参数
--------------------

.. note:: 以下功能都需要将 `intl <https://www.php.net/manual/en/book.intl.php>`_ 扩展名加载到系统上才能正常工作。如果未加载扩展，则不会尝试替换。可以在 `Sitepoint <https://www.sitepoint.com/localization-demystified-understanding-php-intl/>`_ 上找到很棒的概述。

您可以传递一个值数组来替换语言字符串中的占位符，作为 ``lang()`` 函数的第二个参数 。这允许非常简单的数字转换和格式设置::

    // 语言文件, Tests.php:
    return [
        "apples"      => "I have {0, number} apples.",
        "men"         => "I have {1, number} men out-performed the remaining {0, number}",
        "namedApples" => "I have {number_apples, number, integer} apples.",
    ];

    // 显示  "I have 3 apples."
    echo lang('Tests.apples', [ 3 ]);

如果是数字，则占位符中的第一项对应于数组中该项的索引::

    // 显示  "The top 23 men out-performed the remaining 20"
    echo lang('Tests.men', [20, 23]);

如果您愿意，还可以使用命名键来使事情变得更简单。::

    // 显示  "I have 3 apples."
    echo lang("Tests.namedApples", ['number_apples' => 3]);

显然，您可以做的不仅仅是数字替换。根据基础库的 `官方ICU文档 <https://unicode-org.github.io/icu-docs/apidoc/released/icu4c/classMessageFormat.html#details>`_，可以替换以下类型的数据:

* numbers - integer, currency, percent
* dates - short, medium, long, full
* time - short, medium, long, full
* spellout - spells out numbers (i.e. 34 becomes thirty-four)
* ordinal
* duration

这里有一些示例::

    // 语言文件, Tests.php
    return [
        'shortTime'  => 'The time is now {0, time, short}.',
        'mediumTime' => 'The time is now {0, time, medium}.',
        'longTime'   => 'The time is now {0, time, long}.',
        'fullTime'   => 'The time is now {0, time, full}.',
        'shortDate'  => 'The date is now {0, date, short}.',
        'mediumDate' => 'The date is now {0, date, medium}.',
        'longDate'   => 'The date is now {0, date, long}.',
        'fullDate'   => 'The date is now {0, date, full}.',
        'spelledOut' => '34 is {0, spellout}',
        'ordinal'    => 'The ordinal is {0, ordinal}',
        'duration'   => 'It has been {0, duration}',
    ];

    // 显示  "The time is now 11:18 PM"
    echo lang('Tests.shortTime', [time()]);
    // 显示  "The time is now 11:18:50 PM"
    echo lang('Tests.mediumTime', [time()]);
    // 显示  "The time is now 11:19:09 PM CDT"
    echo lang('Tests.longTime', [time()]);
    // 显示  "The time is now 11:19:26 PM Central Daylight Time"
    echo lang('Tests.fullTime', [time()]);

    // 显示  "The date is now 8/14/16"
    echo lang('Tests.shortDate', [time()]);
    // 显示  "The date is now Aug 14, 2016"
    echo lang('Tests.mediumDate', [time()]);
    // 显示  "The date is now August 14, 2016"
    echo lang('Tests.longDate', [time()]);
    // 显示  "The date is now Sunday, August 14, 2016"
    echo lang('Tests.fullDate', [time()]);

    // 显示  "34 is thirty-four"
    echo lang('Tests.spelledOut', [34]);

    // 显示  "It has been 408,676:24:35"
    echo lang('Tests.ordinal', [time()]);

您应该确保阅读MessageFormatter类和基础ICU格式，以更好地了解其具有的功能，例如执行条件替换，复数等。前面提供的两个链接都将为您提供一个关于可用选项的好主意。

指定语言环境
-----------------

要指定替换参数时要使用的其他语言环境，可以将语言环境作为第三个参数传递给 ``lang()`` 方法。::

    // 显示  "The time is now 23:21:28 GMT-5"
    echo lang('Test.longTime', [time()], 'ru-RU');

    // 显示  "£7.41"
    echo lang('{price, number, currency}', ['price' => 7.41], 'en-GB');
    // 显示  "$7.41"
    echo lang('{price, number, currency}', ['price' => 7.41], 'en-US');

嵌套数组
-------------

语言文件还允许嵌套数组使列表等的使用更加容易。::

    // Language/en/Fruit.php

    return [
        'list' => [
            'Apples',
            'Bananas',
            'Grapes',
            'Lemons',
            'Oranges',
            'Strawberries'
        ]
    ];

    // 显示  "Apples, Bananas, Grapes, Lemons, Oranges, Strawberries"
    echo implode(', ', lang('Fruit.list'));

语言回退
=================

例如，如果您有给定语言环境的一组消息 ``Language/en/app.php``，则可以为该语言环境添加语言变体，例如，每个语言变体都位于其自己的文件夹 ``Language/en-US/app.php`` 中。

您只需要为那些针对该语言环境变量而本地化的消息提供值。任何缺少的消息定义将自动从主要语言环境环境设置中提取。

它会变得更好-如果将新消息添加到框架中，并且您还没有机会针对您的语言环境进行翻译，则本地化可以一直追溯到英语。

因此，如果您使用的是语言环境 ``fr-CA``，则将首先在 ``Language/fr/CA`` 中搜索本地化消息，然后在 ``Language/fr`` 中搜索，最后在 ``Language/en`` 中搜索。

信息翻译
====================

我们在 `自己的 存储库 <https://github.com/codeigniter4/translations>`_ 中有一组“官方”翻译 。

您可以下载该存储库，然后将其 ``Language`` 文件夹复制到 ``app``。由于 ``App`` 命名空间已映射到您的 ``app`` 文件夹，因此将自动提取合并的翻译。

或者，更好的做法是 在您的项目内部 ``composer require codeigniter4/translations``，并且由于翻译文件夹得到了适当的映射，翻译的消息将被自动提取。
