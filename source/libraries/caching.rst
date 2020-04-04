##############
Caching 驱动器
##############

CodeIgniter 提供了几种最常用的快速缓存的封装，除了基于文件的缓存， 其他的缓存都需要对服务器进行特殊的配置，如果配置不正确，将会抛出 一个致命错误异常（Fatal Exception）。

.. contents::
    :local:
    :depth: 2

*************
用法示例
*************

以下示例代码展示控制器中的常见使用模式。

::

	if ( ! $foo = cache('foo'))
	{
		echo 'Saving to the cache!<br />';
		$foo = 'foobarbaz!';

		// 保存到缓存中5分钟
		cache()->save('foo', $foo, 300);
	}

	echo $foo;

你可以通过 ``Services`` 类直接获取缓存引擎的实例::

    $cache = \Config\Services::cache();

    $foo = $cache->get('foo');

=====================
配置缓存
=====================

缓存引擎的所有配置都在 **app/Config/Cache.php** 文件中。在该文件中，以下项目可用。

**$handler**

**$handler** 处理器是启动引擎时应用作主处理程序。可用的名称有： ``dummy``, ``file``, ``memcached``, ``redis``, ``wincache``。

**$backupHandler**

在第一选择 **$hanlder** 不可用的情况下，这是要加载的下一个缓存处理程序。这通常是 **file** 处理程序，因为文件系统始终可用，但可能不适合更复杂的多服务器设置。

**$prefix**

如果您有多个应用程序使用相同的缓存存储，则可以在此处添加一个所有键名的自定义前缀。

**$path**

 ``file`` 处理程序使用它来显示应该将缓存文件保存到哪里。

**$memcached**

使用 ``Memcache(d)`` 处理程序时将使用的一系列服务器。

**$redis**

使用 ``Redis`` 处理程序时要使用的Redis服务器的设置。

***************
类参考
***************

.. php:method:: ⠀isSupported()

	:returns:	支持则为TRUE， 不支持则为FALSE
	:rtype:	bool

.. php:method:: ⠀get($key)

	:param	string	$key: 缓存项名称
	:returns:	缓存项值，如果找不到则为NULL
	:rtype:	mixed

	此方法将尝试从缓存存储中获取项目。如果项目不存在，方法将返回空值。

	示例::

		$foo = $cache->get('my_cached_item');

.. php:method:: ⠀save($key, $data[, $ttl = 60[, $raw = FALSE]])

	:param	string	$key: 缓存项名称
	:param	mixed	$data: 要保存的数据
	:param	int	$ttl: 生存时间，以秒为单位 (默认 60)
	:param	bool	$raw: 是否存储原始值
	:returns:	TRUE 为成功，FALSE则失败
	:rtype:	string

	此方法将会将项目保存到缓存存储。如果保存失败，该方法将返回FALSE。

	示例::

		$cache->save('cache_item_id', 'data_to_cache');

.. note:: 该 ``$raw`` 参数仅由 Memcache 使用，以便允许使用 ``increment()`` 和 ``decrement()``。

.. php:method:: ⠀delete($key)

	:param	string	$key: 缓存项名称
	:returns:	TRUE 为成功，FALSE则失败
	:rtype:	bool

	此方法将从缓存存储中删除特定项目。如果删除项目失败，该方法将返回FALSE。

	示例::

		$cache->delete('cache_item_id');

.. php:method:: ⠀increment($key[, $offset = 1])

	:param	string	$key: 缓存 ID
	:param	int	$offset: 要添加的　Step/value
	:returns:	新值为成功, FALSE 为失败
   	:rtype:	mixed

	执行原始存储值的原子递增。

	示例::

		// 'iterator' 原值为2

		$cache->increment('iterator'); // 'iterator' 现在3

		$cache->increment('iterator', 3); // 'iterator' 现在6

.. php:method:: ⠀decrement($key[, $offset = 1])

	:param	string	$key: 缓存 ID
	:param	int	$offset:  减少的　Step/value
	:returns:	新值为成功, FALSE 为失败
	:rtype:	mixed

	执行原始存储值的原子递减。

	示例::

		// 'iterator' 原值6

		$cache->decrement('iterator'); // 'iterator' 现在5

		$cache->decrement('iterator', 2); // 'iterator' 现在3

.. php:method:: ⠀clean()

	:returns:	TRUE 为成功, FALSE 则失败
	:rtype:	bool

	此方法将“清除”整个缓存。 如果删除缓存文件失败，该方法将返回FALSE。

	示例::

			$cache->clean();

.. php:method:: ⠀cache_info()

	:returns:	有关整个缓存数据库的信息
	:rtype:	mixed

	此方法将返回整个缓存中的信息。

	示例::

		var_dump($cache->cache_info());

.. note:: 返回的信息和数据的结构取决于正在使用的适配器。

.. php:method:: ⠀getMetadata($key)

	:param	string	$key: 缓存项名称
	:returns:	缓存项目的元数据
	:rtype:	mixed

	此方法将返回指定项目的详细缓存信息。

	示例::

		var_dump($cache->getMetadata('my_cached_item'));

.. note:: 返回的信息和数据的结构取决于正在使用的适配器。

*******
驱动器
*******

==================
基于文件的缓存
==================

和输出类的缓存不同的是，基于文件的缓存支持只缓存视图的某一部分。使用这个缓存时要注意， 确保对你的应用程序进行基准测试，因为当磁盘 I/O 频繁时可能对缓存有负面影响。这要求可写缓存目录是真正可写的（0777）。

=================
Memcached 缓存
=================

Memcached服务器可以在缓存配置文件中指定。 可用参数有::

	public $memcached = [
		'host'   => '127.0.0.1',
		'port'   => 11211,
		'weight' => 1,
		'raw'    => false,
	];

关于 Memcached 的更多信息，请参阅 `https://www.php.net/memcached <https://www.php.net/memcached>`_。

================
WinCache 缓存
================

在Windows下，您还可以使用 WinCache 驱动器。

关于 WinCache 的更多信息，请参阅 `https://www.php.net/wincache <https://www.php.net/wincache>`_。

=============
Redis 缓存
=============

Redis 是一个在内存中以键值形式存储数据的缓存，使用 LRU（最近最少使用算法）缓存模式， 要使用它，你需要先安装 `Redis 服务器 and phpredis 扩展 <https://github.com/phpredis/phpredis>`_。

连接到Redis服务器的配置选项存储在缓存配置文件中。可用参数有::

	public $redis = [
		'host'     => '127.0.0.1',
		'password' => null,
		'port'     => 6379,
		'timeout'  => 0,
		'database' => 0,
	];
	
有关Redis的更多信息，请参阅 `https://redis.io <https://redis.io>`_。

======================
Dummy（虚拟） 缓存
======================

这是一个永远不会命中的缓存，它不存储数据，但是它允许在你当前环境下缓存不被支持时， 仍然保留使用缓存的代码。
