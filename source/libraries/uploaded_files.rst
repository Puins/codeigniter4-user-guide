***************************
使用文件上传类
***************************

与直接使用PHP的 ``$_FILES`` 数组相比，CodeIgniter使处理通过表单上传的文件更加简单和安全。这扩展了 :doc:`File 类 </libraries/files>` ，从而获得了该类的所有功能

.. note:: 这与早期版本的CodeIgniter中的File Uploading类不同。这为上传的文件提供了一些小功能的原始界面。

.. contents::
    :local:
    :depth: 2

===============
访问文件
===============

全部文件
----------

当您上传文件时，在PHP中可以通过 ``$_FILES`` 超级保留以本地方式访问它们。当处理一次上传的多个文件时，此数组存在一些主要缺点，并且存在许多开发人员未意识到的潜在安全漏洞。CodeIgniter通过在公共接口后面对文件的使用进行标准化来帮助解决这两种情况。

通过当前 ``IncomingRequest`` 实例访问文件。要检索与此请求上传的所有文件，请使用 ``getFiles()`` 。这将返回由以下 ``CodeIgniter\HTTP\Files\UploadedFile`` 实例表示的文件数组::

	$files = $this->request->getFiles();

当然，有很多种方式来为文件 input 标签命名，除了最简单的方法以外的任何命名方式都可能产生奇怪的结果。数组以您期望的方式返回。以最简单的用法，可以提交一个文件::

	<input type="file" name="avatar" />

这将返回一个简单的数组::

	[
		'avatar' => // UploadedFile instance
	]

如果您使用数组符号表示名称，则输入将类似于::

	<input type="file" name="my-form[details][avatar]" />

``getFiles()`` 返回的数组看起来像这样::

	[
		'my-form' => [
			'details' => [
				'avatar' => // UploadedFile instance
			]
		]
	]

在某些情况下，您可以指定要上传的文件数组::

	Upload an avatar: <input type="file" name="my-form[details][avatars][]" />
	Upload an avatar: <input type="file" name="my-form[details][avatars][]" />

在这种情况下，返回的文件数组将更像::

	[
		'my-form' => [
			'details' => [
				'avatar' => [
					0 => /* UploadedFile instance */,
					1 => /* UploadedFile instance */
			]
		]
	]

单文件
-----------

如果只需要访问单个文件，则可以使用 ``getFile()`` 来直接检索文件实例。这将返回的一个 ``CodeIgniter\HTTP\Files\UploadedFile`` 实例:

简单用法
^^^^^^^^^^^^^^

以最简单的用法，可以提交一个文件，例如::

	<input type="file" name="userfile" />

这将返回一个简单的文件实例，例如::

	$file = $this->request->getFile('userfile');

数组符号
^^^^^^^^^^^^^^

如果您使用数组符号表示名称，则输入将类似于::

	<input type="file" name="my-form[details][avatar]" />

要获取文件实例::

	$file = $this->request->getFile('my-form.details.avatar');

多文件
^^^^^^^^^^^^^^
::

    <input type="file" name="images[]" multiple />

在控制器中::

    if($imagefile = $this->request->getFiles())
    {
       foreach($imagefile['images'] as $img)
       {
          if ($img->isValid() && ! $img->hasMoved())
          {
               $newName = $img->getRandomName();
               $img->move(WRITEPATH.'uploads', $newName);
          }
       }
    }

循环中的 **images** 是表单中的字段名称

如果有多个同名文件，则可以使用 ``getFile()`` 或者分别检索每个文件::
在控制器中::

	$file1 = $this->request->getFile('images.0');
	$file2 = $this->request->getFile('images.1');

您可能会发现使用 ``getFileMultiple()`` 获取具有相同名称的一系列上传文件更容易::

	$files = $this->request->getFileMultiple('images');


另一个例子::

	Upload an avatar: <input type="file" name="my-form[details][avatars][]" />
	Upload an avatar: <input type="file" name="my-form[details][avatars][]" />

在控制器中::

	$file1 = $this->request->getFile('my-form.details.avatars.0');
	$file2 = $this->request->getFile('my-form.details.avatars.1');

.. note:: 使用 ``getFiles()`` 更合适

=====================
处理文件
=====================

检索UploadedFile实例后，您可以以安全的方式检索有关文件的信息，以及将文件移动到新位置。

验证文件
-------------

您可以通过调用 ``isValid()`` 方法来检查文件是否实际上是通过HTTP上传的，并且没有错误::

	if (! $file->isValid())
	{
		throw new RuntimeException($file->getErrorString().'('.$file->getError().')');
	}

如本例所示，如果文件有上传错误，则可以使用 ``getError()`` 和 ``getErrorString()`` 方法检索错误代码（整数）和错误消息。通过此方法可以发现以下错误:

* 文件大小超过了 upload_max_filesize 配置的值。
* 文件大小超过了表单定义的上传限制。
* 文件仅部分被上传。
* 没有文件被上传。
* 无法将文件写入磁盘。
* 无法上传文件：缺少临时目录。
* PHP扩展阻止了文件上传。

文件名
----------

**getName()**

您可以使用 ``getName()`` 方法检索客户端提供的原始文件名。这通常是客户端发送的文件名，并且不应被信任。如果文件已移动，将返回已移动文件的最终名称::

	$name = $file->getName();

**getClientName()**

始终返回客户端发送的上传文件的原始名称，即使文件已移动::

  $originalName = $file->getClientName();

**getTempName()**

要获取在上传期间创建的临时文件的完整路径，可以使用 ``getTempName()`` 方法::

	$tempfile = $file->getTempName();

其他文件信息
---------------

**getClientExtension()**
 
根据上传的文件名返回原始文件扩展名。这不是受信任的来源。对于受信任的版本，请使用 ``getExtension()``::

	$ext = $file->getClientExtension();

**getClientMimeType()**

返回客户端提供的文件的mime类型（mime类型）。这不是一个受信任的值。对于受信任的版本，请使用 ``getMimeType()``::

	$type = $file->getClientMimeType();

	echo $type; // image/png

移动文件
------------

可以使用适当命名的 ``move()`` 方法将每个文件移动到其新位置。这将将文件移动到的目录作为第一个参数::

	$file->move(WRITEPATH.'uploads');

默认情况下，使用原始文件名。您可以通过将新文件名作为第二个参数传递来指定它::

	$newName = $file->getRandomName();
	$file->move(WRITEPATH.'uploads', $newName);

删除文件后，将删除临时文件。您可以使用 ``hasMoved()`` 方法检查文件是否已经移动，该方法返回一个布尔值::

    if ($file->isValid() && ! $file->hasMoved())
    {
        $file->move($path);
    }

在某些情况下，带有HTTPException的移动上传的文件可能会失败:

- 文件已被移动
- 文件上传失败
- 文件移动操作失败（例如，权限不当）

存储文件
------------

可以使用适当命名的 ``store()`` 方法将每个文件移动到其新位置。

以最简单的用法，可以提交一个文件，例如::

	<input type="file" name="userfile" />

默认情况下，上传文件保存在 ``writable/uploads`` 目录中。将会创建YYYYMMDD文件夹和随机文件名。返回文件路径::

	$path = $this->request->getFile('userfile')->store();

您可以指定将文件移动到的目录作为第一个参数，通过将文件名作为第二个参数传递来指定新文件名::

	$path = $this->request->getFile('userfile')->store('head_img/', 'user_name.jpg');

在某些情况下，带有HTTPException的移动上传的文件可能会失败:

- 文件已被移动
- 文件上传失败
- 文件移动操作失败（例如，权限不当）
