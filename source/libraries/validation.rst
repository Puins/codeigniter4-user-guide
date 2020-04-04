##################################################
表单验证
##################################################

CodeIgniter提供了一个全面的数据验证类，该类有助于最大程度地减少您要编写的代码量。

.. contents::
    :local:
    :depth: 2

概述
************************************************

在解释CodeIgniter的数据验证方法之前，让我们描述理想的情况:

#. 显示一个表格。
#. 您填写并提交。
#. 如果您提交的内容无效或可能缺少必填项，则会重新显示包含您的数据以及描述问题的错误消息的表格。
#. 此过程将一直持续到您提交有效的表单为止。

在接收端，脚本必须:

#. 检查所需的数据。
#. 验证数据的类型正确，并且符合正确的条件。例如，如果提交了用户名，则必须对其进行验证以仅包含允许的字符。它必须是最小长度，并且不能超过最大长度。用户名不能是别人的现有用户名，也不能是保留字。等等。
#. 清理数据以确保安全。
#. 如有必要，对数据进行预格式化（是否需要修整数据？HTML编码？等等）。
#. 准备要插入数据库的数据。

尽管上述过程没有什么非常复杂的，但通常需要大量代码，并且为了显示错误消息，通常将各种控制结构放置在HTML表单中。表单验证虽然易于创建，但实现起来通常非常麻烦且乏味。

表单验证教程
************************************************

接下来是实现CodeIgniter的表单验证的“动手”教程。

为了实现表单验证，您需要三件事:

#. 一个包含表单的 :doc:`视图 </outgoing/views>` 文件。
#. 成功提交后将显示包含"success"消息的视图文件。
#. 一种用于接收和处理提交数据的 :doc:`controller </incoming/controllers>` 方法。

让我们以成员注册表单为例创建这三件事。

表单
================================================

