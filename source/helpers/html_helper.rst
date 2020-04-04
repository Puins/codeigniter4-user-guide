#################
HTML 辅助函数
#################

HTML辅助函数文件包含协助使用HTML的功能。

.. contents::
    :local:

.. raw:: html

    <div class="custom-index container"></div>

加载此辅助函数
===================

使用以下代码加载此辅助函数::

    helper('html');

可用功能
===================

以下功能可用：

.. php:function:: img([$src = ''[, $indexPage = false[, $attributes = '']]])

    :param  mixed  $src:        Image 源数据
    :param  bool    $indexPage:  是否将$src视为路由的URI字符串
    :param  mixed   $attributes: HTML属性
    :returns:   HTML image 标记
    :rtype: string

    使您可以创建HTML <img />标记。第一个参数包含Image源。示例::

        echo img('images/picture.jpg');
        // <img src="http://site.com/images/picture.jpg" />

    有一个可选的第二个参数，它是一个true/false值，它指定 *src* 是否应将指定的页面 ``$config['indexPage']`` 添加到它创建的地址中。大概是使用媒体控制器的情况::

        echo img('images/picture.jpg', true);
        // <img src="http://site.com/index.php/images/picture.jpg" alt="" />

    此外，可以将关联数组作为第一个参数传递，以完全控制所有属性和值。如果未提供 *alt* 属性，则CodeIgniter将生成一个空字符串。

    示例::

        $imageProperties = [
            'src'    => 'images/picture.jpg',
            'alt'    => 'Me, demonstrating how to eat 4 slices of pizza at one time',
            'class'  => 'post_images',
            'width'  => '200',
            'height' => '200',
            'title'  => 'That was quite a night',
            'rel'    => 'lightbox'
        ];

        img($imageProperties);
        // <img src="http://site.com/index.php/images/picture.jpg" alt="Me, demonstrating how to eat 4 slices of pizza at one time" class="post_images" width="200" height="200" title="That was quite a night" rel="lightbox" />

.. php:function:: link_tag([$href = ''[, $rel = 'stylesheet'[, $type = 'text/css'[, $title = ''[, $media = ''[, $indexPage = false]]]]]])

    :param  string  $href:      链接文件的源
    :param  string  $rel:       关系类型
    :param  string  $type:      相关文档的类型
    :param  string  $title:     链接标题
    :param  string  $media:     媒体类型
    :param  bool    $indexPage: 是否将$src视为路由的URI字符串
    :returns:   HTML链接标记
    :rtype: string

    使您可以创建HTML <link />标记。这对于样式表链接以及其他链接很有用。参数是 *href*，具有可选的 *rel*,
    *type*, *title*, *media* 和 *indexPage*。

    *indexPage* 是一个布尔值，它指定是否应将 *href* 指定的页面 ``$config['indexPage']`` 添加到 *href* 创建的地址中。

    示例::

        echo link_tag('css/mystyles.css');
        // <link href="http://site.com/css/mystyles.css" rel="stylesheet" type="text/css" />

    进一步的例子::

        echo link_tag('favicon.ico', 'shortcut icon', 'image/ico');
        // <link href="http://site.com/favicon.ico" rel="shortcut icon" type="image/ico" />

        echo link_tag('feed', 'alternate', 'application/rss+xml', 'My RSS Feed');
        // <link href="http://site.com/feed" rel="alternate" type="application/rss+xml" title="My RSS Feed" />

    或者，可以将关联数组传递给 ``link_tag()`` 函数，以完全控制所有属性和值::

        $link = [
            'href'  => 'css/printer.css',
            'rel'   => 'stylesheet',
            'type'  => 'text/css',
            'media' => 'print'
        ];

        echo link_tag($link);
        // <link href="http://site.com/css/printer.css" rel="stylesheet" type="text/css" media="print" />

