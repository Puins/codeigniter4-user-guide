###############
Session 库
###############

Session类允许您维护用户的“状态”并在他们浏览您的网站时跟踪他们的活动。

CodeIgniter带有一些Session存储驱动程序，您可以在目录的最后部分中看到它们:

.. contents::
    :local:
    :depth: 2

.. raw:: html

  <div class="custom-index container"></div>

使用 Session 类
*********************************************************************

初始化Session
==================================================================

Sessions通常会在每次加载页面时在全局范围内运行，因此应该神奇地初始化Session类。

要访问并初始化session::

	$session = \Config\Services::session($config);

``$config`` 参数是可选的-您的应用程序配置。如果未提供，服务注册将实例化您的默认注册。

加载后，可以使用以下方法使用Sessions库对象::

	$session

或者，您可以使用将使用默认配置选项的辅助函数。此版本阅读起来更友好一些，但没有任何配置选项。

::

	$session = session();

Sessions 如何工作？
=====================

加载页面后，session类将检查用户的浏览器是否发送了有效的会话cookie。如果会话Cookie并不存在（或者如果它不匹配存储在服务器上的任何一个或已过期）的新会话将被创建和保存。

如果确实存在有效的会话，则其信息将被更新。对于每次更新，如果配置了会话ID，则可以对其进行重新生成。

对您来说很重要的一点是，一旦初始化，Session类就会自动运行。您无需执行任何操作即可导致上述现象发生。如下所示，您可以使用会话数据，但是读取，写入和更新会话的过程是自动的。

.. note:: 在CLI下，Session库将自动停止自身，因为这是一个完全基于HTTP协议的概念。

关于并发的说明
------------------------

除非您要开发使用AJAX的网站，否则可以跳过本节。但是，如果您遇到了性能问题，那么本说明正是您所需要的。

早期版本的CodeIgniter中的会话未实现锁定，这意味着使用同一会话的两个HTTP请求可以完全同时运行。使用更合适的技术术语-请求是非阻塞的。

但是，会话上下文中的非阻塞请求也意味着不安全，因为在一个请求中对会话数据的修改（或会话ID再生）可能会干扰第二个并发请求的执行。这个细节是许多问题的根源，也是CodeIgniter 4拥有完全重写的Session库的主要原因。

我们为什么要告诉你这个？因为在尝试找出性能问题的原因之后，您可能会得出结论，锁定是问题所在，因此研究了如何删除锁定……

不要那样做！移除锁是错误的，它将给您带来更多问题！

锁定不是问题，而是解决方案。您的问题是，您已经打开了会话，但已经处理了该会话，因此不再需要它。因此，您需要的是在不再需要当前请求后关闭会话。
::

    $session->destroy();

什么是Session数据？
=====================

Session数据只是与特定会话ID（cookie）关联的数组。

如果您以前在PHP中使用过会话，则应该熟悉PHP的 `$_SESSION 超全局变量 <https://www.php.net/manual/en/reserved.variables.session.php>`_  （如果没有，请阅读该链接上的内容）。

CodeIgniter使用与PHP提供的会话处理程序机制相同的方式来访问其会话数据。使用会话数据就像操作（读取，设置和未设置值）$_SESSION 数组一样简单。

此外，CodeIgniter还提供2种特殊类型的会话数据，下面将进一步说明：flashdata和tempdata。

检索 Session 数据
=======================

Session数组中的任何信息都可以通过 ``$_SESSION`` 超全局变量获得::

	$_SESSION['item']

或通过常规访问器方法::

	$session->get('item');

或通过魔术获取器::

	$session->item

甚至通过会话辅助方法::

	session('item');

``item`` 是您要获取的项目相对应的数组键。例如，要将先前存储的 ``name`` 项分配给 ``$name`` 变量，您可以这样做::

	$name = $_SESSION['name'];

	// 或:

	$name = $session->name

	// 或:

	$name = $session->get('name');

.. note:: 如果您要访问的项目不存在，则 ``get()`` 方法返回NULL。