使用文本编辑器创建一个名为 **Signup.php** 的表单。在其中放置以下代码，并将其保存到您的 **app/Views/** 文件夹中::

	<html>
	<head>
	    <title>My Form</title>
	</head>
	<body>

        <?= $validation->listErrors() ?>

        <?= form_open('form') ?>

        <h5>Username</h5>
        <input type="text" name="username" value="" size="50" />

        <h5>Password</h5>
        <input type="text" name="password" value="" size="50" />

        <h5>Password Confirm</h5>
        <input type="text" name="passconf" value="" size="50" />

        <h5>Email Address</h5>
        <input type="text" name="email" value="" size="50" />

        <div><input type="submit" value="Submit" /></div>

        </form>

	</body>
	</html>

成功页面
================================================

使用文本编辑器创建一个名为 **Success.php** 的表单。在其中放置以下代码，并将其保存到您的 **app/Views/** 文件夹中::

	<html>
	<head>
	    <title>My Form</title>
	</head>
	<body>

        <h3>Your form was successfully submitted!</h3>

        <p><?= anchor('form', 'Try it again!') ?></p>

	</body>
	</html>

控制器
================================================

使用文本编辑器创建一个名为 **Form.php** 的控制器。在其中放置以下代码，并将其保存到您的 **app/Controllers/** 文件夹中::

	<?php namespace App\Controllers;

	use CodeIgniter\Controller;

	class Form extends Controller
	{
		public function index()
		{
			helper(['form', 'url']);

			if (! $this->validate([]))
			{
				echo view('Signup', [
					'validation' => $this->validator
				]);
			}
			else
			{
				echo view('Success');
			}
		}
	}

试一下！
================================================

要尝试使用表单，请使用类似于以下网址的网址访问您的网站::

	example.com/index.php/form/

如果您提交表单，则只需重新加载表单即可。这是因为您尚未设置任何验证规则。

**由于您尚未告诉Validation类进行任何验证，因此默认情况下它将返回false（布尔false）。validate()方法仅在成功应用规则而没有任何失败的情况下才返回true。**

说明
================================================

您会注意到上述页面的几件事:

表单（Signup.php）是一个标准的Web表单，但有一些例外:

#. 它使用一个表单辅助函数来创建表单开始。从技术上讲，这不是必需的。您可以使用标准HTML创建表单。但是，使用辅助函数的好处是，它基于您的配置文件中的URL为您生成操作URL。这可以使您的应用程序在URL更改的情况下具有更高的可移植性。
#. 在表单顶部，您会注意到以下函数调用:
   ::

	<?= $validation->listErrors() ?>

   此函数将返回验证器发送回的所有错误消息。如果没有消息，则返回一个空字符串。

控制器（Form.php）具有一种方法：``index()``。此方法使用Controller提供的validate方法，并加载视图文件使用的表单辅助函数和URL辅助函数。它还运行验证例程。根据验证是否成功，它会显示表单或成功页面。

加载类库
================================================

该库作为名为 **validation** 的服务加载::

    $validation =  \Config\Services::validation();

这将自动加载 ``Config\Validation`` 文件，该文件包含用于包含多个规则集的设置以及可以轻松重用的规则集合。

.. note:: 您可能永远不需要使用此方法，因为 :doc:`Controller </incoming/controllers>` 和 :doc:`Model </models/model>` 都提供了使验证变得更加容易的方法。

设置验证规则
================================================

使用CodeIgniter，您可以为给定字段设置所需数量的验证规则，并按顺序层叠它们。要设置验证规则，你将使用 ``setRule()``, ``setRules()``, 或 ``withRequest()`` 方法。

setRule()
---------

此方法设置单个规则。它以字段名称作为第一个参数，使用可选标签和带有以竖线分隔的被应用的规则列表的字符串::

    $validation->setRule('username', 'Username', 'required');

**field name** 必须与在发送的任何数据数组的key相匹配，如果数据是从$_POST直接获取，那么它必须是形式输入名称的精确匹配。

setRules()
----------

类似于 ``setRule()``，但接受字段名称及其规则的数组::

    $validation->setRules([
        'username' => 'required',
        'password' => 'required|min_length[10]'
    ]);

要给出带标签的错误消息，您可以设置为::

    $validation->setRules([
        'username' => ['label' => 'Username', 'rules' => 'required'],
        'password' => ['label' => 'Password', 'rules' => 'required|min_length[10]']
    ]);

withRequest()
-------------

在验证从HTTP请求输入的数据时，使用验证库是最常见的情况之一。如果需要，您可以传递当前Request对象的实例，它将获取所有输入数据并将其设置为要验证的数据::

    $validation->withRequest($this->request)
               ->run();

使用验证
************************************************

验证是数组的键
================================================

如果您的数据位于嵌套的关联数组中，则可以使用“点数组语法”轻松验证数据::

    // 测试数据:
    'contacts' => [
        'name' => 'Joe Smith',
        'friends' => [
            [
                'name' => 'Fred Flinstone'
            ],
            [
                'name' => 'Wilma'
            ]
        ]
    ]

    // Joe Smith
    $validation->setRules([
        'contacts.name' => 'required'
    ]);

    // Fred Flintsone & Wilma
    $validation->setRules([
        'contacts.friends.name' => 'required'
    ]);

您可以使用 ``*`` 通配符来匹配数组的任何一级::

    // Fred Flintsone & Wilma
    $validation->setRules([
        'contacts.*.name' => 'required'
    ]);

当您具有一维数组数据时，“点数组语法”也很有用。例如，多选下拉列表返回的数据::

    // 测试数据:
    'user_ids' => [
        1,
        2,
        3
    ]
    // 规则
    $validation->setRules([
        'user_ids.*' => 'required'
    ]);

验证1个值
================================================

根据规则验证一个值::

    $validation->check($value, 'required');

将验证规则集保存到配置文件
=======================================================

Validation类的一个不错的功能是，它允许您将整个应用程序的所有验证规则存储在配置文件中。您将规则组织为"groups"。您可以在每次运行验证时指定一个不同的"groups"。

.. _validation-array:

如何保存规则
-------------------------------------------------------

要存储您的验证规则，只需在 ``Config\Validation`` 类中使用您的组的名称创建一个新的公共属性。该元素将包含一个包含验证规则的数组。如前所示，验证数组将具有以下原型::

    class Validation
    {
        public $signup = [
            'username'     => 'required',
            'password'     => 'required',
            'pass_confirm' => 'required|matches[password]',
            'email'        => 'required|valid_email'
        ];
    }

您可以在调用 ``run()`` 方法时指定要使用的组::

    $validation->run($data, 'signup');

您还可以通过将属性命名为与组相同的名称，在此配置文件中存储自定义错误消息，属性名为组名附加 ``_errors``。当使用该组时，这些将自动用于任何错误::

    class Validation
    {
        public $signup = [
            'username'     => 'required',
            'password'     => 'required',
            'pass_confirm' => 'required|matches[password]',
            'email'        => 'required|valid_email'
        ];

        public $signup_errors = [
            'username' => [
                'required'    => 'You must choose a username.',
            ],
            'email'    => [
                'valid_email' => 'Please check the Email field. It does not appear to be valid.'
            ]
        ];
    }

或通过数组传递所有设置::

    class Validation
    {
        public $signup = [
            'username' => [
                'label'  => 'Username',
                'rules'  => 'required',
                'errors' => [
                    'required' => 'You must choose a {field}.'
                ]
            ],
            'email'    => 'required|valid_email'
        ];

        public $signup_errors = [
            'email' => [
                'valid_email' => 'Please check the Email field. It does not appear to be valid.'
            ]
        ];
    }

有关数组格式的详细信息，请参见下文。

获取和设置规则组
-------------------------------------------------------

**获取规则组**

此方法从验证配置中获取一个规则组::

    $validation->getRuleGroup('signup');

**设置规则组**

此方法将从验证配置到验证服务设置一个规则组::

    $validation->setRuleGroup('signup');

运行多个验证
=======================================================

.. note:: ``run()`` 方法不会重置错误状态。如果先前的运行失败， ``run()`` 它将始终返回false，``getErrors()`` 并将返回所有先前的错误，直到明确重置为止。

如果您打算运行多个验证（例如，对不同的数据集或使用彼此不同的规则进行验证），则可能需要 ``$validation->reset()`` 在每次运行之前调用，以消除上一次运行中的错误。请注意， ``reset()`` 会使您先前设置的任何数据，规则或自定义错误均无效，因此 ``setRules()``，``setRuleGroup()`` 等需要重复::

    for ($userAccounts as $user) {
        $validation->reset();
        $validation->setRules($userAccountRules);
        if (!$validation->run($user)) {
            // 处理验证错误
        }
    }

处理错误
************************************************

验证库提供了几种方法来帮助您设置错误消息，提供自定义错误消息以及检索要显示的一个或多个错误。

默认情况下，错误消息是从 ``system/Language/en/Validation.php`` 中的语言字符串派生的，其中每个规则都有一个条目。

.. _validation-custom-errors:

设置自定义错误消息
=============================

两个 ``setRule()`` 和 ``setRules()`` 方法可以接受将被用作错误特定于每个字段作为其最后一个参数自定义消息的数组。因为错误是针对每个实例量身定制的，所以这为用户提供了非常愉快的体验。如果未提供自定义错误消息，则将使用默认值。

这是提供自定义错误消息的两种方法。

作为最后一个参数::

    $validation->setRules([
            'username' => 'required|is_unique[users.username]',
            'password' => 'required|min_length[10]'
        ],
        [   // Errors
            'username' => [
                'required' => 'All accounts must have usernames provided',
            ],
            'password' => [
                'min_length' => 'Your password is too short. You want to get hacked?'
            ]
        ]
    );

或作为标记样式::

    $validation->setRules([
            'username' => [
                'label'  => 'Username',
                'rules'  => 'required|is_unique[users.username]',
                'errors' => [
                    'required' => 'All accounts must have {field} provided'
                ]
            ],
            'password' => [
                'label'  => 'Password',
                'rules'  => 'required|min_length[10]',
                'errors' => [
                    'min_length' => 'Your {field} is too short. You want to get hacked?'
                ]
            ]
        ]
    );

如果你想有一个字段的“human”的名称，或可选参数的一些规则允许（如MAX_LENGTH），或者得到了验证，你可以添加值 ``{field}``, ``{param}`` 和 ``{value}`` 标记你的消息，分别为::

    'min_length' => 'Supplied value ({value}) for {field} must have at least {param} characters.'

字段(Username)附加规则(min_length[6])值为“Pizza”, 错误显示: “Supplied value (Pizza) for Username must have at least 6 characters.”

.. note:: 如果您传递最后一个参数，则带标签的样式错误消息将被忽略。

获取所有错误
==================

如果需要检索失败字段的所有错误消息，则可以使用 ``getErrors()`` 方法::

    $errors = $validation->getErrors();

    // 返回:
    [
        'field1' => 'error message',
        'field2' => 'error message',
    ]

如果没有错误，将返回一个空数组。

获取一个错误
======================

您可以使用 ``getError()`` 方法检索单个字段的错误。唯一的参数是字段名称::

    $error = $validation->getError('username');

如果不存在错误，将返回一个空字符串。

检查存在错误
=====================

您可以使用 ``hasError()`` 方法检查是否存在错误。唯一的参数是字段名称::

    if ($validation->hasError('username'))
    {
        echo $validation->getError('username');
    }

自定义错误显示
************************************************

当您调用 ``$validation->listErrors()`` 或 ``$validation->showError()`` 时，它将在后台加载一个视图文件，该文件确定错误的显示方式。默认情况下，它们在包装div上显示为 ``errors`` 类。您可以轻松创建新视图并在整个应用程序中使用它们。

创建视图
==================

第一步是创建自定义视图。它们可以放置在 ``view()`` 方法可以找到它们的任何位置，这意味着标准View目录或任何命名空间的View文件夹都可以使用。例如，您可以在 **/app/Views/_errors_list.php** 中创建一个新视图::

    <div class="alert alert-danger" role="alert">
        <ul>
        <?php foreach ($errors as $error) : ?>
            <li><?= esc($error) ?></li>
        <?php endforeach ?>
        </ul>
    </div>

