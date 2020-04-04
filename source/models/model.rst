#########################
使用 CodeIgniter4 的模型
#########################

.. contents::
    :local:
    :depth: 2

模型
======

模型提供了一种与数据库中的特定表进行交互的方式。 他们提供开箱即用的辅助方法，对于与数据库表进行交互所需的许多标准方式的方法，包括查找记录，更新记录，删除记录等。

访问模型
================

模型通常存储在 ``app/Models`` 目录中。它们应该有一个与其匹配的命名空间目录中的位置，如 ``namespace App\Models`` 。

您可以通过创建新实例或使用 ``model()`` 辅助函数来访问类中的模型。

::

    // 手动创建新类
    $userModel = new App\Models\UserModel();

    // 用 ``model()`` 函数创建一个新类
    $userModel = model('App\Models\UserModel', false);

    // 创建模型的共享实例
    $userModel = model('App\Models\UserModel');

    // 使用提供的数据库连接创建共享实例
    // 当没有给定命名空间时，它将搜索所有命名空间
    // 系统知道并尝试定位 ``UserModel`` 类。
    $db = db_connect('custom');
    $userModel = model('UserModel', true, $db);


CodeIgniter4 的模型
===================

CodeIgniter确实提供了一个模型类，该类提供了一些不错的功能， 包含:

- 自动数据库连接
- 基本的CRUD方法
- 模型内验证
- 自动分页
- 更多

此类为构建自己的模型提供了坚实的基础，使您可以快速构建应用程序的模型层。

创建模型
===================

为了更好的使用CodeIgniter的模型，您只需创建一个新的模型类扩展 ``CodeIgniter\Model`` ::

        <?php namespace App\Models;

        use CodeIgniter\Model;

		class UserModel extends Model
		{

		}

该空类提供了对数据库连接，查询生成器，以及许多其他便利方法。

连接数据库
--------------------------

首次实例化该类时，如果没有数据库连接实例传递给构造函数，它将自动连接到配置中设置的默认数据库组。 您可以通过将 **DBGroup** 属性添加到您的类来修改每个模型使用的组。这样可以确保在模型中通过适当的方式对 ``$this->db`` 进行任何引用连接。
::

    <?php namespace App\Models;

    use CodeIgniter\Model;

	class UserModel extends Model
	{
		protected $DBGroup = 'group_name';
	}

您可以用数据库中定义的数据库组的名称替换配置文件中的 ``group_name`` 。

配置模型
----------------------

模型类具有一些配置选项，可以将其设置为允许类的方法为您无缝地工作。 所有CRUD方法都使用前两个方法来确定使用什么表以及如何找到所需的记录::

        <?php namespace App\Models;

        use CodeIgniter\Model;

		class UserModel extends Model
		{
			protected $table      = 'users';
			protected $primaryKey = 'id';

			protected $returnType = 'array';
			protected $useSoftDeletes = true;

			protected $allowedFields = ['name', 'email'];

			protected $useTimestamps = false;
			protected $createdField  = 'created_at';
			protected $updatedField  = 'updated_at';
			protected $deletedField  = 'deleted_at';

			protected $validationRules    = [];
			protected $validationMessages = [];
			protected $skipValidation     = false;
		}

**$table**

指定此模型主要使用的数据库表。 这仅适用于内置的CRUD方法。 您不仅限于自己使用此表查询。

**$primaryKey**

这是唯一标识此表中记录的列的名称。这个不一定必须匹配数据库中指定的主键，但是与诸如 ``find()`` 之类的方法一起使用，以知道要将指定值匹配到哪个列。

.. note:: 所有模型都必须指定一个主键，以允许所有功能按预期工作。

**$returnType**

该模型的CRUD方法将使您远离工作，并自动返回结果数据，而不是Result对象。 此设置使您可以定义返回的数据类型。 有效值为 'array', 'object' 或完整值可以与Result对象的 **getCustomResultObject()** 一起使用的类的限定名称方法。

**$useSoftDeletes**