如果要检索所有现有的用户数据，则可以简单地省略item键（magic getter仅适用于单个属性值）::

	$_SESSION

	// 或:

	$session->get();

添加 Session 数据
===================

假设某个特定用户登录到您的网站。身份验证后，您可以将其用户名和电子邮件地址添加到会话中，从而使您可以全局使用该数据，而不必在需要时运行数据库查询。

您可以像其他变量一样，将数据简单地分配给 ``$_SESSION`` 数组。或作为 ``$session`` 的属性。

不推荐使用以前的userdata方法，但是您可以将包含新会话数据的数组传递给``set()`` 方法::

	$session->set($array);

``$array`` 是包含新数据的关联数组。这是一个示例::

	$newdata = [
		'username'  => 'johndoe',
		'email'     => 'johndoe@some-site.com',
		'logged_in' => TRUE
	];

	$session->set($newdata);

如果要一次添加一个会话数据一个值，``set()`` 还支持以下语法::

	$session->set('some_name', 'some_value');

如果要验证会话值是否存在，只需使用 ``isset()`` 命令进行检查::

	// 如果'some_name'项目不存在或为NULL，则返回FALSE，否则返回TRUE:
	isset($_SESSION['some_name'])

或者您可以调用 ``has()``::

	$session->has('some_name');

向 session 数据推送新值
=================================

push方法用于将新值推送到作为数组的会话值上。例如，如果'hobbies'键包含一个兴趣爱好数组，则可以将新值添加到数组中，如下所示::

$session->push('hobbies', ['sport'=>'tennis']);

删除 Session 数据
=====================

与其他任何变量一样，可以通过 ``unset()`` 取消设置在 ``$_SESSION`` 的值::

	unset($_SESSION['some_name']);

	// 或者多个值:

	unset(
		$_SESSION['some_name'],
		$_SESSION['another_name']
	);

同样，就像 ``set()`` 可以用来向会话添加信息一样，``remove()`` 也可以通过传递会话key来删除信息。例如，如果要从会话数据数组中删除'some_name'::

	$session->remove('some_name');

此方法还接受要取消设置的项目键数组::

	$array_items = ['username', 'email'];
	$session->remove($array_items);

Flashdata
=========

CodeIgniter支持“flashdata”或仅对下一个请求会话数据可用，然后将其自动清除。

这可能非常有用，特别是对于一次性的信息，错误或状态消息（例如：“记录2已删除”）。

应当注意，flashdata变量是常规会话变量，在CodeIgniter会话处理程序内部进行管理。

要将现有项目标记为 "flashdata"::

	$session->markAsFlashdata('item');

如果要将多个项目标记为flashdata，只需将键作为数组传递::

	$session->markAsFlashdata(['item', 'item2']);

要添加flashdata::

	$_SESSION['item'] = 'value';
	$session->markAsFlashdata('item');

或者使用 ``setFlashdata()`` 方法::

	$session->setFlashdata('item', 'value');

与 ``set()`` 相同的方式，您还可以将数组传递给 ``setFlashdata()``。

读取flashdata变量与通过 ``$_SESSION`` 读取常规会话数据相同::

	$_SESSION['item']

.. important:: 当通过键检索单个项时，``get()`` 方法将返回flashdata项。但是，从会话中获取所有用户数据时，它不会返回flashdata。

但是，如果您想确定自己正在读取"flashdata"（而不是其他种类的数据），则也可以使用 ``getFlashdata()`` 方法::

	$session->getFlashdata('item');

或者，要获取包含所有flashdata的数组，只需省略key参数::

	$session->getFlashdata();

.. note:: 如果找不到该项目，则 ``getFlashdata()`` 方法返回NULL。

如果发现需要通过其他请求保留flashdata变量，则可以使用 ``keepFlashdata()`` 方法来实现。您可以传递单个项或一组flashdata项来保留。

::

	$session->keepFlashdata('item');
	$session->keepFlashdata(['item1', 'item2', 'item3']);