视图中包含一个名为 ``$errors`` 的数组，其中包含错误列表，其中键是发生错误的字段的名称，值是错误消息，如下所示::

    $errors = [
        'username' => 'The username field must be unique.',
        'email'    => 'You must provide a valid email address.'
    ];

您实际上可以创建两种视图。第一个数组包含所有错误，这就是我们刚刚查看的内容。另一种类型更简单，仅包含一个 ``$error`` 包含错误消息的变量。这与必须指定字段的 ``showError()`` 方法一起使用::

    <span class="help-block"><?= esc($error) ?></span>

配置
=============

创建视图后，需要让Validation库知道它们。打开 ``Config/Validation.php``。在内部，您将找到 ``$templates`` 属性，可以在其中列出所需的任意多个自定义视图，并提供一个简短的别名，以供参考。如果要从上方添加示例文件，则该文件应类似于::

    public $templates = [
        'list'    => 'CodeIgniter\Validation\Views\list',
        'single'  => 'CodeIgniter\Validation\Views\single',
        'my_list' => '_errors_list'
    ];

指定模板
=======================

您可以通过将其别名作为 ``listErrors()`` 方法的第一个参数传递来指定要使用的模板::

    <?= $validation->listErrors('my_list') ?>

当显示特定于字段的错误时，您可以将别名作为第二个参数传递给 ``showError`` 方法，该错误应在字段名称之后::

    <?= $validation->showError('username', 'my_single') ?>

