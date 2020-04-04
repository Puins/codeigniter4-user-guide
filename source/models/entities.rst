#####################
使用实体类
#####################

CodeIgniter4 支持实体类作为其数据库层中的一级公民，同时保持它们完全可以随意使用。
它们通常用作存储库模式的一部分，如果您需要更适合的，请直接与 :doc:`Model </models/model>` 一起使用。

.. contents::
    :local:
    :depth: 2

Entity 用法
============

实体类的核心是表示单个数据库行的类。它有类属性表示数据库列，并提供任何其他方法来实现行的业务逻辑。不过，它的核心特性是，它不知道如何持久化自身。那是模型或存储库类的责任。这样，如果您需要保存对象，您不必更改该对象在整个应用程序中的使用方式。这使得在快速原型设计阶段使用JSON或XML文件存储对象，
当你已经证明了这个概念的有效性时，您就可以轻松切换到数据库了。

让我们来看一个非常简单的 ``Users`` 实体，以及如何使用它来使事情变得清晰。

假设您有一个名为 ``users`` 的数据库表，该表具有以下架构::

    id          - integer
    username    - string
    email       - string
    password    - string
    created_at  - datetime

创建 Entity类
-----------------------

现在创建一个新的实体类。 由于没有默认位置可存储这些类，因此不适合使用现有目录结构，在 **app/Entities** 中创建一个新目录。 创建
实体本身位于 **app/Entities/User.php** 。

::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;

    class User extends Entity
    {
        //
    }

简单地说，这就是您需要做的，尽管我们将在一分钟内使它变得更加有用。

创建模型
----------------

首先在 **app/Models/UserModel.php** 创建模型，以便我们可以与之交互::

    <?php namespace App\Models;

    use CodeIgniter\Model;

    class UserModel extends Model
    {
        protected $table         = 'users';
        protected $allowedFields = [
            'username', 'email', 'password'
        ];
        protected $returnType    = 'App\Entities\User';
        protected $useTimestamps = true;
    }

该模型将数据库中的 ``users`` 表用于其所有活动。我们已经设置了 ``$allowedFields`` 属性，它包括我们希望外部类更改的所有字段。 ``id`` , ``created_at`` , 和 ``updated_at`` 字段由类或数据库自动处理，因此我们不想更改它们。最后，我们设置了实体类作为 ``$returnType`` 。这样可以确保模型上所有从数据库返回行的方法都将返回用户实体类的实例，而不是像普通对象那样的对象或数组。

使用 Entity类
-----------------------------

