######################
数据库配置
######################

.. contents::
    :local:
    :depth: 2

CodeIgniter有一个配置文件，可让您存储数据库连接值（用户名，密码，数据库名等）。配置文件位于 ``app/Config/Database.php``。您也可以在 ``.env`` 文件中设置数据库连接值。请参阅下面的更多细节。

配置设置存储在类属性中，该属性是此原型的数组::

	public $default = [
		'DSN'	   => '',
		'hostname' => 'localhost',
		'username' => 'root',
		'password' => '',
		'database' => 'database_name',
		'DBDriver' => 'MySQLi',
		'DBPrefix' => '',
		'pConnect' => TRUE,
		'DBDebug'  => TRUE,
		'cacheOn'  => FALSE,
		'cacheDir' => '',
		'charset'  => 'utf8',
		'DBCollat' => 'utf8_general_ci',
		'swapPre'  => '',
		'encrypt'  => FALSE,
		'compress' => FALSE,
		'strictOn' => FALSE,
		'failover' => [],
	];

类属性的名称是连接名称，可以在连接时用于指定组名称。

某些数据库驱动程序（例如PDO，PostgreSQL，Oracle，ODBC）可能需要提供完整的DSN字符串。如果是这种情况，则应使用“DSN”配置设置，就像使用该驱动的原生PHP扩展一样，如下所示::

	// PDO
	$default['DSN'] = 'pgsql:host=localhost;port=5432;dbname=database_name';

	// Oracle
	$default['DSN'] = '//localhost/XE';

.. note:: 如果您没有为需要它的驱动程序指定DSN字符串，则CodeIgniter将尝试使用其余提供的设置来构建它。

.. note:: 如果提供了DSN字符串，但缺少某些有效设置（例如，数据库字符集），这些设置出现在其余配置字段中，则CodeIgniter会将其附加。

您还可以为主连接由于某种原因无法连接的情况指定故障转移。可以通过如下设置连接的故障转移来指定这些故障转移::

	$default['failover'] = [
			[
				'hostname' => 'localhost1',
				'username' => '',
				'password' => '',
				'database' => '',
				'DBDriver' => 'MySQLi',
				'DBPrefix' => '',
				'pConnect' => TRUE,
				'DBDebug'  => TRUE,
				'cacheOn'  => FALSE,
				'cacheDir' => '',
				'charset'  => 'utf8',
				'DBCollat' => 'utf8_general_ci',
				'swapPre'  => '',
				'encrypt'  => FALSE,
				'compress' => FALSE,
				'strictOn' => FALSE
			],
			[
				'hostname' => 'localhost2',
				'username' => '',
				'password' => '',
				'database' => '',
				'DBDriver' => 'MySQLi',
				'DBPrefix' => '',
				'pConnect' => TRUE,
				'DBDebug'  => TRUE,
				'cacheOn'  => FALSE,
				'cacheDir' => '',
				'charset'  => 'utf8',
				'DBCollat' => 'utf8_general_ci',
				'swapPre'  => '',
				'encrypt'  => FALSE,
				'compress' => FALSE,
				'strictOn' => FALSE
			]
		];

您可以指定任意数量的故障转移。

您可以选择存储多组连接值。例如，如果在一个安装中运行多个环境（development,production, test等），则可以为每个环境设置一个连接组，然后根据需要在组之间进行切换。例如，要设置“test”环境，您可以这样做::

	public $test = [
		'DSN'	   => '',
		'hostname' => 'localhost',
		'username' => 'root',
		'password' => '',
		'database' => 'database_name',
		'DBDriver' => 'MySQLi',
		'DBPrefix' => '',
		'pConnect' => TRUE,
		'DBDebug'  => TRUE,
		'cacheOn'  => FALSE,
		'cacheDir' => '',
		'charset'  => 'utf8',
		'DBCollat' => 'utf8_general_ci',
		'swapPre'  => '',
		'compress' => FALSE,
		'encrypt'  => FALSE,
		'strictOn' => FALSE,
		'failover' => []
	);