创建自定义规则
************************************************

规则存储在简单的命名空间类中。只要自动加载器可以找到它们，它们就可以存储在您想要的任何位置。这些文件称为 **RuleSets**。要添加新的 **RuleSet**，请编辑 **Config/Validation.php** 并将新文件添加到 ``$ruleSets`` 数组中::

    public $ruleSets = [
		\CodeIgniter\Validation\Rules::class,
		\CodeIgniter\Validation\FileRules::class,
		\CodeIgniter\Validation\CreditCardRules::class,
	];

您可以将其添加为具有完全限定的类名称的简单字符串，也可以使用 ``::class`` 如上所示的后缀添加。这里的主要好处是，它在更高级的IDE中提供了一些额外的导航功能。

在文件本身内，每个方法都是一个规则，必须接受一个字符串作为第一个参数，并且必须返回布尔值true或false值，如果它通过测试则表示true，否则返回false::

    class MyRules
    {
        public function even(string $str): bool
        {
            return (int)$str % 2 == 0;
        }
    }

默认情况下，系统将在 ``CodeIgniter\Language\en\Validation.php`` 中查找所使用的语言字符串及错误。在自定义规则中，您可以通过引用第二个参数中的$error变量来提供错误消息::

    public function even(string $str, string &$error = null): bool
    {
        if ((int)$str % 2 != 0)
        {
            $error = lang('myerrors.evenError');
            return false;
        }

        return true;
    }

现在，您可以像使用其他任何规则一样使用新的自定义规则::

    $this->validate($request, [
        'foo' => 'required|even'
    ]);

允许参数
===================

