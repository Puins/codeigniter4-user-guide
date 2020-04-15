#################
文件系统辅助函数
#################

目录辅助函数文件包含协助使用目录的功能。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

加载辅助函数
===================

使用以下代码加载辅助函数::

	helper('filesystem');

可用函数
===================

该辅助函数有下列可用函数：

.. php:function:: directory_map($source_dir[, $directory_depth = 0[, $hidden = FALSE]])

	:param	string  $source_dir: 源目录的路径
	:param	int	    $directory_depth: 要遍历的目录深度（0 =完全递归，1 =当前目录，依此类推）
	:param	bool	$hidden: 是否包含隐藏目录
	:returns:	文件数组
	:rtype:	array

	示例::

		$map = directory_map('./mydirectory/');

	.. note:: 路径几乎总是相对于您的主index.php文件。

	目录中包含的子文件夹也将被映射。如果希望控制递归深度，则可以使用第二个参数（整数）来控制。深度为1只会映射顶层目录::

		$map = directory_map('./mydirectory/', 1);

	默认情况下，隐藏文件将不包含在返回的数组中。要覆盖此行为，可以将第三个参数设置为true（布尔值）::

		$map = directory_map('./mydirectory/', FALSE, TRUE);

	每个文件夹名称将是一个数组索引，而其包含的文件将被数字索引。这是一个典型数组的示例::

		Array (
			[libraries] => Array
				(
					[0] => benchmark.html
					[1] => config.html
					["database/"] => Array
						(
							[0] => query_builder.html
							[1] => binds.html
							[2] => configuration.html
							[3] => connecting.html
							[4] => examples.html
							[5] => fields.html
							[6] => index.html
							[7] => queries.html
						)
					[2] => email.html
					[3] => file_uploading.html
					[4] => image_lib.html
					[5] => input.html
					[6] => language.html
					[7] => loader.html
					[8] => pagination.html
					[9] => uri.html
				)

	如果未找到结果，则将返回一个空数组。

.. php:function:: write_file($path, $data[, $mode = 'wb'])

	:param	string	$path: 文件路径
	:param	string	$data: 要写入文件的数据
	:param	string	$mode: ``fopen()`` 模式
	:returns:	如果写入成功，则为TRUE；如果发生错误，则为FALSE
	:rtype:	bool

	将数据写入路径中指定的文件。如果文件不存在，则函数将创建它。示例::

		$data = 'Some file data';
		if ( ! write_file('./path/to/file.php', $data))
		{     
			echo 'Unable to write the file';
		}
		else
		{     
			echo 'File written!';
		}

	您可以选择通过第三个参数设置写入模式::

		write_file('./path/to/file.php', $data, 'r+');

	默认模式是 “wb”。 有关模式选项，请参见 `PHP用户指南 <https://www.php.net/manual/en/function.fopen.php>`_ 。

	.. note:: 为了使此功能将数据写入文件，必须将其权限设置为可写。如果该文件尚不存在，则包含该文件的目录必须是可写的。

	.. note:: 该路径是相对于您的主站点index.php文件的，而不是相对于您的控制器或视图文件的。CodeIgniter使用前端控制器，因此路径始终相对于主站点索引。

	.. note:: 该功能在写入文件时获得文件的排他锁。

.. php:function:: delete_files($path[, $del_dir = FALSE[, $htdocs = FALSE]])

	:param	string	$path: 目录路径
	:param	bool	$del_dir: 是否也删除目录
	:param	bool	$htdocs: 是否跳过删除 ``.htaccess`` 和 ``index`` 页文件
	:returns:	成功则为TRUE，如果发生错误则为FALSE
	:rtype:	bool

	删除提供的路径中包含的所有文件。示例::

		delete_files('./path/to/directory/');

	如果第二个参数设置为TRUE，则提供的根路径中包含的所有目录也会被删除。示例::

		delete_files('./path/to/directory/', TRUE);

	.. note:: 这些文件必须是可写的或由系统拥有者才能被删除。

.. php:function:: get_filenames($source_dir[, $include_path = FALSE])

	:param	string	$source_dir: 目录路径
	:param	bool|null	$include_path: 是否在文件名中包含路径；false为不包含，null为$source_dir的相对路径，true为全路径
	:param	bool	$hidden: 是否包含隐藏文件（以句点开头的文件）
	:returns:	文件名数组
	:rtype:	array

	将服务器路径作为输入，并返回一个包含的所有文件的名称数组。通过将第二个参数设置为TRUE，可以选择将文件路径添加到文件名。示例::

		$controllers = get_filenames(APPPATH.'controllers/');

.. php:function:: get_dir_file_info($source_dir, $top_level_only)

	:param	string	$source_dir: 目录路径
	:param	bool	$top_level_only: 是否仅查看指定目录（不包括子目录）
	:returns:	包含有关所提供目录内容信息的数组
	:rtype:	array

	读取指定的目录并构建一个包含文件名，文件大小，日期和权限的数组。仅当将第二个参数作为FALSE强制执行时，才读取指定路径中包含的子文件夹，因为这可能是一项繁重的操作。示例::

		$models_info = get_dir_file_info(APPPATH.'models/');

.. php:function:: get_file_info($file[, $returned_values = ['name', 'server_path', 'size', 'date']])

	:param	string	        $file: 文件路径 
	:param	array|string    $returned_values: 返回哪种类型（数组或逗号分隔的字符串）的信息
	:returns:	包含有关指定文件的信息的数组或失败时为FALSE
	:rtype:	array

	给定文件和路径，返回（可选）文件的名称，路径，大小和日期修改信息信息属性。第二个参数允许您显式声明要返回的信息。

	有效 ``$returned_values`` 选项包括：`name`, `size`, `date`, `readable`, `writeable`, `executable` and `fileperms`。

.. php:function:: symbolic_permissions($perms)

	:param	int	$perms: 权限
	:returns:	象征权限的字符串
	:rtype:	string

	获取数字权限（例如由返回 ``fileperms()``），并返回文件权限的标准符号表示法。

	::

		echo symbolic_permissions(fileperms('./index.php'));  // -rw-r--r--

.. php:function:: octal_permissions($perms)

	:param	int	$perms: 权限
	:returns:	八进制权限字符串
	:rtype:	string

	获取数字权限（例如由返回 ``fileperms()``），并返回文件权限的三字符八进制表示法。

	::

		echo octal_permissions(fileperms('./index.php')); // 644

.. php:function:: set_realpath($path[, $check_existence = FALSE])

	:param	string	$path: 路径
	:param	bool	$check_existence: 是否检查路径是否确实存在
	:returns:	绝对路径
	:rtype:	string

	此函数将返回没有符号链接或相对目录结构的服务器路径。如果无法解析路径，则可选的第二个参数将导致触发错误。

	示例::

		$file = '/etc/php5/apache2/php.ini';
		echo set_realpath($file); // 打印 '/etc/php5/apache2/php.ini'

		$non_existent_file = '/path/to/non-exist-file.txt';
		echo set_realpath($non_existent_file, TRUE);	// 显示错误，如同路径不能决定
		echo set_realpath($non_existent_file, FALSE);	// 打印 '/path/to/non-exist-file.txt'

		$directory = '/etc/php5';
		echo set_realpath($directory);	// 打印 '/etc/php5/'

		$non_existent_directory = '/path/to/nowhere';
		echo set_realpath($non_existent_directory, TRUE);	// 显示错误，如同路径不能决定
		echo set_realpath($non_existent_directory, FALSE);	// 打印 '/path/to/nowhere'
