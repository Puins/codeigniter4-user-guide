##################
Encryption 服务
##################

.. important:: 请勿使用此或任何其他加密库来存储密码！密码必须改为散列，而您应该通过PHP的 `密码散列扩展 <https://www.php.net/password>`_ 进行。	

加密服务提供双向对称（秘密密钥）数据加密。该服务将实例化和/或初始化加密处理程序以适合您的参数，如下所述。

加密服务处理程序必须实现CodeIgniter的简单的 ``EncrypterInterface``。使用适当的PHP密码扩展或第三方库可能需要在服务器上安装其他软件，并且/或者可能需要在PHP实例中显式启用。

当前支持以下PHP扩展:

- `OpenSSL <https://www.php.net/openssl>`_

这不是完整的密码解决方案。如果您需要更多功能（例如，公钥加密），建议您考虑直接使用OpenSSL或其他 `Cryptography 扩展 <https://www.php.net/manual/en/refs.crypto.php>`_ 之一。还有一种更全面的软件包，例如 `Halite <https://github.com/paragonie/halite>`_ （基于libsodium构建的OO软件包）。

.. note:: 自PHP 7.2起已弃用了对 ``MCrypt`` 扩展的支持。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

.. _usage:

****************************
使用 Encryption 库
****************************

像CodeIgniter中的所有服务一样，可以通过 ``Config\Services`` 以下方式加载它::

    $encrypter = \Config\Services::encrypter();

假设您已经设置了开始密钥（请参阅 :ref:`配置库` ），则加密和解密数据很简单-将适当的字符串传递给 ``encrypt()``
和/或 ``decrypt()`` 方法::

	$plainText = 'This is a plain-text message!';
	$ciphertext = $encrypter->encrypt($plainText);

	// 输出: 这是一条纯文本消息！
	echo $encrypter->decrypt($ciphertext);

就是这样！加密库将为加密整个过程提供开箱即用的加密安全性。您无需担心。

.. _配置库:

配置库
=======================

上面的示例使用中的配置设置 ``app/Config/Encryption.php``。

只有两种设置:

======== ===============================================
选项      可能的值（括号中为默认值）
======== ===============================================
key      加密密钥启动器
driver   首选处理程序 (OpenSSL)
======== ===============================================

您可以通过将自己的配置对象传递给 ``Services`` 调用来替换配置文件的设置。该 ``$config`` 变量必须是 `Config\\Encryption` 类的实例，或者是扩展 `CodeIgniter\\Config\\BaseConfig` 的对象。
::

    $config         = new Config\Encryption();
    $config->key    = 'aBigsecret_ofAtleast32Characters';
    $config->driver = 'OpenSSL';

    $encrypter = \Config\Services::encrypter($config);


默认行为
================

默认情况下，加密库使用OpenSSL处理程序。该处理程序使用AES-256-CTR算法，您配置的密钥和SHA512 HMAC身份验证进行加密。

设置 Encryption 密钥
===========================

您的加密密钥必须在使用的加密算法允许的范围内。对于AES-256，则为256位或32个字节（字符）长。

密钥应该尽可能地随机，并且不能是常规文本字符串，也不能是哈希函数的输出等。要创建正确的密钥，可以使用加密库的 ``createKey()`` 方法。
::

	// $key将被分配一个32字节（256位）随机密钥
	$key = Encryption::createKey(32);

密钥可以存储在 *app/Config/Encryption.php* 中，或者您可以设计自己的存储机制，并在加密/解密时动态传递密钥。

要将密钥保存到 *app/Config/Encryption.php* 中，请打开文件并进行以下设置::

	public $key = 'YOUR KEY';

对密钥或结果编码
------------------------

你会注意到 ``createKey()`` 方法会输出二进制数据，这是很难解决的（即复制粘贴可能会损坏）， 所以你可以使用 bin2hex() 、 hex2bin() 或编码的 Base64 处理以更友好的密钥。例如::

	// 获取一个十六进制形式的密钥
	$encoded = bin2hex(Encryption::createKey(32));

	// 使用 hex2bin() 将相同的值放入配置中，
	// 这样它仍会以二进制形式传递给库配置
	$key = hex2bin(<your hex-encoded key>);