如果您的方法需要使用参数，则该函数至少需要三个参数：要验证的字符串，参数字符串以及包含提交表单的所有数据的数组。$data数组对于诸如此类的规则特别有用，像 ``require_with`` 需要检查另一个提交字段的值以使其结果基于::

	public function required_with($str, string $fields, array $data): bool
	{
		$fields = explode(',', $fields);

		// 如果字段存在，我们可以安全地假设该字段存在，不管相应的搜索字段是否存在。
		$present = $this->required($str ?? '');

		if ($present)
		{
			return true;
		}

		// 还在这儿？如果$data中存在任何字段，则测试失败，因为$fields是列表
		$requiredFields = [];

		foreach ($fields as $field)
		{
			if (array_key_exists($field, $data))
			{
				$requiredFields[] = $field;
			}
		}

		// 删除任何空值的键，因为就这一点而言，它们并不真正存在。
		$requiredFields = array_filter($requiredFields, function ($item) use ($data) {
			return ! empty($data[$item]);
		});

		return empty($requiredFields);
	}

如上所述，可以将自定义错误作为第四个参数返回。

可用规则
***************

以下是所有可用的本地规则的列表:

.. note:: Rule是一个字符串；参数之间不得有空格，尤其是“is_unique”规则。“ignore_value”前后不能有空格。

- "is_unique[supplier.name,uuid, $uuid]"   不正确
- "is_unique[supplier.name,uuid,$uuid ]"   不正确
- "is_unique[supplier.name,uuid,$uuid]"    正确

======================= =========== =============================================================================================== ===================================================
规则                    参数         描述                                                                                             示例
======================= =========== =============================================================================================== ===================================================
alpha                   No          如果字段中没有字母字符，则失败。
alpha_space             No          如果字段包含字母字符或空格以外的任何内容，则失败。
alpha_dash              No          如果字段包含字母数字字符，下划线或破折号以外的任何内容，则失败。
alpha_numeric           No          如果字段包含字母数字字符以外的内容，则失败。
alpha_numeric_space     No          如果字段包含字母数字或空格字符以外的内容，则失败。
alpha_numeric_punct     No          如果字段包含字母数字，空格或以下有限的标点符号集之外的内容，则失败: ~ (tilde), ! (exclamation) 
                                    , # (number), $ (dollar), % (percent), & (ampersand), * (asterisk), - (dash)
                                    , _ (underscore), + (plus), = (equals), | (vertical bar), : (colon), . (period).
