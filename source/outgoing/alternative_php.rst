###################################
在视图文件中使用 PHP 替代语法
###################################

如果您没有使用模板引擎来简化输出，那么您将在View文件中使用纯PHP。为了最大程度地减少这些文件中的PHP代码，并使识别代码块更加容易，建议您对控件结构和短标签echo语句使用PHP的替代语法。如果您不熟悉此语法，则可以使用它消除代码中的花括号，并消除“echo”语句。

Echo 的替代语法
=================

通常要回显或打印出一个变量，您可以这样做::

	<?php echo $variable; ?>

使用其他语法，您可以改用这种方式::

	<?= $variable ?>

替代控制结构
==============================

控制结构（例如if，for，foreach和while）也可以以简化格式编写。这是一个使用 ``foreach`` 的示例::

	<ul>

	<?php foreach ($todo as $item) : ?>

		<li><?= $item ?></li>

	<?php endforeach ?>

	</ul>

请注意，没有括号。而是将尾括号替换为 ``endforeach``。上面列出的每种控制结构具有类似的关闭语法: ``endif``, ``endfor``, ``endforeach``, 和 ``endwhile``。

另请注意，在每个结构（最后一个结构除外）后都没有使用分号，而是使用冒号。这个很重要！

这是使用 ``if``/``elseif``/``else`` 的另一个示例。注意冒号::

	<?php if ($username === 'sally') : ?>

		<h3>Hi Sally</h3>

	<?php elseif ($username === 'joe') : ?>

		<h3>Hi Joe</h3>

	<?php else : ?>

		<h3>Hi unknown user</h3>

	<?php endif ?>