.. php:function:: script_tag([$src = ''[, $indexPage = false]])

    :param  mixed  $src: JavaScript文件的源名称
    :param  bool    $indexPage: 是否将$src视为路由的URI字符串
    :returns:   HTML script 标记
    :rtype: string

    使您可以创建HTML <script> </script>标记。参数是 *src*，带有可选的 *indexPage*。

    *indexPage* 是一个布尔值，它指定 *src* 是否应将指定的页面 ``$config['indexPage']`` 添加到它创建的地址中。

    示例::

        echo script_tag('js/mystyles.js');
        // <script src="http://site.com/js/mystyles.js" type="text/javascript"></script>

    或者，可以将关联数组传递给 ``script_tag()`` 函数，以完全控制所有属性和值::    

        $script = ['src'  => 'js/printer.js'];

        echo script_tag($script);
        // <script src="http://site.com/js/printer.js" type="text/javascript"></script>

.. php:function:: ul($list[, $attributes = ''])

    :param  array   $list: 列出条目
    :param  array   $attributes: HTML属性
    :returns:   HTML格式的无序列表
    :rtype: string

    允许您从简单或多维数组生成无序HTML列表。示例::

        $list = [
            'red',
            'blue',
            'green',
            'yellow'
        ];

        $attributes = [
            'class' => 'boldlist',
            'id'    => 'mylist'
        ];

        echo ul($list, $attributes);

    上面的代码将产生以下结果:

    .. code-block:: html

        <ul class="boldlist" id="mylist">
            <li>red</li>
            <li>blue</li>
            <li>green</li>
            <li>yellow</li>
        </ul>

    这是一个使用多维数组的更复杂的示例::

        $attributes = [
            'class' => 'boldlist',
            'id'    => 'mylist'
        ];

        $list = [
            'colors' => [
                'red',
                'blue',
                'green'
            ],
            'shapes' => [
                'round',
                'square',
                'circles' => [
                    'ellipse',
                    'oval',
                    'sphere'
                ]
            ],
            'moods'  => [
                'happy',
                'upset'   => [
                    'defeated' => [
                        'dejected',
                        'disheartened',
                        'depressed'
                    ],
                    'annoyed',
                    'cross',
                    'angry'
                ]
            ]
        ];

        echo ul($list, $attributes);

    上面的代码将产生以下结果:

    .. code-block:: html

        <ul class="boldlist" id="mylist">
            <li>colors
                <ul>
                    <li>red</li>
                    <li>blue</li>
                    <li>green</li>
                </ul>
            </li>
            <li>shapes
                <ul>
                    <li>round</li>
                    <li>suare</li>
                    <li>circles
                        <ul>
                            <li>elipse</li>
                            <li>oval</li>
                            <li>sphere</li>
                        </ul>
                    </li>
                </ul>
            </li>
            <li>moods
                <ul>
                    <li>happy</li>
                    <li>upset
                        <ul>
                            <li>defeated
                                <ul>
                                    <li>dejected</li>
                                    <li>disheartened</li>
                                    <li>depressed</li>
                                </ul>
                            </li>
                            <li>annoyed</li>
                            <li>cross</li>
                            <li>angry</li>
                        </ul>
                    </li>
                </ul>
            </li>
        </ul>

.. php:function:: ol($list, $attributes = '')

    :param  array   $list: 列出条目
    :param  array   $attributes: HTML属性
    :returns:   HTML格式的有序列表
    :rtype: string

    与 :php:func:`ul()` 相同，只是它为有序列表生成<ol>标记而不是<ul>。