Tempdata
========

CodeIgniter还支持“tempdata”或具有特定到期时间的会话数据。该值过期或会话过期或被删除后，该值将自动删除。

与flashdata相似，tempdata变量由CodeIgniter会话处理程序在内部进行管理。

要将现有项目标记为“tempdata”，只需将其key和有效时间（以秒为单位）传递给 ``mark_as_temp()`` 方法::

	// 300秒后“item”将被删除
	$session->markAsTempdata('item', 300);

您可以通过两种方式将多个项目标记为临时数据，具体取决于您是否希望它们都具有相同的到期时间::

	// “item”和“item2”都将在300秒后过期
	$session->markAsTempdata(['item', 'item2'], 300);

	// “item”将在300秒后删除，而“item2”将在240秒后删除
	$session->markAsTempdata([
		'item'	=> 300,
		'item2'	=> 240
	]);

要添加tempdata::

	$_SESSION['item'] = 'value';
	$session->markAsTempdata('item', 300); // Expire in 5 minutes

或者使用 ``setTempdata()`` 方法::

	$session->setTempdata('item', 'value', 300);

您还可以将数组传递给 ``set_tempdata()``::

	$tempdata = ['newuser' => TRUE, 'message' => 'Thanks for joining!'];
	$session->setTempdata($tempdata, NULL, $expire);

.. note:: 如果省略了到期时间或将其设置为0，则将使用默认的生存时间值为300秒（或5分钟）。

要读取tempdata变量，同样可以通过 ``$_SESSION`` 超全局数组访问它::

	$_SESSION['item']

.. important:: 当通过键检索单个项时，``get()`` 方法将返回tempdata项。但是，从会话中获取所有用户数据时，它不会返回tempdata。

或者如果您想确定自己正在读取"tempdata"（而不是其他种类的数据），则也可以使用 ``getTempdata()`` 方法::

	$session->getTempdata('item');

当然，如果要检索所有现有的临时数据::

	$session->getTempdata();

.. note:: 如果找不到该项目，则 ``getTempdata()`` 方法返回NULL。

如果需要在到期前删除tempdata值，可以直接从 ``$_SESSION`` 数组中取消设置它::

	unset($_SESSION['item']);

但是，这不会删除使该特定项成为tempdata的标记（它将在下一个HTTP请求中失效），因此，如果您打算在同一请求中重用同一键，则需要使用 ``removeTempdata()``::

	$session->removeTempdata('item');

销毁 Session
====================

要清除当前会话（例如，在注销过程中），您可以简单地使用PHP的 `session_destroy() <https://www.php.net/session_destroy>`_ 函数或库的``destroy()`` 方法。两者将以完全相同的方式工作::

	session_destroy();

	// 或

	$session->destroy();

.. note:: 这必须是您在同一请求期间执行的与会话有关的最后一个操作。销毁会话后，所有会话数据（包括flashdata和tempdata）将被永久销毁，并且在同一请求期间功能将无法使用。	

您还可以使用 ``stop()`` 方法删除旧的session_id，销毁所有数据并销毁包含会话ID的cookie，使会话完全终止::

    $session->stop();

访问 session 元数据
==========================

在以前的CodeIgniter版本中，默认情况下，会话数据数组包括4个项目：'session_id', 'ip_address', 'user_agent', 'last_activity'。

这是由会话如何工作的细节所致，但现在在我们的新实现中不再需要。但是，您的应用程序可能会依赖这些值，因此下面是访问它们的替代方法:

  - session_id: ``session_id()``
  - ip_address: ``$_SERVER['REMOTE_ADDR']``
  - user_agent: ``$_SERVER['HTTP_USER_AGENT']`` (unused by sessions)
  - last_activity: Depends on the storage, no straightforward way. Sorry!

Session 首选项
*********************************************************************

通常，CodeIgniter可以使所有工作立即可用。但是，会话是任何应用程序中非常敏感的组件，因此必须进行一些仔细的配置。请花点时间考虑所有选项及其效果。