如果为true，则任何 **delete** 方法调用都将在数据库中设置 ``deleted_at`` ，而不是实际删除行。这可以在数据可能在其他地方被引用时保留数据，或者可以维护可还原对象的“回收站”，甚至可以简单地将其保留为安全跟踪的一部分。如果为true， **find** 方法将只返回未删除的行，除非在调用 **find** 方法之前调用 ``withDeleted()`` 方法。

这需要数据库中的DATETIME或INTEGER字段作为模型的$dateFormat设置。默认的字段名是 ``deleted_at`` 但是这个名称可以是使用 **$deletedField** 属性配置为您选择的任何名称。

**$allowedFields**

此数组应使用在保存、插入或更新方法中。除此之外的任何字段名都将被丢弃。这有助于保护而不是仅仅从表单中获取输入并将其全部扔给模型，从而导致潜在的批量分配漏洞。

**$useTimestamps**

此布尔值确定是否将当前日期自动添加到所有插入以及更新中。如果为true，将以 **$dateFormat** 指定的格式设置当前时间。这个要求该表在相应的位置具有名为 ``created_at`` 和 ``updated_at`` 的列数据类型。

**$createdField**

指定应使用哪个数据库字段来保留数据记录创建时间戳。将其保留为空以避免更新（即使启用了 useTimestamps）

**$updatedField**

指定应使用哪个数据库字段来保留数据记录更新时间戳。将其保留为空以避免更新 (即使启用了 useTimestamps)

**$dateFormat**

此值与 **$useTimstamps** 和 **$useSoftDeletes** 一起使用，以确保日期值的准确类型被插入到数据库中。默认情况下，这将创建 **DATETIME** 值，但是有效选项为：``datetime``、``date`` 或 ``int`` （PHP时间戳）。使用 ``useSoftDeletes`` 或日期格式无效或丢失的 ``usetimstamps`` 将导致异常。

**$validationRules**

包含验证规则数组，如 :ref:`validation-array` 中所述或包含验证组名称的字符串，如同一节所述。下面将详细介绍。

**$validationMessages**

包含在验证期间应使用的自定义错误消息数组，如说明见 :ref:`validation-custom-errors`。下面将详细介绍。

**$skipValidation**

是否应在所有 ``inserts`` 和 ``updates`` 期间跳过验证。默认值为false，这意味着数据将始终尝试验证。这主要由 ``skipValidation`` 属性决定，但可以更改为 ``true`` ，以便此模型将永远无需验证。

**$beforeInsert**
**$afterInsert**
**$beforeUpdate**
**$afterUpdate**
**afterFind**
**afterDelete**

这些数组允许您指定将在数据上运行的回调方法同时被指定在属性名中。

处理数据
=================

查询数据
------------

提供了几个函数来对表执行基本的CRUD工作，包括find(),insert(), update(), delete() 等。

**find()**

返回一行，其中主键与作为第一个参数传入的值匹配::

	$user = $userModel->find($user_id);

值将以 **$returnType** 中指定的格式返回。

您可以通过传递primaryKey值数组来指定要返回的多行::

	$users = $userModel->find([1,2,3]);

如果没有传入任何参数，将返回该模型表中的所有行，从而有效地执行操作像 **findAll()** 一样，尽管不太明确。

**findColumn()**

返回null或列值的索引数组::

 	$user = $userModel->findColumn($column_name);

**$column_name** 应该是单列的名称，否则您将获得 **DataException**。

**findAll()**

返回所有结果::

	$users = $userModel->findAll();

在调用此方法之前，可以根据需要通过插入查询生成器命令来修改此查询::

	$users = $userModel->where('active', 1)
	                   ->findAll();

可以将限制值(**$limit**)和偏移值(**$offset**)作为第一个值和第二个值传入参数，分别::

	$users = $userModel->findAll($limit, $offset);

**first()**

返回结果集中的第一行。 最好与查询生成器结合使用。
::

	$user = $userModel->where('deleted', 0)
	                  ->first();