.. php:function:: video($src[, $unsupportedMessage = ''[, $attributes = ''[, $tracks = [][, $indexPage = false]]]])

    :param  mixed   $src:                源字符串或源数组。查看 :php:func:`source()` 函数
    :param  string  $unsupportedMessage: 浏览器不支持媒体标记时显示的消息
    :param  string  $attributes:         HTML属性
    :param  array   $tracks:             在数组中使用track函数。查看 :php:func:`track()` 函数
    :param  bool    $indexPage:
    :returns:                            HTML格式的视频元素
    :rtype: string

    允许您从简单数组或源数组生成HTML视频元素。示例::

        $tracks =
        [
            track('subtitles_no.vtt', 'subtitles', 'no', 'Norwegian No'),
            track('subtitles_yes.vtt', 'subtitles', 'yes', 'Norwegian Yes')
        ];

        echo video('test.mp4', 'Your browser does not support the video tag.', 'controls');

        echo video
        (
            'http://www.codeigniter.com/test.mp4',
            'Your browser does not support the video tag.',
            'controls',
            $tracks
        );

        echo video
        (
            [
              source('movie.mp4', 'video/mp4', 'class="test"'),
              source('movie.ogg', 'video/ogg'),
              source('movie.mov', 'video/quicktime'),
              source('movie.ogv', 'video/ogv; codecs=dirac, speex')
            ],
            'Your browser does not support the video tag.',
            'class="test" controls',
            $tracks
         );

    上面的代码将产生以下结果:

    .. code-block:: html

        <video src="test.mp4" controls>
          Your browser does not support the video tag.
        </video>

        <video src="http://www.codeigniter.com/test.mp4" controls>
          <track src="subtitles_no.vtt" kind="subtitles" srclang="no" label="Norwegian No" />
          <track src="subtitles_yes.vtt" kind="subtitles" srclang="yes" label="Norwegian Yes" />
          Your browser does not support the video tag.
        </video>

        <video class="test" controls>
          <source src="movie.mp4" type="video/mp4" class="test" />
          <source src="movie.ogg" type="video/ogg" />
          <source src="movie.mov" type="video/quicktime" />
          <source src="movie.ogv" type="video/ogv; codecs=dirac, speex" />
          <track src="subtitles_no.vtt" kind="subtitles" srclang="no" label="Norwegian No" />
          <track src="subtitles_yes.vtt" kind="subtitles" srclang="yes" label="Norwegian Yes" />
          Your browser does not support the video tag.
        </video>

.. php:function:: audio($src[, $unsupportedMessage = ''[, $attributes = ''[, $tracks = [][, $indexPage = false]]]])

    :param  mixed   $src:                源字符串或源数组。 :php:func:`source()` 函数
    :param  string  $unsupportedMessage: 浏览器不支持媒体标记时显示的消息
    :param  string  $attributes:         HTML属性
    :param  array   $tracks:             在数组中使用track函数。 :php:func:`track()` 函数
    :param  bool    $indexPage:
    :returns:                            HTML格式的音频元素
    :rtype: string

    与 :php:func:`video()` 相同，只是产生标记是<audio>而不是<video>。

.. php:function:: source($src = ''[, $type = false[, $attributes = '']])

    :param  string  $src:        媒体资源的路径
    :param  bool    $type:       具有可选编解码器参数的资源的MIME类型
    :param  array   $attributes: HTML属性
    :returns:   HTML source 标记
    :rtype: string

    使您可以创建HTML <source />标记。第一个参数包含 ``source`` 源。示例::

        echo source('movie.mp4', 'video/mp4', 'class="test"');
        // <source src="movie.mp4" type="video/mp4" class="test" />

.. php:function:: embed($src = ''[, $type = false[, $attributes = ''[, $indexPage = false]]])

    :param  string  $src:        嵌入资源的路径
    :param  bool    $type:       MIME类型
    :param  array   $attributes: HTML属性
    :param  bool    $indexPage:
    :returns:   HTML embed 标记
    :rtype: string

    使您可以创建HTML <embed />标记。第一个参数包含 ``embed`` 源。 示例::

        echo embed('movie.mov', 'video/quicktime', 'class="test"');
        // <embed src="movie.mov" type="video/quicktime" class="test"/>

.. php:function:: object($data = ''[, $type = false[, $attributes = '']])

    :param  string  $data:       资源URL
    :param  bool    $type:       资源的内容类型
    :param  array   $attributes: HTML属性
    :param  array   $params:     在数组内部使用param函数。查看 :php:func:`param()` 函数
    :returns:   HTML object 标记
    :rtype: string

    使您可以创建HTML <object />标记。第一个参数包含 ``object`` 数据。示例::

        echo object('movie.swf', 'application/x-shockwave-flash', 'class="test"');

        echo object
        (
            'movie.swf',
            'application/x-shockwave-flash',
            'class="test"',
            [
                param('foo', 'bar', 'ref', 'class="test"'),
                param('hello', 'world', 'ref', 'class="test"')
            ]
        );

    上面的代码将产生以下结果:

    .. code-block:: html

        <object data="movie.swf" class="test"></object>

        <object data="movie.swf" class="test">
          <param name="foo" type="ref" value="bar" class="test" />
          <param name="hello" type="ref" value="world" class="test" />
        </object>