你可能会发现相同的技术对于加密结果也是有效的::

	// 加密一些文本并生成密文
	$encoded = base64_encode($encrypter->encrypt($plaintext));

加密处理程序说明
========================

OpenSSL 说明
-------------

长期以来，`OpenSSL <https://www.php.net/openssl>`_ 扩展一直是PHP的标准部分。

CodeIgniter的OpenSSL处理程序使用AES-256-CTR算法。

你的配置提供的 *key* 用于派生另外两个密钥，一个用于加密，另一个用于身份验证。 这是通过一种叫做 `基于HMAC的密钥派生函数 <https://en.wikipedia.org/wiki/HKDF>`_ （HKDF）的技术来实现的。

消息长度
==============

加密的字符串通常长于原始的纯文本字符串（取决于密码）。

这受密码算法本身，加在密码文本之前的初始化向量（IV）以及也加在前面的HMAC身份验证消息的影响。此外，加密的消息也经过Base64编码，因此无论使用什么字符集，它都可以安全地存储和传输。

选择数据存储机制时，请记住此信息。例如，Cookie只能保存4K信息。

直接使用加密服务
=====================================

除了使用 :ref:`usage` 中 ``Services`` 那样的方法外，你还可以直接创建“加密器”，或更改现有实例的设置。
::

    // 创建一个加密器实例
    $encryption = new \Encryption\Encryption();

    // 用不同的设置重新配置实例
    $encrypter = $encryption->initialize($config);

请记住， ``$config`` 必须是 `Config\Encryption` 类或扩展 `CodeIgniter\Config\BaseConfig` 的对象的实例。


***************
类参考
***************

.. php:class:: CodeIgniter\\Encryption\\Encryption

	.. php:staticmethod:: createKey($length)

		:param	int	$length: 输出长度
		:returns:	具有指定长度的伪随机密码密钥，失败则为FALSE
		:rtype:	string

		通过从操作系统的源（即 /dev/urandom）获取随机数据来创建加密密钥。

	.. php:method:: initialize($config)

		:param	BaseConfig	$config: 配置参数
		:returns:	CodeIgniter\\Encryption\\EncrypterInterface 实例
		:rtype:	CodeIgniter\\Encryption\\EncrypterInterface
		:throws:	CodeIgniter\\Encryption\\Exceptions\\EncryptionException

		初始化（或配置）库以使用不同的设置。

		示例::

			$encrypter = $encryption->initialize(['cipher' => '3des']);

		请参阅 :ref:`配置库` 部分以获取详细信息。

.. php:interface:: CodeIgniter\\Encryption\\EncrypterInterface

	.. php:method:: encrypt($data, $params = null)

		:param	string	$data: 要加密的数据
		:param		$params: 配置参数（键）
		:returns:	加密数据或失败时为FALSE
		:rtype:	string
		:throws:	CodeIgniter\\Encryption\\Exceptions\\EncryptionException

		加密输入数据并返回其密文。

		如果将参数作为第二个参数传递，则如果 ``key`` 元素 ``$params`` 是数组，则该元素将用作此操作的起始键；或者起始键可以作为字符串传递。

		示例::

			$ciphertext = $encrypter->encrypt('My secret message');
			$ciphertext = $encrypter->encrypt('My secret message', ['key' => 'New secret key']);
			$ciphertext = $encrypter->encrypt('My secret message', 'New secret key');

	.. php:method:: decrypt($data, $params = null)

		:param	string	$data: 要解密的数据
		:param		$params: 配置参数（键）
		:returns:	解密数据或失败时为FALSE
		:rtype:	string
		:throws:	CodeIgniter\\Encryption\\Exceptions\\EncryptionException

		解密输入数据并以纯文本形式返回。

		如果将参数作为第二个参数传递，则如果 ``key`` 元素 ``$params`` 是数组，则该元素将用作此操作的起始键；或者起始键可以作为字符串传递。

		示例::

			echo $encrypter->decrypt($ciphertext);
			echo $encrypter->decrypt($ciphertext, ['key' => 'New secret key']);
			echo $encrypter->decrypt($ciphertext, 'New secret key');