**withDeleted()**

如果 **$useSoftDeletes** 为true，则 **find*()** 方法将不返回 ``deleted_at IS NOT NULL`` 的任何行。要临时覆盖此方法，可以在调用 **find*()** 方法之前使用 **withDeleted()** 方法。
::

	// 仅获取未删除的行 (deleted = 0)
	$activeUsers = $userModel->findAll();

	// 获取所有行
	$allUsers = $userModel->withDeleted()
	                      ->findAll();

**onlyDeleted()**

尽管 **withDeleted()** 将同时返回已删除行和未删除行，但此方法会修改接下来的 **find*()** 方法仅返回软删除的行::

	$deletedUsers = $userModel->onlyDeleted()
	                          ->findAll();

保存数据
-----------

**insert()**

一个关联的数据数组作为创建新的数据库中的数据行。数组的键必须与 **$table** 中列的名称匹配，而数组的值是要为该键保存的值::

	$data = [
		'username' => 'darth',
		'email'    => 'd.vader@theempire.com'
	];

	$userModel->insert($data);

**update()**

更新数据库中的现有记录。第一个参数是要更新的记录的 **$primaryKey** 。数据的关联数组作为第二个参数传递给此方法。数组的键必须与 **$table** 中列的名称匹配，而数组的值是要为该键保存的值::

	$data = [
		'username' => 'darth',
		'email'    => 'd.vader@theempire.com'
	];

	$userModel->update($id, $data);

通过传递一个主键数组作为第一个参数，可以通过一次调用更新多个记录::

    $data = [
		'active' => 1
	];

	$userModel->update([1, 2, 3], $data);

当您需要更灵活的解决方案时，可以将参数保留为空，其功能类似于查询生成器的 **update** 命令，具有 **validation** ， **events** ，等等::

    $userModel
        ->whereIn('id', [1,2,3])
        ->set(['active' => 1])
        ->update();

**save()**

这是用于处理插入或更新记录的 **insert()** 和 **update()** 方法的包装器,根据是否找到与 **$primaryKey** 值匹配的数组键自动进行::

	// 定义为模型属性
	$primaryKey = 'id';

	// 是否 insert()
	$data = [
		'username' => 'darth',
		'email'    => 'd.vader@theempire.com'
	];

	$userModel->save($data);

	// 由于找到了主键 ``id`` ，因此执行更新。 
	$data = [
		'id'       => 3,
		'username' => 'darth',
		'email'    => 'd.vader@theempire.com'
	];
	$userModel->save($data);

save方法还可以通过识别非简单结果对象来简化自定义类结果对象的处理对象，并将其公共值和受保护的值捕获到一个数组中，然后将其传递给适当的插入或更新方法。 这使您可以非常干净地使用Entity类。 实体类是代表对象类型的单个实例的简单类，例如用户，博客文章，工作等。类负责维护围绕对象本身的业务逻辑，例如格式化以某种方式保存的元素等。它们对如何将它们保存到数据库一无所知。 最简单的说，他们可能看起来像这样::

	namespace App\Entities;

	class Job
	{
		protected $id;
		protected $name;
		protected $description;

		public function __get($key)
		{
			if (property_exists($this, $key))
			{
				return $this->$key;
			}
		}

		public function __set($key, $value)
		{
			if (property_exists($this, $key))
			{
				$this->$key = $value;
			}
		}
	}

一个非常简单的模型可能看起来像::

        use CodeIgniter\Model;

	class JobModel extends Model
	{
		protected $table = 'jobs';
		protected $returnType = '\App\Entities\Job';
		protected $allowedFields = [
			'name', 'description'
		];
	}

此模型处理来自 ``jobs`` 表的数据，并将所有结果作为 ``App\Entities\Job`` 的实例返回。当需要将该记录持保存到数据库中时，需要编写自定义方法，或使用模型的 ``save()`` 方法检查类，获取任何公共和私有属性，并将它们保存到数据库中::

	// 检索Job实例
	$job = $model->find(15);

	// 做些改变
	$job->name = "Foobar";

	// 保存更改
	$model->save($job);