.. php:function:: param($name = ''[, $type = false[, $attributes = '']])

    :param  string  $name:       参数的名称
    :param  string  $value:      参数的值
    :param  array   $attributes: HTML属性
    :returns:   HTML param 标记
    :rtype: string

    使您可以创建HTML <param />标记。第一个参数包含 ``param`` 源。示例::

        echo param('movie.mov', 'video/quicktime', 'class="test"');
        // <param src="movie.mov" type="video/quicktime" class="test"/>

.. php:function:: track($name = ''[, $type = false[, $attributes = '']])

    :param  string  $name:       参数的名称
    :param  string  $value:      参数的值
    :param  array   $attributes: HTML属性
    :returns:   HTML track 标记
    :rtype: string

    生成track元素以指定定时轨道。曲目以WebVTT格式格式化。 示例::

        echo track('subtitles_no.vtt', 'subtitles', 'no', 'Norwegian No');
        // <track src="subtitles_no.vtt" kind="subtitles" srclang="no" label="Norwegian No" />

.. php:function:: doctype([$type = 'html5'])

    :param  string  $type: 文档类型名称
    :returns:   HTML DocType 标记
    :rtype: string

    帮助您生成文档类型声明或DTD。默认情况下使用HTML 5，但是有许多文档类型可用。

    示例::

        echo doctype();
        // <!DOCTYPE html>

        echo doctype('html4-trans');
        // <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">

    以下是预定义的文档类型选择的列表。这些是可配置的，可以从 `app/Config/DocTypes.php` 获取，也可以在 `.env` 配置中覆盖它们。

    =============================== =================== ==================================================================================================================================================
    Document type                   Option              Result
    =============================== =================== ==================================================================================================================================================
    XHTML 1.1                       xhtml11             <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
    XHTML 1.0 Strict                xhtml1-strict       <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
    XHTML 1.0 Transitional          xhtml1-trans        <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
    XHTML 1.0 Frameset              xhtml1-frame        <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">
    XHTML Basic 1.1                 xhtml-basic11       <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML Basic 1.1//EN" "http://www.w3.org/TR/xhtml-basic/xhtml-basic11.dtd">
    HTML 5                          html5               <!DOCTYPE html>
    HTML 4 Strict                   html4-strict        <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
    HTML 4 Transitional             html4-trans         <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    HTML 4 Frameset                 html4-frame         <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">
    MathML 1.01                     mathml1             <!DOCTYPE math SYSTEM "http://www.w3.org/Math/DTD/mathml1/mathml.dtd">
    MathML 2.0                      mathml2             <!DOCTYPE math PUBLIC "-//W3C//DTD MathML 2.0//EN" "http://www.w3.org/Math/DTD/mathml2/mathml2.dtd">
    SVG 1.0                         svg10               <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.0//EN" "http://www.w3.org/TR/2001/REC-SVG-20010904/DTD/svg10.dtd">
    SVG 1.1 Full                    svg11               <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
    SVG 1.1 Basic                   svg11-basic         <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1 Basic//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11-basic.dtd">
    SVG 1.1 Tiny                    svg11-tiny          <!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1 Tiny//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11-tiny.dtd">
    XHTML+MathML+SVG (XHTML host)   xhtml-math-svg-xh   <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1 plus MathML 2.0 plus SVG 1.1//EN" "http://www.w3.org/2002/04/xhtml-math-svg/xhtml-math-svg.dtd">
    XHTML+MathML+SVG (SVG host)     xhtml-math-svg-sh   <!DOCTYPE svg:svg PUBLIC "-//W3C//DTD XHTML 1.1 plus MathML 2.0 plus SVG 1.1//EN" "http://www.w3.org/2002/04/xhtml-math-svg/xhtml-math-svg.dtd">
    XHTML+RDFa 1.0                  xhtml-rdfa-1        <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML+RDFa 1.0//EN" "http://www.w3.org/MarkUp/DTD/xhtml-rdfa-1.dtd">
    XHTML+RDFa 1.1                  xhtml-rdfa-2        <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML+RDFa 1.1//EN" "http://www.w3.org/MarkUp/DTD/xhtml-rdfa-2.dtd">
    =============================== =================== ==================================================================================================================================================
