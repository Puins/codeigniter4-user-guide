###############
Text 辅助函数
###############

Text 辅助函数包含协助处理文本的功能。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

加载辅助函数
===================

使用以下代码加载辅助函数::

	helper('text');

可用函数
===================

该辅助函数有下列可用函数：

.. php:function:: random_string([$type = 'alnum'[, $len = 8]])

	:param	string	$type: 随机类型
	:param	int	$len: 输出字符串长度
	:returns:	随机字符串
	:rtype:	string

	根据您指定的类型和长度生成一个随机字符串。用于创建密码或生成随机哈希。

	第一个参数指定字符串的类型，第二个参数指定长度。提供以下选择:

	-  **alpha**: 仅包含小写和大写字母的字符串。
	-  **alnum**: 包含小写和大写字符的字母数字字符串。
	-  **basic**: 基于 ``mt_rand()`` （忽略长度）的随机数。
	-  **numeric**: 数字字符串。
	-  **nozero**: 不为零的数字字符串。
	-  **md5**: 基于 ``md5()`` (固定长度为32)的加密随机数。
	-  **sha1**: 基于 ``sha1()`` (固定长度为40)的加密随机数。
	-  **crypto**: 基于 ``random_bytes()`` 的随机字符串。

	用法示例::

		echo random_string('alnum', 16);

.. php:function:: increment_string($str[, $separator = '_'[, $first = 1]])

	:param	string	$str: 输入字符串
	:param	string	$separator: 附加重复数字的分隔符
	:param	int	$first: 起始号码
	:returns:	递增的字符串
	:rtype:	string

	通过在字符串后附加数字或增加数字来增加字符串。对于创建“副本”或文件或复制具有唯一标题或标签的数据库内容很有用。

	用法示例::

		echo increment_string('file', '_'); // "file_1"
		echo increment_string('file', '-', 2); // "file-2"
		echo increment_string('file_4'); // "file_5"

.. php:function:: alternator($args)

	:param	mixed	$args: 可变数量的参数
	:returns:	交替字符串(s)
	:rtype:	mixed

	在循环中循环时，允许两个或多个项目交替出现。 示例::

		for ($i = 0; $i < 10; $i++)
		{     
			echo alternator('string one', 'string two');
		}

	您可以根据需要添加任意数量的参数，循环的每次迭代都将返回下一项。

	::

		for ($i = 0; $i < 10; $i++)
		{     
			echo alternator('one', 'two', 'three', 'four', 'five');
		}

	.. note:: 要对该函数使用多个单独的调用，只需调用不带参数的函数即可重新初始化。

.. php:function:: reduce_double_slashes($str)

	:param	string	$str: 输入字符串
	:returns:	带有正斜杠的字符串
	:rtype:	string

	将字符串中的双斜杠转换为单斜杠，但URL协议前缀（例如 http&#58;//）中的斜杠除外。

	示例::

		$string = "http://example.com//index.php";
		echo reduce_double_slashes($string); // 结果是 "http://example.com/index.php"

.. php:function:: strip_slashes($data)

	:param	mixed	$data: 输入字符串 or an array of strings
	:returns:	带斜杠的字符串
	:rtype:	mixed

	从字符串数组中删除任何斜杠。

	示例::

		$str = [
			'question' => 'Is your name O\'reilly?',
			'answer'   => 'No, my name is O\'connor.'
		];

		$str = strip_slashes($str);

	上面将返回以下数组::

		[
			'question' => "Is your name O'reilly?",
			'answer'   => "No, my name is O'connor."
		];

	.. note:: 由于历史原因，此函数还将接受并处理字符串输入。但是，这只是 ``stripslashes()`` 的别名。

.. php:function:: reduce_multiples($str[, $character = ''[, $trim = FALSE]])

	:param	string	$str: 搜索的文本
	:param	string	$character: 要减少的字符
	:param	bool	$trim: 是否也移除指定的字符
	:returns:	减少的字符串
	:rtype:	string

	减少一个特定字符的多个实例彼此直接发生。示例::

		$string = "Fred, Bill,, Joe, Jimmy";
		$string = reduce_multiples($string,","); //结果是 "Fred, Bill, Joe, Jimmy"

	如果第三个参数设置为TRUE，它将删除在字符串的开头和结尾出现的字符。示例::

		$string = ",Fred, Bill,, Joe, Jimmy,";
		$string = reduce_multiples($string, ", ", TRUE); //结果是 "Fred, Bill, Joe, Jimmy"