.. note:: 如果您经常使用实体，CodeIgniter提供了一个内置的 :doc:`Entity class </models/entities>` 它提供了几个方便的特性，使开发实体更简单.

删除数据
-------------

**delete()**

将主键值作为第一个参数，并从模型表中删除匹配的记录::

	$userModel->delete(12);

如果模型的**$useSoftDeletes**值为true，这将更新行以将 ``deleted_at`` 设置为当前日期和时间。 您可以通过将第二个参数设置为true来强制永久删除。

可以将主键数组作为第一个参数传入，以便一次删除多个记录::

    $userModel->delete([1,2,3]);

如果未传入任何参数，则将类似于查询生成器的**delete**方法，需要where调用在前::

    $userModel->where('id', 12)->delete();

**purgeDeleted()**

通过永久删除所有具有 ``deleted_at IS NOT NULL`` 的行来清除数据库表。::

	$userModel->purgeDeleted();

验证数据
---------------

对于许多人来说，验证模型中的数据是确保将数据保存为单个数据的首选方法，无需重复代码。 Model类提供了一种自动验证所有数据的方法在使用 ``insert()``, ``update()``, 或 ``save()`` 方法保存到数据库之前。

第一步是使用被请求字段和规则填充 ``$validationRules`` 类属性。 如果您有要使用的自定义错误消息，请将其放置在 ``$validationMessages`` 数组中::

	class UserModel extends Model
	{
		protected $validationRules    = [
			'username'     => 'required|alpha_numeric_space|min_length[3]',
			'email'        => 'required|valid_email|is_unique[users.email]',
			'password'     => 'required|min_length[8]',
			'pass_confirm' => 'required_with[password]|matches[password]'
		];

		protected $validationMessages = [
			'email'        => [
				'is_unique' => 'Sorry. That email has already been taken. Please choose another.'
			]
		];
	}

通过函数将验证消息设置为字段的另一种方法,

.. php:function:: setValidationMessage($field, $fieldMessages)

	:参数	  string	    $field
	:参数	  array	        $fieldMessages

	此函数将设置按字段显示的错误消息。

	用法示例::

			$fieldName = 'name';
			$fieldValidationMessage = array(
							'required'   => 'Your name is required here',
					);
			$model->setValidationMessage($fieldName, $fieldValidationMessage);

.. php:function:: setValidationMessages($fieldMessages)

	:参数	 array	       $fieldMessages

	此函数将设置字段消息。

	用法示例::

			$fieldValidationMessage = array(
					'name' => array(
							'required'   => 'Your baby name is missing.',
							'min_length' => 'Too short, man!',
					),
			);
			$model->setValidationMessages($fieldValidationMessage);