您将在 **app/Config/App.php** 文件中找到以下与会话相关的首选项:                                            

============================== ================ ============================= =======================================================================================================================
首选项                          默认              选项                           描述
============================== ================ ============================= =======================================================================================================================
**sessionDriver**              FileHandler      FileHandler                   要使用的session 驱动程序。(命名空间 **CodeIgniter\\Session\\Handlers**)                                                                              
                                                DatabaseHandler
                                                MemcachedHandler
                                                RedisHandler
                                                ArrayHandler
**sessionCookieName**          ci_session       仅 [A-Za-z\_-] 字符             要使用的会话cookie名称。
**sessionExpiration**          7200 (2小时)      以秒为单位时间 (整数)             您希望会话持续的秒数。如果您希望会话不过期（直到浏览器关闭），请将值设置为零：0                                                                               
**sessionSavePath**            NULL             None                           指定存储位置，取决于所使用的驱动程序。
**sessionMatchIP**             FALSE            TRUE/FALSE (布尔值)             读取会话cookie时是否验证用户的IP地址。请注意，某些ISP会动态更改IP，因此，如果您希望会话不过期，则可能会将其设置为FALSE。                                                                                                                                                                                                    
**sessionTimeToUpdate**        300              以秒为单位时间 (整数)             此选项控制会话类重新生成自身并创建新会话ID的频率。将其设置为0将禁用会话ID再生。                                                                                
**sessionRegenerateDestroy**   FALSE            TRUE/FALSE (布尔值)             自动重新生成会话ID时是否销毁与旧会话ID相关联的会话数据。设置为FALSE时，垃圾收集器稍后将删除数据。                                                                               
============================== ================ ============================= =======================================================================================================================

.. note:: 作为最后的选择，如果未配置上述任何项，则会话库将尝试获取PHP的与会话相关的INI设置以及旧式CI设置，例如“sess_expire_on_close”。但是，您永远不要依赖此行为，因为它可能导致意外的结果或将来被更改。请正确配置所有内容。

除了上述值之外，cookie和本机驱动程序还应用了 :doc:`IncomingRequest </incoming/incomingrequest>` 和 :doc:`Security <security>` 类共享的以下配置值:

================== =============== ===========================================================================
首选项               默认             描述
================== =============== ===========================================================================
**cookieDomain**   ''              会话适用的域
**cookiePath**     /               会话适用的路径
**cookieSecure**   FALSE           是否仅在加密（HTTPS）连接上创建会话cookie
================== =============== ===========================================================================

.. note:: “cookieHTTPOnly”设置对会话没有影响。出于安全原因，始终启用HttpOnly参数。此外，“cookiePrefix”设置被完全忽略。

Session 驱动程序
*********************************************************************

如前所述，Session库带有4个处理程序或存储引擎，您可以使用它们:

  - CodeIgniter\Session\Handlers\FileHandler
  - CodeIgniter\Session\Handlers\DatabaseHandler
  - CodeIgniter\Session\Handlers\MemcachedHandler
  - CodeIgniter\Session\Handlers\RedisHandler
  - CodeIgniter\Session\Handlers\ArrayHandler

默认情况下，在初始化会话时将使用 ``FileHandler`` 驱动程序，因为它是最安全的选择，并且有望在任何地方都可以使用（实际上每个环境都有一个文件系统）。

但是，可以选择通过 **app/Config/App.php** 文件中的 ``public $sessionDriver`` 行选择任何其他驱动程序。请记住，每个驱动程序都有不同的警告，因此在做出选择之前，一定要使自己熟悉（如下）。

.. note:: 在测试期间使用ArrayHandler并将其存储在PHP数组中，同时防止数据被持久保存。

FileHandler 驱动程序（默认）
==================================================================

“FileHandler”驱动程序使用您的文件系统来存储会话数据。