.. php:function:: quotes_to_entities($str)

	:param	string	$str: 输入字符串
	:returns:	带引号的字符串转换为HTML实体
	:rtype:	string

	将字符串中的单引号和双引号转换为相应的HTML实体。示例::

		$string = "Joe's \"dinner\"";
		$string = quotes_to_entities($string); //结果是 "Joe&#39;s &quot;dinner&quot;"

.. php:function:: strip_quotes($str)

	:param	string	$str: 输入字符串
	:returns:	带有引号的字符串
	:rtype:	string

	从字符串中删除单引号和双引号。示例::

		$string = "Joe's \"dinner\"";
		$string = strip_quotes($string); //结果是 "Joes dinner"

.. php:function:: word_limiter($str[, $limit = 100[, $end_char = '&#8230;']])

	:param	string	$str: 输入字符串
	:param	int	$limit: 限制
	:param	string	$end_char: 结束字符（通常为省略号）
	:returns:	字数限制字串
	:rtype:	string

	将字符串截断为指定的单词数。示例::

		$string = "Here is a nice text string consisting of eleven words.";
		$string = word_limiter($string, 4);
		// 返回:  Here is a nice

	第三个参数是添加到字符串的可选后缀。默认情况下，它添加省略号。

.. php:function:: character_limiter($str[, $n = 500[, $end_char = '&#8230;']])

	:param	string	$str: 输入字符串
	:param	int	$n: 字符数
	:param	string	$end_char: 结束字符（通常为省略号）
	:returns:	字符限制的字符串
	:rtype:	string

	将字符串截断为指定的字符数。它保持单词的完整性，因此字符数可能比您指定的略多或少。

	示例::

		$string = "Here is a nice text string consisting of eleven words.";
		$string = character_limiter($string, 20);
		// 返回:  Here is a nice text string

	第三个参数是添加到字符串的可选后缀，如果未声明，则此辅助函数使用省略号。

	.. note:: 如果需要截断精确的字符数，请参见下面的 :php:func:`ellipsize()` 函数。

.. php:function:: ascii_to_entities($str)

	:param	string	$str: 输入字符串
	:returns:	具有ASCII值的字符串转换为实体
	:rtype:	string

	将ASCII值转换为字符实体，包括高ASCII和MS Word字符，这些字符在网页中使用时可能会引起问题，因此无论浏览器设置如何都可以一致地显示它们或将其可靠地存储在数据库中。取决于服务器支持的字符集，因此它不一定在所有情况下都是100％可靠的，但是在大多数情况下，它应正确识别正常范围之外的字符（如重音字符）。

	示例::

		$string = ascii_to_entities($string);

.. php:function:: entities_to_ascii($str[, $all = TRUE])

	:param	string	$str: 输入字符串
	:param	bool	$all: 是否也转换不安全的实体
	:returns:	将HTML实体转换为ASCII字符的字符串
	:rtype:	string

	此功能的作用与 :php:func:`ascii_to_entities()` 相反。它将字符实体转换回ASCII。

.. php:function:: convert_accented_characters($str)

	:param	string	$str: 输入字符串
	:returns:	带有重音字符的字符串已转换
	:rtype:	string

	将高ASCII字符音译为低ASCII等价字符。在仅安全使用标准ASCII字符（例如URL）的情况下需要使用非英语字符时很有用。

	示例::

		$string = convert_accented_characters($string);

	.. note:: 此函数使用配套的配置文件 `app/Config/ForeignCharacters.php` 定义用于音译的to和from数组。

.. php:function:: word_censor($str, $censored[, $replacement = ''])

	:param	string	$str: 输入字符串
	:param	array	$censored: 要检查的不良词列表
	:param	string	$replacement: 用什么替换不良词
	:returns:	审查字符串
	:rtype:	string

	使您可以审查文本字符串中的单词。第一个参数将包含原始字符串。第二个将包含您不允许的单词数组。第三个（可选）参数可以包含单词的替换值。如果未指定，则将其替换为井号：####。

	示例::

		$disallowed = ['darn', 'shucks', 'golly', 'phooey'];
		$string     = word_censor($string, $disallowed, 'Beep!');

.. php:function:: highlight_code($str)

	:param	string	$str: 输入字符串
	:returns:	通过HTML突出显示代码的字符串
	:rtype:	string

	着色一串代码（PHP，HTML等）。示例::

		$string = highlight_code($string);

	该函数使用PHP的 ``highlight_string()`` 函数，因此使用的颜色是在 `php.ini` 文件中指定的颜色。

