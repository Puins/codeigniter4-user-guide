******************
File 类
******************

CodeIgniter提供了一个文件类,它将提供 `SplFileInfo <https://www.php.net/manual/en/class.splfileinfo.php>`_ 类方法和一些额外的便利方法.这个类是 :doc:`uploaded files </libraries/uploaded_files>` 的基类和 :doc:`images </libraries/images>`。

.. contents::
    :local:
    :depth: 2

获取文件类实例
=======================

通过传递构造函数中文件的路径来创建新的文件实例。默认情况下,文件不需要存在。但是您可以传递一个附加参数 ``true``,以检查该文件是否存在,并在不存在的情况下抛出 ``FileNotFoundException()`` 的异常提示。

::

    $file = new \CodeIgniter\Files\File($path);

利用Spl
=======================

一旦你有一个实例,你就可以完成 ``SplFileInfo`` 类的全部功能,包括::

    // 获取文件的基本名称
    echo $file->getBasename();
    // 获取上次修改的时间
    echo $file->getMTime();
    // 获取真正的实际路径
    echo $file->getRealPath();
    // 获取文件权限
    echo $file->getPerms();

    // 向CSV中写入几行数据。
    if ($file->isWritable())
    {
        $csv = $file->openFile('w');

        foreach ($rows as $row)
        {
            $csv->fputcsv($row);
        }
    }

新功能
============

除了 SplFileInfo 类中的所有方法之外,还有一些新的方法.

**getRandomName()**

您可以生成一个加密安全的随机文件名,其中包含当前时间戳, ``getRandomName()``　方法在移动文件时重命名文件很有用::

	// 生成类似于: 1465965676_385e33f741.jpg
	$newName = $file->getRandomName();

**getSize()**

返回上传文件的大小（以字节为单位）。 可以将 'kb' 或 'mb' 作为第一个参数传入方法,将分别返回千字节和兆字节的结果::

	$bytes     = $file->getSize();      // 256901
	$kilobytes = $file->getSize('kb');  // 250.880
	$megabytes = $file->getSize('mb');  // 0.245

**getSizeByUnit()**

返回上传文件的默认大小（以字节为单位）。 可以将 'kb' 或 'mb' 作为第一个参数传入方法,将分别返回千字节和兆字节的结果::

	$bytes     = $file->getSizeByUnit();      // 256901
	$kilobytes = $file->getSizeByUnit('kb');  // 250.880
	$megabytes = $file->getSizeByUnit('mb');  // 0.245

**getMimeType()**

检索文件的媒体类型（MIME类型）。 尽可能在确定文件安全的前提下,使用该方法获取文件的类型::

	$type = $file->getMimeType();

	echo $type; // image/png

**guessExtension()**

使用 ``getMimeType()`` 方法确定文件扩展名时。如果文件类型未知,将返回 null。与简单地使用文件名提供的扩展名相比，这通常是一个更受信任的来源。用在 **app/Config/Mimes.php** 中的值来确定文件扩展名::

	// 返回 'jpg' (WITHOUT the period)
	$ext = $file->guessExtension();

移动文件
------------

每个文件可以使用 ``move()`` 方法移动到新的位置.指定文件的目录作为该方法的第一个参数::

	$file->move(WRITEPATH.'uploads');

默认情况下,使用原始文件名.您可以通过第二个参数重命名你要移动的文件::

	$newName = $file->getRandomName();
	$file->move(WRITEPATH.'uploads', $newName);

**move()** 方法为重新定位的文件返回一个新的文件实例，如果需要结果位置，则必须获取结果::

    $file = $file->move(WRITEPATH.'uploads');