可以肯定地说，它的工作原理与PHP自己的默认会话实现完全相同，但是如果这对您来说是一个重要的细节，请记住，它实际上不是相同的代码，并且有一些限制（和优点）。

更具体地说，它不支持PHP的 `directory level and mode formats used in session.save_path <https://www.php.net/manual/en/session.configuration.php#ini.session.save-path>`_ ，并且为了安全起见，大多数选项都经过硬编码。相反， ``public $sessionSavePath`` 仅支持绝对路径。

您还应该知道的另一件事是，确保不要使用公共可读或共享目录来存储会话文件。确保 **只有您** 有权查看所选 **sessionSavePath** 目录的内容。否则，任何能够做到这一点的人都可以窃取当前的任何会话（也称为“会话固定”攻击）。

在类似UNIX的操作系统上，这通常是通过使用 `chmod` 命令在该目录上设置0700模式权限来实现的，该命令仅允许目录所有者对目录执行读取和写入操作。但是要小心，因为 **运行** 脚本的系统用户通常不是您自己的，而是'www-data'之类的东西，因此仅设置这些权限可能会破坏您的应用程序。

取而代之的是，您应该根据自己的环境执行类似的操作
::

	mkdir /<path to your application directory>/Writable/sessions/
	chmod 0700 /<path to your application directory>/Writable/sessions/
	chown www-data /<path to your application directory>/Writable/sessions/

Bonus Tip
--------------------------------------------------------

某些人可能会选择其他会话驱动程序，因为文件存储通常较慢。这只有一半是正确的。

一个非常基本的测试可能会让您相信SQL数据库更快，但是在99％的情况下，只有当您只有几个当前会话时，这才是正确的。随着会话数的增加和服务器负载的增加（这很重要），文件系统将始终胜过几乎所有的关系数据库设置。

此外，如果只考虑性能，则可能需要研究使用 `tmpfs <https://eddmann.com/posts/storing-php-sessions-file-caches-in-memory-using-tmpfs/>`_，（警告：外部资源），它可以使会话快速发展。

DatabaseHandler 驱动程序
==================================================================

“DatabaseHandler”驱动程序使用关系数据库（例如MySQL或PostgreSQL）来存储会话。这是许多用户中的一个流行选择，因为它使开发人员可以轻松访问应用程序中的会话数据-它只是数据库中的另一个表。

但是，必须满足一些条件:

  - 您不能使用持久连接。
  - 您不能在启用 **cacheOn** 设置的情况下使用连接。

为了使用“DatabaseHandler”会话驱动程序，您还必须创建我们已经提到的该表，然后将其设置为您的 ``$sessionSavePath`` 值。例如，如果您想使用“ci_sessions”作为表名，则可以这样做::

	public $sessionDriver   = 'CodeIgniter\Session\Handlers\DatabaseHandler';
	public $sessionSavePath = 'ci_sessions';

然后，当然要创建数据库表 ...

对于MySQL::

	CREATE TABLE IF NOT EXISTS `ci_sessions` (
		`id` varchar(128) NOT NULL,
		`ip_address` varchar(45) NOT NULL,
		`timestamp` int(10) unsigned DEFAULT 0 NOT NULL,
		`data` blob NOT NULL,
		KEY `ci_sessions_timestamp` (`timestamp`)
	);

对于PostgreSQL::

	CREATE TABLE "ci_sessions" (
		"id" varchar(128) NOT NULL,
		"ip_address" varchar(45) NOT NULL,
		"timestamp" bigint DEFAULT 0 NOT NULL,
		"data" text DEFAULT '' NOT NULL
	);

	CREATE INDEX "ci_sessions_timestamp" ON "ci_sessions" ("timestamp");

您还需要 **根据您的“sessionMatchIP”设置** 添加主键。以下示例在MySQL和PostgreSQL上均可使用::

	// 当 sessionMatchIP = TRUE 时
	ALTER TABLE ci_sessions ADD PRIMARY KEY (id, ip_address);

	// 当 sessionMatchIP = FALSE 时
	ALTER TABLE ci_sessions ADD PRIMARY KEY (id);

	// 删除先前创建的主键（在更改设置时使用）
	ALTER TABLE ci_sessions DROP PRIMARY KEY;
 
