########################
Image 处理类
########################

CodeIgniter的图像处理类允许你执行以下操作:

-  图像大小调整
-  创建缩略图
-  图像裁剪
-  图像旋转
-  图像水印

支持以下图像库: GD/GD2, 和 ImageMagick.

.. contents::
    :local:
    :depth: 2

初始化类
**********************

与CodeIgniter中的大多数其他类一样，图像类通过调用Services类在控制器中初始化::

	$image = \Config\Services::image();

您可以将想要使用的图像库的别名传递到Service函数中::

    $image = Config\Services::image('imagick');

可用的处理程序如下:

- gd        The GD/GD2 图像库
- imagick   The ImageMagick 库。

如果使用ImageMagick库，则必须在 **app/Config/Images.php** 中设置服务器上库的路径。

.. note:: ImageMagick处理程序不需要将imagick扩展名加载到服务器上。只要您的脚本可以访问该库并且可以在服务器上运行 ``exec()``，它就应该起作用。

处理图像
**********************

无论您要执行哪种处理（调整大小，裁剪，旋转或加水印），一般过程都是相同的。您将设置一些与要执行的操作相对应的首选项，然后调用可用的处理功能之一。例如，要创建图像缩略图，请执行以下操作::

	$image = \Config\Services::image()
		->withFile('/path/to/image/mypic.jpg')
		->fit(100, 100, 'center')
		->save('/path/to/image/mypic_thumb.jpg');

上面的代码告诉库在source_image文件夹中查找名为mypic.jpg的图像，然后使用GD2图像库从该图像中创建一个100 x 100像素的新图像，并将其保存到新文件（缩略图）。由于它使用fit（）方法，因此它将尝试根据所需的宽高比查找要裁剪的图像的最佳部分，然后裁剪并调整结果的大小。

保存之前，可以通过所需的多种可用方法来处理图像。原始图像保持不变，并且使用新图像并将其通过每种方法，并将结果应用于先前结果之上::

	$image = \Config\Services::image()
		->withFile('/path/to/image/mypic.jpg')
		->reorient()
		->rotate(90)
		->crop(100, 100, 0, 0)
		->save('/path/to/image/mypic_thumb.jpg');

此示例将拍摄相同的图像，并首先解决所有移动电话方向问题，将图像旋转90度，然后从左上角开始将结果裁剪为100x100像素的图像。结果将另存为缩略图。

.. note:: 为了允许图像类进行任何处理，包含图像文件的文件夹必须具有写权限。

.. note:: 对于某些操作，图像处理可能需要大量的服务器内存。如果在处理图像时遇到内存不足错误，则可能需要限制其最大大小和/或调整PHP内存限制。

图像质量
**********************

``save()`` 可以使用其他参数 ``$quality`` 来更改生成的图像质量。值的范围是0到100，其中90是框架默认值。此参数仅适用于JPEG图像，否则将被忽略::

	$image = \Config\Services::image()
		->withFile('/path/to/image/mypic.jpg')
		->save('/path/to/image/my_low_quality_pic.jpg', 10);

.. note:: 较高的质量将导致较大的文件大小。另请参阅 `imagejpeg 函数 <https://www.php.net/manual/en/function.imagejpeg.php>`_

处理方法
**********************

有七种可用的处理方法:

-  $image->crop()
-  $image->convert()
-  $image->fit()
-  $image->flatten()
-  $image->flip()
-  $image->resize()
-  $image->rotate()
-  $image->text()

这些方法返回类实例，因此可以将它们链接在一起，如上所示。如果他们失败了，他们将抛出一个 ``CodeIgniter\Images\ImageException`` 包含错误信息的消息。一个好的做法是捕获异常，并在失败时显示错误，如下所示::

	try {
        $image = \Config\Services::image()
            ->withFile('/path/to/image/mypic.jpg')
            ->fit(100, 100, 'center')
            ->save('/path/to/image/mypic_thumb.jpg');
	}
	catch (CodeIgniter\Images\ImageException $e)
	{
		echo $e->getMessage();
	}

裁剪图片
================

可以裁切图像，以便仅保留原始图像的一部分。在创建应与特定尺寸/纵横比匹配的缩略图时经常使用此功能。这可以通过 ``crop()`` 方法处理::

    crop(int $width = null, int $height = null, int $x = null, int $y = null, bool $maintainRatio = false, string $masterDim = 'auto')

- **$width** 是所需结果图像宽度，以像素为单位。
- **$height** 是生成的图像所需的高度，以像素为单位。
- **$x** 是从图像左侧开始裁切的像素数。
- **$y** 是从图像顶部开始裁剪的像素数。
- **$maintainRatio** 为true，将根据需要调整最终尺寸，以保持图像的原始纵横比。
- **$masterDim** 指定当 ``$maintainRatio`` 为true时应保持不变的尺寸。值可以是: 'width', 'height', 或 'auto'。

要从图像中心取出50x50像素的正方形，您需要首先计算适当的x和y偏移值::

    $info = \Config\Services::image('imagick')
		->withFile('/path/to/image/mypic.jpg')
		->getFile()
		->getProperties(true);

    $xOffset = ($info['width'] / 2) - 25;
    $yOffset = ($info['height'] / 2) - 25;

    \Config\Services::image('imagick')
		->withFile('/path/to/image/mypic.jpg')
		->crop(50, 50, $xOffset, $yOffset)
		->save('/path/to/new/image.jpg');

图像转换
================

``convert()`` 方法将库的内部指示器更改为所需的文件格式。这不会涉及实际的图像资源，但会指示 ``save()`` 要使用的格式::

	convert(int $imageType) 