现在所有部分都准备就绪了，您可以像处理任何其他类一样处理实体类::

    $user = $userModel->find($id);

    // 显示
    echo $user->username;
    echo $user->email;

    // 更新
    unset($user->username);
    if (! isset($user->username)
    {
        $user->username = 'something new';
    }
    $userModel->save($user);

    // 创建
    $user = new App\Entities\User();
    $user->username = 'foo';
    $user->email    = 'foo@example.com';
    $userModel->save($user);

您可能已经注意到用户类没有为列设置任何属性，但是您仍然可以像访问公共属性一样访问它们。基类 ``CodeIgniter\Entity`` 为您解决了这一问题，除了提供使用 **isset()** 或 **unset()** 属性检查属性外，还跟踪创建对象或从数据库中提取对象以来哪些列已更改。

当User传递给模型的 **save()** 方法时，它将自动读取属性并保存对模型的 **$allowedFields** 属性中列出的列的所有更改。 它还知道是否要创建
新行，或更新现有行。

快速填充属性
--------------------------

Entity类还提供了一个方法 ``fill()`` ，允许您将键/值对数组推送到类中并填充类属性。数组中的任何属性都将在实体上设置。但是，当通过模型保存数据时，只有 ``$allowedFields`` 中的字段将实际保存到数据库中，因此您可以存储其他数据在你的实体上，而不必担心丢失的字段会被错误地保存。

::

    $data = $this->request->getPost();

    $user = new App\Entities\User();
    $user->fill($data);
    $userModel->save($user);

您也可以在构造函数中传递数据，并且数据将在实例化过程中通过``fill()`` 方法传递。

::

    $data = $this->request->getPost();

    $user = new App\Entities\User($data);
    $userModel->save($user);

处理业务逻辑
=======================

尽管上面的示例很方便，但是它们并不能帮助您实施任何业务逻辑。 基础Entity类实现一些机敏的 ``__get()`` 和 ``__set()`` 方法，将检查特殊方法并使用这些方法而不是使用直接使用属性，允许您强制执行所需的任何业务逻辑或数据转换。

下面是一个更新的用户实体，提供了一些如何使用它的示例::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;
    use CodeIgniter\I18n\Time;

    class User extends Entity
    {
        public function setPassword(string $pass)
        {
            $this->attributes['password'] = password_hash($pass, PASSWORD_BCRYPT);

            return $this;
        }

        public function setCreatedAt(string $dateString)
        {
            $this->attributes['created_at'] = new Time($dateString, 'UTC');

            return $this;
        }

        public function getCreatedAt(string $format = 'Y-m-d H:i:s')
        {
            // Convert to CodeIgniter\I18n\Time object
            $this->attributes['created_at'] = $this->mutateDate($this->attributes['created_at']);

            $timezone = $this->timezone ?? app_timezone();

            $this->attributes['created_at']->setTimezone($timezone);

            return $this->attributes['created_at']->format($format);
        }
    }

首先要注意的是我们添加的方法的名称。 对于每个类，该类都需要将 **snake_case** 列名转换为 **PascalCase** ，并以 ``set`` 或 ``get`` 作为前缀。 然后这些方法每当您使用直接语法（即，$user->email）设置或检索类属性时，都会自动调用。除非您希望从其他类访问它们，否则这些方法不需要是公共的。 例如， ``created_at`` 类属性将通过 ``setCreatedAt()`` 和 ``getCreatedAt()`` 方法访问。

.. note:: 这只在尝试从类外部访问属性时有效。任何类的内部方法必须直接调用 ``setX()`` 和 ``getX()`` 方法。
    
在 ``setPassword()`` 方法中，我们确保始终对密码进行哈希处理。

在 ``setCreatedAt()`` 中，我们将从模型接收的字符串转换为 **DateTime** 对象，确保我们的时区是UTC，因此我们可以轻松转换观众的当前时区。在 ``getCreatedAt()`` 中，它将时间转换为应用程序当前时区中的格式化字符串。

虽然相当简单，但这些示例表明使用实体类可以提供一种非常灵活的方法来强制业务逻辑和创建易于使用的对象。

::

    // 自动对密码进行哈希处理-两者都做相同的事情
    $user->password = 'my great password';
    $user->setPassword('my great password');

数据映射
============

在您的职业生涯中的许多时候，您都会遇到这样一种情况，即应用程序的使用发生了变化，并且数据库中的原始列名不再有意义。 或者您发现您的编码风格更喜欢camelCase类属性，但是您的数据库架构需要snake_case名称。 这些情况很容易处理具有Entity类的数据映射功能。

作为一个例子，假设您有一个在整个应用程序中使用的简化用户实体::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;

    class User extends Entity
    {
        protected $attributes = [
            'id' => null,
            'name' => null,        // 表示用户名
            'email' => null,
            'password' => null,
            'created_at' => null,
            'updated_at' => null,
        ];
    }

您的老板来找您，说没有人再使用用户名，因此您将切换为仅使用电子邮件进行登录。但是他们确实希望对应用程序进行一些个性化设置，因此他们希望您更改名称字段以代表用户的现在全名，而不是当前的用户名。 为了使事情保持整洁并确保事情继续有意义在数据库中，您需要进行一次迁移，以将 `name` 字段重命名为`full_name` ，以使其更加清晰。

忽略这个例子是如何设计的，我们现在有两个关于如何修复User类的选择。 我们可以修改类属性从 ``$name`` 到 ``$full_name``，但这需要在整个应用程序中进行更改。 相反，我们可以只需将数据库中的 ``full_name`` 列映射到 ``$name`` 属性，并完成Entity更改即可::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;

    class User extends Entity
    {
        protected $attributes = [
            'id' => null,
            'name' => null,        // 表示用户名
            'email' => null,
            'password' => null,
            'created_at' => null,
            'updated_at' => null,
        ];

        protected $datamap = [
            'full_name' => 'name'
        ];
    }

通过将新的数据库名称添加到 ``$datamap`` 数组中，我们可以告诉类数据库列的哪个类属性应该可以通过。数组的键是数据库中列的名称，其中数组中的值是要将其映射到的类属性。

在此示例中，当模型在User类上设置 ``full_name`` 字段时，它实际上将该值分配给了类的 ``$name`` 属性，因此可以通过 ``$user->name`` 进行设置和检索。 该值仍可以通过原始的 ``$user->full_name`` 来访问，因为模型需要将它取回并保存数据到数据库。 但是， ``unset`` 和 ``isset`` 仅适用于映射属性 ``$name``，而不适用于原始名称， ``full_name`` 。

修改器
========

Date 类型转换
-------------

默认情况下， Entity类会将名为 ``created_at``, ``updated_at``, 或者 ``deleted_at`` 的字段转换为 :doc:`Time </libraries/time>` 设置或检索时间实例。 Time类提供了大量以一种不可变的、局部的方式提供帮助的方法。

您可以通过将名称添加到 **options['dates']** 数组中来定义要自动转换的属性::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;

    class User extends Entity
    {
        protected $dates = ['created_at', 'updated_at', 'deleted_at'];
    }

现在，当设置了这些属性中的任何一个时，它们将使用应用程序的当前时区，在 **app/Config/App.php** 中设置::

    $user = new App\Entities\User();

    // 转换为时间实例
    $user->created_at = 'April 15, 2017 10:30:00';

    // 现在可以使用任何时间方法:
    echo $user->created_at->humanize();
    echo $user->created_at->setTimezone('Europe/London')->toDateString();

属性类型转换
----------------

您可以使用 **casts** 属性指定应将实体中的属性转换为通用数据类型。此选项应该是一个数组，其中键是类属性的名称，而值是应该被转换的数据类型。
仅当读取值时，强制转换才会生效。不会产生影响永久值的实体或数据库的转换。属性可以被强制转换为以下任何数据类型:
**integer**, **float**, **double**, **string**, **boolean**, **object**, **array**, **datetime**, 和 **timestamp** 。
在类型开头添加问号以将属性标记为可空, 例如。 **?string**, **?integer**.

如下, 如果有一个 **User** 实体具有 **is-banned** 属性，则可以将其强制转换为布尔值::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;

    class User extends Entity
    {
        protected $casts = [
            'is_banned' => 'boolean',
            'is_banned_nullable' => '?boolean'
        ],
    }

Array/Json 转换
------------------
对于存储序列化数组或Json的字段，Array/Json转换尤其有用。当转换为:

* 一个 **array**, 它们将自动地被非序列化,
* 一个 **json**, 它们将自动设置为 ``json_decode($value, false)`` 的值,
* 一个 **json-array**, 它们将自动设置为 ``json_decode($value, true)`` 的值,

当您读取属性值时。与您可以将属性投射到的其他数据类型不同，:

* **array** 转换类型将序列化,
* **json** 和 **json-array** 将使用 ``json_encode`` 函数转换

设置属性时的值::

    <?php namespace App\Entities;

    use CodeIgniter\Entity;

    class User extends Entity
    {
        protected $casts => [
            'options' => 'array',
		    'options_object' => 'json',
		    'options_array' => 'json-array'
        ];
    }

    $user    = $userModel->find(15);
    $options = $user->options;

    $options['foo'] = 'bar';

    $user->options = $options;
    $userModel->save($user);

检查更改的属性
-------------------------------

您可以检查实体属性自创建以来是否已更改。唯一的参数是要检查的属性::

    $user = new User();
    $user->hasChanged('name');      // false

    $user->name = 'Fred';
    $user->hasChanged('name');      // true

或者 若要检查整个实体的更改值，请忽略该参数::

    $user->hasChanged();            // true