然后，要全局告诉系统使用该组，您可以在配置文件中设置此变量::

	$defaultGroup = 'test';

.. note:: 名称“test”是任意的。可以是您想要的任何东西。默认情况下，我们为主要连接使用了“默认”一词，但也可以将其重命名为与您的项目更相关的名称。

您可以修改配置文件以检测环境，并通过在类的构造函数中添加所需的逻辑，将 `defaultGroup` 值自动更新为正确的值::

	class Database
	{
	    public $development = [...];
	    public $test        = [...];
	    public $production  = [...];

		public function __construct()
		{
			$this->defaultGroup = ENVIRONMENT;
		}
	}

使用.env文件配置
--------------------------

您还可以将配置值与当前服务器的数据库设置一起保存在 ``.env`` 文件中。您只需要输入与默认组的配置设置中的值不同的值。值应为遵循此格式的名称，其中 ``default`` 为组名::

	database.default.username = 'root';
	database.default.password = '';
	database.default.database = 'ci4';

和其他所有的一样

值说明:
----------------------

======================  ===========================================================================================================
名称配置                  描述
======================  ===========================================================================================================
**dsn**			        DSN连接字符串（多合一配置序列）。
**hostname** 		    数据库服务器的主机名。通常这是 'localhost'。
**username**		用于连接数据库的用户名。
**password**		用于连接数据库的密码。
**database**		您要连接的数据库的名称。
**DBDriver**		数据库类型。例如：MySQLi，Postgre等。大小写必须与驱动程序名称匹配
**DBPrefix**		一个可选的表前缀，将在运行 :doc:`Query Builder <query_builder>` 查询时添加到表名中 。这允许多个CodeIgniter安装共享一个数据库。
**pConnect**		TRUE/FALSE (boolean) - 是否使用持久连接。
**DBDebug**		TRUE/FALSE (boolean) - 是否应显示数据库错误。
**cacheOn**		TRUE/FALSE (boolean) - 是否启用数据库查询缓存。
**cacheDir**		数据库查询缓存目录的绝对服务器路径。
**charset**	    	与数据库通信时使用的字符集。
**DBCollat**		与数据库通信时使用的字符排序规则

			.. note:: 仅在“ MySQLi”驱动程序中使用。

**swapPre**		应该与dbprefix交换的默认表前缀。这对于分布式应用程序很有用，在分布式应用程序中，您可能需要运行手动编写的查询，并且仍需要最终用户自定义前缀。
**schema**		数据库模式，默认为“ public”。由PostgreSQL和ODBC驱动程序使用。
**encrypt**		是否使用加密连接。

			  - 'sqlsrv' and 'pdo/sqlsrv' 驱动程序接受 TRUE/FALSE
			  - 'MySQLi' and 'pdo/mysql' 驱动程序接受具有以下选项的数组:

			    - 'ssl_key'    - 私钥文件的路径
			    - 'ssl_cert'   - 公钥证书文件的路径
			    - 'ssl_ca'     - 证书颁发机构文件的路径
			    - 'ssl_capath' - 包含PEM格式的受信任CA证书的目录的路径 
			    - 'ssl_cipher' - *允许* 用于加密的密码列表，以冒号(':')分隔
			    - 'ssl_verify' - TRUE/FALSE; 是否验证服务器证书 (仅 'MySQLi')

**compress**		是否使用客户端压缩（仅MySQL）。
**strictOn**		TRUE/FALSE (boolean) - 是否强制“严格模式”连接，有利于在开发应用程序时确保严格的SQL。
**port**		数据库端口号。要使用此值，您必须在数据库配置数组中添加一行。
			::

				$default['port'] = 5432;

======================  ===========================================================================================================

.. note:: 根据所使用的数据库平台（MySQL，PostgreSQL等），并不需要所有值。例如，当使用SQLite时，您将不需要提供用户名或密码，并且数据库名称将是数据库文件的路径。以上信息假定您正在使用MySQL。