- **$imageType** 是PHP的图像类型常量之一（例如，请参阅 `image-type 函数 <https://www.php.net/manual/zh/function.image-type-to-mime-type.php>`_）::

	\Config\Services::image()
		->withFile('/path/to/image/mypic.jpg')
		->convert(IMAGETYPE_PNG)
		->save('/path/to/new/image.png');

.. note:: ImageMagick已经以扩展名指示的类型保存文件，忽略 **$imageType**

裁剪图像
================

``fit()`` 方法旨在通过执行以下步骤来帮助简化以“智能”方式裁剪图像的一部分的操作:

- 确定要裁剪的原始图像的正确部分，以保持所需的宽高比。
- 裁剪原始图像。
- 调整为最终尺寸。

::

    fit(int $width, int $height = null, string $position = 'center')

- **$width** 是所需的图像最终宽度。
- **$height** 是所需的图像最终高度。
- **$position** 确定要裁剪的图像部分。允许的位置: 'top-left', 'top', 'top-right', 'left', 'center', 'right', 'bottom-left', 'bottom', 'bottom-right'。

这提供了一种始终保持纵横比的更简单的裁剪方式::

	\Config\Services::image('imagick')
		->withFile('/path/to/image/mypic.jpg')
		->fit(100, 150, 'left')
		->save('/path/to/new/image.jpg');

展平图像
================

``flatten()`` 方法旨在在透明图像（PNG）后面添加背景色并将RGBA像素转换为RGB像素:

- 从透明图像转换为jpgs时，请指定背景色。

::

    flatten(int $red = 255, int $green = 255, int $blue = 255)

- **$red** 是背景的红色值。
- **$green** 是背景的绿色值。
- **$blue** 是背景的蓝色值。

::

	\Config\Services::image('imagick')
		->withFile('/path/to/image/mypic.png')
		->flatten()
		->save('/path/to/new/image.jpg');

	\Config\Services::image('imagick')
		->withFile('/path/to/image/mypic.png')
		->flatten(25,25,112)
		->save('/path/to/new/image.jpg');

翻转图像
================

图像可以沿其水平或垂直轴翻转::

    flip(string $dir)

- **$dir** 指定要沿其翻转的轴。可以是 'vertical' 或 'horizontal'。

::

	\Config\Services::image('imagick')
		->withFile('/path/to/image/mypic.jpg')
		->flip('horizontal')
		->save('/path/to/new/image.jpg');

调整图像
================

可以使用resize()方法调整图像的大小以适合您需要的任何尺寸::

	resize(int $width, int $height, bool $maintainRatio = false, string $masterDim = 'auto')

- **$width** 是新图像的所需宽度（以像素为单位）
- **$height** 是新图像的期望高度（以像素为单位）
- **$maintainRatio** 确定是否拉伸图像以适合新尺寸，还是保持原始宽高比。
- **$masterDim** 指定在维持比率时应遵循哪个轴的尺寸。 'width', 'height'。

调整图像大小时，可以选择是保持原始图像的比例，还是拉伸/压缩新图像以适合所需的尺寸。如果 ``$maintainRatio`` 为true，则 ``$masterDim`` 指定的尺寸将保持不变，而其他尺寸将被更改以匹配原始图像的宽高比。

::

	\Config\Services::image('imagick')
		->withFile('/path/to/image/mypic.jpg')
		->resize(200, 100, true, 'height')
		->save('/path/to/new/image.jpg');

旋转图像
================

``rotate()`` 方法允许您以90度为增量旋转图像::

	rotate(float $angle)

- **$angle** 是旋转的度数。 '90', '180', '270'之一。

.. note:: 虽然 ``$angle`` 参数接受浮点数，但在处理过程中会将其转换为整数。如果该值是上面列出的三个值以外的任何其他值，它将抛出 ``CodeIgniter\Images\ImageException``。

添加文本水印
================

您可以使用text()方法非常简单地将文本水印叠加到图像上。这对于放置版权声明，摄影师姓名或仅将图像标记为预览非常有用，这样它们就不会在其他人的最终产品中使用。

::

	text(string $text, array $options = [])

第一个参数是您希望显示的文本字符串。第二个参数是一个选项数组，允许您指定应如何显示文本::

	\Config\Services::image('imagick')
		->withFile('/path/to/image/mypic.jpg')
		->text('Copyright 2017 My Photo Co', [
		    'color'      => '#fff',
		    'opacity'    => 0.5,
		    'withShadow' => true,
		    'hAlign'     => 'center',
		    'vAlign'     => 'bottom',
		    'fontSize'   => 20
		])
		->save('/path/to/new/image.jpg');

可以识别的可能选项如下:

- color         文本颜色（十六进制数字），即 #ff0000
- opacity		介于0和1之间的数字，表示文本的不透明度。
- withShadow	布尔值，是否显示阴影。
- shadowColor   阴影的颜色（十六进制数字）
- shadowOffset	偏移阴影的像素数量。适用于垂直和水平值。
- hAlign        水平对齐: left, center, right
- vAlign        垂直对齐: top, middle, bottom
- hOffset		x轴上的附加偏移量（以像素为单位）
- vOffset		y轴上的附加偏移量（以像素为单位）
- fontPath		您要使用的TTF字体的完整服务器路径。如果没有给出系统字体。
- fontSize		要使用的字体大小。当将GD处理程序与系统字体一起使用时，有效值在1-5之间。

.. note:: ImageMagick驱动程序无法识别fontPath的完整服务器路径。相反，只需提供您要使用的已安装系统字体之一的名称，即 Calibri。