您可以通过在 **app\\Config\\App.php** 文件中添加新行并使用要使用的组名来选择要使用的数据库组::

  public $sessionDBGroup = 'groupName';

如果您不想手工完成所有这些操作，则可以使用cli中的 ``session:migration`` 命令为您生成一个迁移文件::

  > php spark session:migration
  > php spark migrate

该命令在生成代码时将考虑 **sessionSavePath** 和 **sessionMatchIP** 设置。

.. important:: 由于缺少其他平台上的建议性锁定机制，因此仅正式支持MySQL和PostgreSQL数据库。使用不带锁的会话会导致各种问题，尤其是在大量使用AJAX的情况下，我们不支持这种情况。如果遇到性能问题，请在处理完会话数据后使用 ``session_write_close()``。

RedisHandler 驱动程序
==================================================================

.. note:: 由于Redis没有公开锁定机制，因此该驱动程序的锁定由一个单独的值模拟，该值最多可保留300秒。

Redis是一种存储引擎，由于其高性能而通常用于缓存并广受欢迎，这可能也是您使用'RedisHandler'会话驱动程序的原因。

缺点是它不像关系数据库那样普遍存在，并且需要在系统上安装 `phpredis <https://github.com/phpredis/phpredis>`_ PHP扩展，并且没有与PHP捆绑在一起。很有可能，仅当您已经熟悉Redis并将其用于其他目的时，才使用RedisHandler驱动程序。

与“FileHandler”和“DatabaseHandler”驱动程序一样，您还必须通过 ``$sessionSavePath`` 设置配置会话的存储位置 。此处的格式有些不同，同时又很复杂。最好用 *phpredis* 扩展的README文件来解释，所以我们将简单地链接到它:

	https://github.com/phpredis/phpredis

.. warning:: CodeIgniter的会话库不使用实际的'redis' ``session.save_handler``。**仅** 注意上面链接中的路径格式。

但是，对于最常见的情况，一个简单的 ``host:port`` 配对就足够了::

	public $sessionDiver    = 'CodeIgniter\Session\Handlers\RedisHandler';
	public $sessionSavePath = 'tcp://localhost:6379';

MemcachedHandler 驱动程序
==================================================================

.. note:: 由于Memcached没有公开锁定机制，因此该驱动程序的锁定由一个单独的值模拟，该值最多保留300秒。

除了可能的可用性外，“MemcachedHandler”驱动程序的所有属性都与“RedisHandler”驱动程序非常相似，因为PHP的 `Memcached <https://www.php.net/memcached>`_ 扩展是通过PECL分发的，并且某些Linux发行版使其可以作为易于安装的软件包使用。

除此之外，对于Redis并没有任何故意的偏见，关于Memcached的说法没有多大不同-它也是一种流行的产品，通常用于缓存并以其速度著称。

但是，值得注意的是，Memcached给出的唯一保证是将值X设置为在Y秒后过期将导致在Y秒过去之后将其删除（但不一定要在该时间之前过期）。这种情况很少发生，但是应该考虑，因为这可能会导致会话丢失。

``$sessionSavePath`` 格式相当简单，仅仅是一个 ``host:port`` 对::

	public $sessionDriver   = 'CodeIgniter\Session\Handlers\MemcachedHandler';
	public $sessionSavePath = 'localhost:11211';

Bonus Tip
--------------------------------------------------------

还支持使用可选的权重参数作为第三个冒号（``:weight``）值的多服务器配置，但是我们必须注意，我们尚未测试这是否可靠。

如果要尝试使用此功能（后果自负），只需用逗号分隔多个服务器路径::

	// 与192.0.2.1相比，此处的本地主机将具有更高的优先级（5），权重为1。
	public $sessionSavePath = 'localhost:11211:5,192.0.2.1:11211:1';