decimal                 No          如果字段中包含除十进制数字以外的任何内容，则失败。也接受数字的+或-号。 
differs                 Yes         如果字段与参数中的字段相同，则失败。                                                                    differs[field_name]
exact_length            Yes         如果字段不完全是参数值，则失败。一个或多个逗号分隔的值。                                                    exact_length[5] or exact_length[5,8,12]
greater_than            Yes         如果字段小于或等于参数值或非数字，则失败。                                                              greater_than[8]
greater_than_equal_to   Yes         如果字段小于参数值或非数字，则失败。                                                                   greater_than_equal_to[5]
hex                     No          如果字段包含十六进制字符以外的任何内容，则失败。
if_exist                No          如果存在此规则，则验证将仅在存在字段密钥的情况下返回可能的错误，而不管其值如何。
in_list                 Yes         如果字段不在预定列表内，则失败。                                                                        in_list[red,blue,green]
integer                 No          如果字段包含除整数以外的任何内容，则失败。
is_natural              No          如果字段包含非自然数之外的任何内容，则失败：0、1、2、3等。
is_natural_no_zero      No          如果字段包含自然数以外的任何内容（零，1、2、3等除外），则失败。
is_not_unique           Yes         检查数据库以查看给定值是否存在。可以忽略按字段/值进行过滤的记录（当前仅接受一个过滤器）。
is_unique               Yes         检查此字段值在数据库中是否存在。（可选）设置要忽略的列和值，在更新记录以忽略自身时很有用。                         is_unique[table.field,ignore_field,ignore_value]
less_than               Yes         如果字段大于或等于参数值或非数字，则失败。                                                               less_than[8]
less_than_equal_to      Yes         如果字段大于参数值或非数字，则失败。                                                                    less_than_equal_to[8]
matches                 Yes         该值必须与参数中字段的值匹配。                                                                           matches[field]
max_length              Yes         如果字段长于参数值，则失败。                                                                             max_length[8]
min_length              Yes         如果字段小于参数值，则失败。                                                                             min_length[3]
numeric                 No          如果字段中包含数字字符以外的任何内容，则失败。
regex_match             Yes         如果字段与正则表达式不匹配，则失败。                                                                       regex_match[/regex/]
permit_empty            No          允许该字段接收一个空数组，空字符串，null或false。
required                No          如果字段是空数组，空字符串，null或false，则失败。
required_with           Yes         当数据中存在其他任何必填字段时，该字段是必填字段。                                                           required_with[field1,field2]
required_without        Yes         当数据中存在所有其他字段时，该字段是必需的，但不是必需的。                                                     required_without[field1,field2]
string                  No          确认元素为字符串的 alpha* 规则的通用替代方法
timezone                No          如果字段确实与时区 ``timezone_identifiers_list`` 匹配，则失败 
valid_base64            No          如果字段包含有效Base64字符以外的任何内容，则失败。
valid_json              No          如果字段不包含有效的JSON字符串，则失败。
valid_email             No          如果该字段不包含有效的电子邮件地址，则失败。
valid_emails            No          如果逗号分隔列表中提供的任何值不是有效电子邮件，则失败。
valid_ip                No          如果提供的IP无效，则失败。接受可选参数“ ipv4”或“ ipv6”以指定IP格式。                                          valid_ip[ipv6]
valid_url               No          如果字段不包含有效的URL，则失败。
valid_date              No          如果字段不包含有效日期，则失败。接受可选参数以匹配日期格式。                                                valid_date[d/m/Y]
valid_cc_number         Yes         验证信用卡号是否与指定提供者使用的格式匹配。当前受支持的提供商包括:                                          valid_cc_number[amex]
                                    American Express (amex), China Unionpay (unionpay),
                                    Diners Club CarteBlance (carteblanche), Diners Club (dinersclub), Discover Card (discover),
                                    Interpayment (interpayment), JCB (jcb), Maestro (maestro), Dankort (dankort), NSPK MIR (mir),
                                    Troy (troy), MasterCard (mastercard), Visa (visa), UATP (uatp), Verve (verve),
                                    CIBC Convenience Card (cibc), Royal Bank of Canada Client Card (rbc),
                                    TD Canada Trust Access Card (tdtrust), Scotiabank Scotia Card (scotia), BMO ABM Card (bmoabm),
                                    HSBC Canada Card (hsbc)
======================= =========== =============================================================================================== ===================================================

规则文件上传
======================

这些验证规则使您可以进行基本检查，以检查上传的文件是否满足您的业务需求。由于文件上载HTML字段的值不存在，并且存储在$_FILES全局变量中，因此输入字段的名称需要使用两次。一次指定字段名称，就像使用其他任何规则一样，但再次指定为所有文件上传相关规则的第一个参数::

    // 在HTML文件中
    <input type="file" name="avatar">

    // 在控制器中
    $this->validate([
        'avatar' => 'uploaded[avatar]|max_size[avatar,1024]'
    ]);

======================= =========== ================================================================================ ========================================
规则                     参数        描述                                                                              示例
======================= =========== ================================================================================ ========================================
uploaded                Yes         如果参数名称与任何上传文件的名称不匹配，则失败。                                         uploaded[field_name]
max_size                Yes         如果参数中命名的上传文件大于第二个参数（以千字节（kb）为单位），则失败。                     max_size[field_name,2048]                                    
max_dims                Yes         如果上传的图像的最大宽度和高度超过值，则失败。第一个参数是字段名称。                         max_dims[field_name,300,150]
                                    第二个是宽度，第三个是高度。如果无法将文件确定为图像，也会失败。
mime_in                 Yes         如果文件的mime类型不是参数中列出的一种，则失败。                                         mime_in[field_name,image/png,image/jpg]
ext_in                  Yes         如果文件的扩展名不是参数中列出的扩展名之一，则失败。                                      ext_in[field_name,png,jpg,gif]
is_image                Yes         如果无法根据mime类型将文件确定为图像，则失败。                                          is_image[field_name]
======================= =========== ================================================================================ ========================================

文件验证规则适用于单个文件上传和多个文件上传。

.. note:: 您还可以使用任何允许最多两个参数的本地PHP函数，其中至少需要一个参数（以传递字段数据）。