.. php:function:: highlight_phrase($str, $phrase[, $tag_open = '<mark>'[, $tag_close = '</mark>']])

	:param	string	$str: 输入字符串
	:param	string	$phrase: 高亮的短语
	:param	string	$tag_open: 用于高亮的开始标记
	:param	string	$tag_close: 高亮的结束标记
	:returns:	通过HTML高亮短语的字符串
	:rtype:	string

	将突出显示文本字符串中的短语。第一个参数将包含原始字符串，第二个参数将包含您要突出显示的短语。第三个和第四个参数将包含您希望将短语包装在其中的打开/关闭HTML标签。

	示例::

		$string = "Here is a nice text string about nothing in particular.";
		echo highlight_phrase($string, "nice text", '<span style="color:#990000;">', '</span>');

	上面的代码打印::

		Here is a <span style="color:#990000;">nice text</span> string about nothing in particular.

	.. note:: 此功能 ``<strong>`` 默认情况下使用标记。较旧的浏览器可能不支持新的HTML5标记，因此，如果需要支持这样的浏览器，建议将以下CSS代码插入样式表。
    
	::
	
		mark {
			background: #ff0;
			color: #000;
		};

.. php:function:: word_wrap($str[, $charlim = 76])

	:param	string	$str: 输入字符串
	:param	int	$charlim: 字符数限制
	:returns:	换行字符串
	:rtype:	string

	以指定的字符数换行，同时保留完整的单词。

	示例::

		$string = "Here is a simple string of text that will help us demonstrate this function.";
		echo word_wrap($string, 25);

		// Would produce:
		// Here is a simple string
		// of text that will help us
		// demonstrate this
		// function.

        过长的单词将被分割，但URL不会被分割。

.. php:function:: ellipsize($str, $max_length[, $position = 1[, $ellipsis = '&hellip;']])

	:param	string	$str: 输入字符串
	:param	int	$max_length: 字符串长度限制
	:param	mixed	$position: 拆分位置（int或float）
	:param	string	$ellipsis: 用作省略号字符
	:returns:	Ellipsized 字符串
	:rtype:	string

	此函数将从字符串中剥离标签，以定义的最大长度将其分割，然后插入省略号。

	第一个参数是ellipsize的字符串，第二个参数是最终字符串中的字符数。第三个参数是字符串中的省略号应从0到1，从左到右出现的位置。例如。值1将省略号放在字符串的右边，.5放在字符串的中间，0放在左边。

	可选的第四个参数是省略号的种类。默认情况下，ellipsis 将被插入。

	示例::

		$str = 'this_string_is_entirely_too_long_and_might_break_my_design.jpg';
		echo ellipsize($str, 32, .5);

	产生::

		this_string_is_e&hellip;ak_my_design.jpg

.. php:function:: excerpt($text, $phrase = false, $radius = 100, $ellipsis = '...')

	:param	string	$text: 文本摘录
	:param	string	$phrase: 提取单词周围的短语或单词
	:param	int		$radius: `$phrase` 之前和之后的字符数
	:param	string	$ellipsis: 用作省略号字符
	:returns:	摘抄。
	:rtype:		string

	此函数将提取中心短语之前和之后的 `$radius` 字符数，并在其前后加上一个省略号。

	第一个参数是要从中摘录的文本，第二个参数是要计算的前后单词的中心词或短语。第三个参数是中心短语之前和之后要计数的字符数。如果未通过任何短语，则摘录将包括第一个 `$radius` 字符，并在结尾加上省略号。

	示例::

		$text = 'Ut vel faucibus odio. Quisque quis congue libero. Etiam gravida
		eros lorem, eget porttitor augue dignissim tincidunt. In eget risus eget
		mauris faucibus molestie vitae ultricies odio. Vestibulum id ultricies diam.
		Curabitur non mauris lectus. Phasellus eu sodales sem. Integer dictum purus
		ac enim hendrerit gravida. Donec ac magna vel nunc tincidunt molestie sed
		vitae nisl. Cras sed auctor mauris, non dictum tortor. Nulla vel scelerisque
		arcu. Cras ac ipsum sit amet augue laoreet laoreet. Aenean a risus lacus.
		Sed ut tortor diam.';

		echo excerpt($str, 'Donec');

	产生::

		... non mauris lectus. Phasellus eu sodales sem. Integer dictum purus ac
		enim hendrerit gravida. Donec ac magna vel nunc tincidunt molestie sed
		vitae nisl. Cras sed auctor mauris, non dictum ...