现在，无论何时调用 ``insert()``, ``update()``, 或者 ``save()`` 方法，数据都将得到验证。 如果失败了该模型将返回布尔 **false**。 您可以使用 ``errors()`` 方法检索验证错误::

	if ($model->save($data) === false)
	{
		return view('updateUser', ['errors' => $model->errors()];
	}

这将返回一个包含字段名及其相关错误的数组，该数组可用于显示表单顶部的错误，或单独显示它们::

	<?php if (! empty($errors)) : ?>
		<div class="alert alert-danger">
		<?php foreach ($errors as $field => $error) : ?>
			<p><?= $error ?></p>
		<?php endforeach ?>
		</div>
	<?php endif ?>

如果您希望在 **Validation** 配置文件中组织规则和错误消息，只需将 ``$validationRules`` 设置为您创建的验证规则组的名称即可::

	class UserModel extends Model
	{
		protected $validationRules = 'users';
	}

检索验证规则
---------------------------

您可以通过访问模型的 ``validationRules`` 属性来检索模型的验证规则::

    $rules = $model->validationRules;

还可以通过调用访问器仅检索这些规则的一个子集方法，带有选项::

    $rules = $model->getValidationRules($options);

 ``$options`` 参数是一个带有一个元素的关联数组，其键是 ``except`` 或 ``only`` ，并且它的值是一个由感兴趣的字段名组成的数组。::

    // 获取除 ``username`` 字段外的所有规则
    $rules = $model->getValidationRules(['except' => ['username']]);
    // 只获取 ``city`` 和 ``state`` 字段的规则
    $rules = $model->getValidationRules(['only' => ['city', 'state']]);

验证占位符
-----------------------

该模型提供了一种简单的方法，可以根据传递给规则的数据替换规则的某些部分。 这个听起来相当晦涩，但在使用 ``is_unique`` 验证规则时尤其方便。 占位符就是以 **$data** 形式传入的字段（或数组键）的名称，并用大括号括起来。 这将是替换为匹配的传入字段的 **value**。 举例说明这一点::

    protected $validationRules = [
        'email' => 'required|valid_email|is_unique[users.email,id,{id}]'
    ];

在这组规则中，它声明电子邮件地址在数据库中应该是唯一的，除了行具有与占位符值匹配的id。假设表单 **POST** 数据具有以下内容::

    $_POST = [
        'id' => 4,
        'email' => 'foo@example.com'
    ]

然后 ``{id}`` 占位符将替换为数字 **4**，从而给出此修改后的规则::

    protected $validationRules = [
        'email' => 'required|valid_email|is_unique[users.email,id,4]'
    ];

因此，当它验证电子邮件是唯一的时，它将忽略数据库中具有 ``id=4`` 的行。

这也可以用于在运行时创建更多的动态规则，只要您注意传入的密钥与表单数据不冲突。

保护字段
-----------------

为了防止大规模分配攻击，模型类 **requires** 列出所有字段名可以在插入和更新 ``$allowedFields`` 类属性期间更改。提供的任何数据除此之外，这些将在命中数据库之前删除。这对于确保时间戳，或者主键不被更改很有用。
::

	protected $allowedFields = ['name', 'email', 'address'];

有时，您会发现需要多次更改这些元素。 这通常是在 ``testing`` , ``migrations`` , 或者 ``seeds`` 。 在这种情况下，您可以打开或关闭保护::

	$model->protect(false)
	      ->insert($data)
	      ->protect(true);

使用 Query Builder
--------------------------

当需要它时，您可以随时访问该模型的数据库连接的查询生成器共享实例::

	$builder = $userModel->builder();

该生成器已使用模型的 **$table** 设置。

您还可以非常优雅的在同一个链式调用中使用查询生成器方法和模型的CRUD方法::

	$users = $userModel->where('status', 'active')
			   ->orderBy('last_login', 'asc')
			   ->findAll();

.. note:: 您还可以无缝地访问模型的数据库连接::

			$user_name = $userModel->escape($name);

运行时返回类型更改
----------------------------

您可以指定使用 **find*()** 方法作为类属性时应返回数据的格式，``$returnType``。 不过，有时候您可能希望数据以其他格式返回。 该模型
提供的方法可让您做到这一点。

.. note:: 这些方法只更改下一个 **find*()** 方法调用的返回类型。之后，它将重置为默认值。

**asArray()**

从下一个 **find*()** 方法作为关联数组返回数据::

	$users = $userModel->asArray()->where('status', 'active')->findAll();

**asObject()**

将下一个 **find*()** 方法中的数据作为标准对象或自定义类入口返回::

	// 作为标准对象返回
	$users = $userModel->asObject()->where('status', 'active')->findAll();

	// 作为自定义类实例返回
	$users = $userModel->asObject('User')->where('status', 'active')->findAll();

处理大量数据
--------------------------------

有时，您需要处理大量数据，可能会出现内存不足的风险。为了简化这个过程，可以使用chunk（）方法来获取更小的数据块，然后做你的工作。第一个参数是要在单个块中检索的行数。第二个参数是将作为每行数据调用的闭包使用。

这最适合在cronjobs、数据导出或其他大型任务中使用。
::

	$userModel->chunk(100, function ($data)
	{
		// do something.
		// $data是一行数据。
	});

模型事件
============

在模型的执行过程中，可以指定多个要运行的回调方法。这些方法可用于规范化数据，哈希密码，保存相关实体等等。 以下模型执行中的每个点都可以通过类属性受到影响: **$beforeInsert**, **$afterInsert**, **$beforeUpdate**, **afterUpdate**, **afterFind**, 和 **afterDelete**。

定义回调
------------------

通过首先在模型中创建要使用的新类方法来指定回调。 这个类将永远接收 **$data** 数组作为其唯一参数。 **$data** 数组的确切内容在事件之间会有所不同，但是将始终包含一个名为 **$data** 的键，该键包含传递给原始方法的主要数据。 在这种情况下的insert*或update*方法，它们是将插入数据库中的键/值对。这个主数组还将包含传递给该方法的其他值，稍后将进行详细说明。 回调方法必须返回原始的 ``$data`` 数组，以便其他回调具有完整的信息。

::

	protected function hashPassword(array $data)
	{
		if (! isset($data['data']['password']) return $data;

		$data['data']['password_hash'] = password_hash($data['data']['password'], PASSWORD_DEFAULT);
		unset($data['data']['password'];

		return $data;
	}

指定要运行的回调
---------------------------

您可以通过将方法名称添加到适当的类属性（beforeInsert，afterUpdate，等等）。 可以将多个回调添加到单个事件中，并将它们依次处理。 您可以在多个事件中使用相同的回调::

	protected $beforeInsert = ['hashPassword'];
	protected $beforeUpdate = ['hashPassword'];

事件参数
----------------

由于传递给每个回调的确切数据略有不同，下面是有关 **$data** 参数中的内容的详细信息传递到每个事件:

================ =========================================================================================================
Event            $data contents
================ =========================================================================================================
beforeInsert      **data** = (key/value)正在插入的键/值对。 如果将对象或实体类传递给insert方法，则首先将其转换为数组。

afterInsert       **id** = 新行的主键，失败则为0。
                  **data** = (key/value)正在插入的键/值对。
                  **result** = 通过查询生成器使用的insert()方法的结果。

beforeUpdate      **id** = 正在更新的行的主键。
                  **data** = (key/value)正在插入的键/值对。如果将对象或实体类传递给insert方法，则首先将其转换为数组。

afterUpdate       **id** = 正在更新的行的主键。
                  **data** = (key/value)正在更新的键/值对。
                  **result** = 通过查询生成器使用的update()方法的结果。

afterFind         因find*方法而异。见下表:

- find()          **id** = 正在搜索的行的主键。
                  **data** = 结果数据行，如果未找到结果，则为空。

- findAll()       **data** = 生成的数据行，如果未找到结果，则为空。
                  **limit** = 要查找的行数。
                  **offset** = 搜索期间要跳过的行数。
				  
- first()         **data** = 在搜索期间找到的结果行，如果未找到则为空。

beforeDelete      因delete*方法而异。见下表:

- delete()        **id** = 正在删除的行的主键。
                  **purge** = 布尔值是否应硬删除软删除行。

afterDelete       因delete*方法而异。见下表:

- delete()        **id** = 正在删除的行的主键。
                  **purge** = 布尔值是否应硬删除软删除行。
                  **result** = 对查询生成器调用delete()的结果。
                  **data** = 未使用。
================ =========================================================================================================


手动创建模型
=====================

您无需扩展任何特殊的类即可为您的应用程序创建模型。 您所需要做的就是数据库连接的实例，这样就OK了。 这可以绕过CodeIgniter的模型为您提供了开箱即用的功能，并创建了完全自定义的体验。

::

    <?php namespace App\Models;

	use CodeIgniter\Database\ConnectionInterface;

	class UserModel
	{
		protected $db;

		public function __construct(ConnectionInterface &$db)
		{
			$this->db =& $db;
		}
	}
