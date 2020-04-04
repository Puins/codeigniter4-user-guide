###############
Form 辅助函数
###############

Form辅助函数文件包含协助使用Form的功能。

.. contents::
  :local:

.. raw:: html

  <div class="custom-index container"></div>

加载此辅助函数
===================

使用以下代码加载此辅助函数::

	helper('form');

转义字段值
=====================

您可能需要在表单元素中使用HTML和诸如引号的字符。为了安全地执行此操作，您需要使用 :doc:`common function <../general/common_functions>` 中的 :php:func:`esc()`。

考虑下文的示例::

	$string = 'Here is a string containing "quoted" text.';

	<input type="text" name="myfield" value="<?= $string; ?>" />

由于上面的字符串包含一组引号，因此将导致表单中断。:php:func:`esc()` 函数转换HTML特殊字符，以便可以安全地使用它::

	<input type="text" name="myfield" value="<?= esc($string); ?>" />

.. note:: 如果使用此页面上列出的任何表单辅助函数，并且将值作为关联数组传递，则表单值将自动转义，因此无需调用此函数。仅在创建自己的表单元素（将其作为字符串传递）时使用。

可用功能
===================

以下功能可用：

.. php:function:: form_open([$action = ''[, $attributes = ''[, $hidden = []]]])

	:param	string	$action: 表单操作/目标URI字符串
	:param	mixed	$attributes: HTML属性，作为数组或转义字符串
	:param	array	$hidden: 隐藏字段定义的数组
	:returns:	表单开始标签
	:rtype:	string

	使用您的配置首选项创建的带有基本URL的表单开始标签。可以选择添加表单属性和隐藏的输入字段，并始终根据配置文件中的字符集值添加 `accept-charset` 属性。

	使用此标签而不是对您自己的HTML进行硬编码的主要好处是，如果您的URL发生更改，它可以使您的网站更具可移植性。

	这是一个简单的示例::

		echo form_open('email/send');

	上面的示例将创建一个指向您的基本URL以及 "email/send" URI段的表单，如下所示::

		<form method="post" accept-charset="utf-8" action="http://example.com/index.php/email/send">

	**添加属性**

		可以通过将关联数组传递给第二个参数来添加属性，如下所示::

			$attributes = ['class' => 'email', 'id' => 'myform'];
			echo form_open('email/send', $attributes);

		或者，您可以将第二个参数指定为字符串::

			echo form_open('email/send', 'class="email" id="myform"');

		上面的示例将创建与此类似的表单::

			<form method="post" accept-charset="utf-8" action="http://example.com/index.php/email/send" class="email" id="myform">
			
		如果打开CSRF筛选器，则 `form_open()` 将在表单的开头生成CSRF字段。您可以通过将csrf_id传递为 `$attribute` 数组之一来指定此字段的ID:
		
			form_open('/u/sign-up', ['csrf_id' => 'my-id']);
			
		将返回:
		
			<form action="/u/sign-up" method="post" accept-charset="utf-8">
			<input type="hidden" id="my-id" name="csrf_field" value="964ede6e0ae8a680f7b8eab69136717d" />

	**添加隐藏的输入字段**

		可以通过将关联数组传递给第三个参数来添加隐藏字段，如下所示::

			$hidden = ['username' => 'Joe', 'member_id' => '234'];
			echo form_open('email/send', '', $hidden);

		您可以通过向其传递 `false` 值来跳过第二个参数。

		上面的示例将创建与此类似的表单::

			<form method="post" accept-charset="utf-8" action="http://example.com/index.php/email/send">
				<input type="hidden" name="username" value="Joe" />
				<input type="hidden" name="member_id" value="234" />

.. php:function:: form_open_multipart([$action = ''[, $attributes = ''[, $hidden = []]]])

	:param	string	$action: 表单操作/目标URI字符串
	:param	mixed	$attributes: HTML属性，作为数组或转义字符串
	:param	array	$hidden: 隐藏字段定义的数组
	:returns:	HTML multipart表单开始标签
	:rtype:	string

	此功能与上面的 :php:func:`form_open()` 功能相同，不同之处在于它添加了一个 *multipart* 属性，如果您想使用表单上传文件，则此功能是必需的。

.. php:function:: form_hidden($name[, $value = ''])

	:param	string	$name: 字段名
	:param	string	$value: 字段值
	:returns:	HTML隐藏的输入字段标签 
	:rtype:	string

	使您生成隐藏的输入字段。您可以提交名称/值字符串来创建一个字段::

		form_hidden('username', 'johndoe');
		// 将产生： <input type="hidden" name="username" value="johndoe" />

	... 或您可以提交关联数组以创建多个字段::

		$data = [
			'name'	=> 'John Doe',
			'email'	=> 'john@example.com',
			'url'	=> 'http://example.com'
		];

		echo form_hidden($data);

		/*
			将产生：
			<input type="hidden" name="name" value="John Doe" />
			<input type="hidden" name="email" value="john@example.com" />
			<input type="hidden" name="url" value="http://example.com" />
		*/

	您还可以将关联数组传递给value字段::

		$data = [
			'name'	=> 'John Doe',
			'email'	=> 'john@example.com',
			'url'	=> 'http://example.com'
		];

		echo form_hidden('my_array', $data);

		/*
			将产生：

			<input type="hidden" name="my_array[name]" value="John Doe" />
			<input type="hidden" name="my_array[email]" value="john@example.com" />
			<input type="hidden" name="my_array[url]" value="http://example.com" />
		*/

	如果要创建具有额外属性的隐藏输入字段，请执行以下操作::

		$data = [
			'type'	=> 'hidden',
			'name'	=> 'email',
			'id'	=> 'hiddenemail',
			'value'	=> 'john@example.com',
			'class'	=> 'hiddenemail'
		];

		echo form_input($data);

		/*
			将产生：

			<input type="hidden" name="email" value="john@example.com" id="hiddenemail" class="hiddenemail" />
		*/

.. php:function:: form_input([$data = ''[, $value = ''[, $extra = ''[, $type = 'text']]]])

	:param	array	$data: 字段属性数据
	:param	string	$value: 字段值
	:param	mixed	$extra: 额外属性以数组或文字字符串形式添加到标签中
	:param  string  $type: 输入字段的类型。即'text', 'email', 'number', 等等。
	:returns:	HTML文本输入字段标签
	:rtype:	string

	使您可以生成标准文本输入字段。您可以在第一个和第二个参数中最小地传递字段名称和值::

		echo form_input('username', 'johndoe');

	或者，您可以传递一个包含希望表单包含的任何数据的关联数组::

		$data = [
			'name'      => 'username',
			'id'        => 'username',
			'value'     => 'johndoe',
			'maxlength' => '100',
			'size'      => '50',
			'style'     => 'width:50%'
		];

		echo form_input($data);

		/*
			将产生：

			<input type="text" name="username" value="johndoe" id="username" maxlength="100" size="50" style="width:50%"  />
		*/

	如果您希望表单包含一些其他数据（例如JavaScript），则可以将其作为字符串传递给第三个参数::

		$js = 'onClick="some_function()"';
		echo form_input('username', 'johndoe', $js);

	或者您可以将其作为数组传递::

		$js = ['onClick' => 'some_function();'];
		echo form_input('username', 'johndoe', $js);

	为了支持扩展的HTML5输入字段范围，您可以传入输入类型作为第四个参数::

		echo form_input('email', 'joe@example.com', ['placeholder' => 'Email Address...'], 'email');

		/*
			 将产生：

			<input type="email" name="email" value="joe@example.com" placeholder="Email Address..." />
		*/

.. php:function:: form_password([$data = ''[, $value = ''[, $extra = '']]])

	:param	array	$data: 字段属性数据
	:param	string	$value: 字段值
	:param	mixed	$extra: 额外属性以数组或文字字符串形式添加到标签中
	:returns:	HTML密码输入字段标签
	:rtype:	string

	:php:func:`form_input()` 除了使用 "password" 输入类型外，此功能在所有方面均与上述功能相同。

.. php:function:: form_upload([$data = ''[, $value = ''[, $extra = '']]])

	:param	array	$data: 字段属性数据
	:param	string	$value: 字段值
	:param	mixed	$extra: 额外属性以数组或文字字符串形式添加到标签中
	:returns:	HTML文件上传输入字段标签
	:rtype:	string

	该功能在所有方面都与上述 :php:func:`form_input()` 功能相同，不同之处在于它使用 "file" 输入类型，可用于上传文件。

.. php:function:: form_textarea([$data = ''[, $value = ''[, $extra = '']]])

	:param	array	$data: 字段属性数据
	:param	string	$value: 字段值
	:param	mixed	$extra: 额外属性以数组或文字字符串形式添加到标签中
	:returns:	HTML textarea标签
	:rtype:	string

	该函数在所有方面都与上述 :php:func:`form_input()` 函数相同，除了它会生成 "textarea" 类型。

	.. note:: 代替上面示例中的 *maxlength* 和 *size* 属性，您将改为指定 *rows* 和 *cols*。

.. php:function:: form_dropdown([$name = ''[, $options = [][, $selected = [][, $extra = '']]]])

	:param	string	$name: 字段名
	:param	array	$options: 要列出的选项的关联数组
	:param	array	$selected: 要标记为 *selected* 属性的字段列表
	:param	mixed	$extra: 额外属性以数组或文字字符串形式添加到标签中
	:returns:	HTML下拉选择字段标记
	:rtype:	string

	使您可以创建标准下拉字段。第一个参数将包含字段名称，第二个参数将包含选项的关联数组，第三个参数将包含您希望选择的值。您还可以通过第三个参数传递多个项目的数组，帮助程序将为您创建一个多重选择。

	示例::

		$options = [
			'small'  => 'Small Shirt',
			'med'    => 'Medium Shirt',
			'large'  => 'Large Shirt',
			'xlarge' => 'Extra Large Shirt',
		];

		$shirts_on_sale = ['small', 'large'];
		echo form_dropdown('shirts', $options, 'large');

		/*
			将产生：

			<select name="shirts">
				<option value="small">Small Shirt</option>
				<option value="med">Medium Shirt</option>
				<option value="large" selected="selected">Large Shirt</option>
				<option value="xlarge">Extra Large Shirt</option>
			</select>
		*/

		echo form_dropdown('shirts', $options, $shirts_on_sale);

		/*
			将产生：

			<select name="shirts" multiple="multiple">
				<option value="small" selected="selected">Small Shirt</option>
				<option value="med">Medium Shirt</option>
				<option value="large" selected="selected">Large Shirt</option>
				<option value="xlarge">Extra Large Shirt</option>
			</select>
		*/

	如果希望开头的<select>包含其他数据（例如id属性或JavaScript），则可以将其作为字符串传递给第四个参数::

		$js = 'id="shirts" onChange="some_function();"';
		echo form_dropdown('shirts', $options, 'large', $js);

	或者您可以将其作为数组传递::

		$js = [
			'id'       => 'shirts',
			'onChange' => 'some_function();'
		];
		echo form_dropdown('shirts', $options, 'large', $js);

	如果按原样传递的数组 ``$options`` 是多维数组， ``form_dropdown()`` 则将生成一个<optgroup>，其中数组键为标签。

.. php:function:: form_multiselect([$name = ''[, $options = [][, $selected = [][, $extra = '']]]])

	:param	string	$name: 字段名
	:param	array	$options: 要列出的选项的关联数组
	:param	array	$selected: 要标记为 *selected* 属性的字段列表
	:param	mixed	$extra: 额外属性以数组或文字字符串形式添加到标签中
	:returns:	HTML下拉多选字段标记
	:rtype:	string

	使您可以创建标准的多选字段。第一个参数将包含字段名称，第二个参数将包含选项的关联数组，第三个参数将包含您希望选择的一个或多个值。

	参数的用法与上面的 :php:func:`form_dropdown()` 用法相同，除了字段名需要使用POST数组语法，例如foo[]。

.. php:function:: form_fieldset([$legend_text = ''[, $attributes = []]])

	:param	string	$legend_text: 放在<legend>标记中的文本
	:param	array	$attributes: 在<fieldset>标签上设置的属性
	:returns:	HTML字段集开始标记
	:rtype:	string

	用于生成字段 ``fieldset/legend`` 字段。

	示例::

		echo form_fieldset('Address Information');
		echo "<p>fieldset content here</p>\n";
		echo form_fieldset_close();

		/*
			产生：

				<fieldset>
					<legend>Address Information</legend>
						<p>form content here</p>
				</fieldset>
		*/

	与其他功能类似，如果您希望设置其他属性，则可以在第二个参数中提交关联数组::

		$attributes = [
			'id'	=> 'address_info',
			'class'	=> 'address_info'
		];

		echo form_fieldset('Address Information', $attributes);
		echo "<p>fieldset content here</p>\n";
		echo form_fieldset_close();

		/*
			产生：

			<fieldset id="address_info" class="address_info">
				<legend>Address Information</legend>
				<p>form content here</p>
			</fieldset>
		*/

.. php:function:: form_fieldset_close([$extra = ''])

	:param	string	$extra: 闭合标记附加的任何字段, *as is*
	:returns:	HTML fieldset结束标记
	:rtype:	string

	产生一个结束</ fieldset>标记。使用此功能的唯一好处是它允许您将数据传递给它，该数据将添加到标签下方。例如

	::

		$string = '</div></div>';
		echo form_fieldset_close($string);
		// 将产生： </fieldset></div></div>

.. php:function:: form_checkbox([$data = ''[, $value = ''[, $checked = FALSE[, $extra = '']]]])

	:param	array	$data: 字段属性数据
	:param	string	$value: 字段值
	:param	bool	$checked: 是否将复选框标记为 *checked* 状态
	:param	mixed	$extra: 额外属性以数组或文字字符串形式添加到标签中
	:returns:	HTML复选框输入标签
	:rtype:	string

	让您生成一个复选框字段。简单示例::

		echo form_checkbox('newsletter', 'accept', TRUE);
		// 将产生：  <input type="checkbox" name="newsletter" value="accept" checked="checked" />

	第三个参数包含布尔TRUE/FALSE，以确定是否应选中该框。

	类似于此辅助函数中的其他表单函数，您还可以将属性数组传递给该函数::

		$data = [
			'name'    => 'newsletter',
			'id'      => 'newsletter',
			'value'   => 'accept',
			'checked' => TRUE,
			'style'   => 'margin:10px'
		];

		echo form_checkbox($data);
		// 将产生： <input type="checkbox" name="newsletter" id="newsletter" value="accept" checked="checked" style="margin:10px" />

	与其他函数一样，如果您希望标记包含其他数据（例如JavaScript），则可以将其作为字符串传递给第四个参数::

		$js = 'onClick="some_function()"';
		echo form_checkbox('newsletter', 'accept', TRUE, $js);

	或者您可以将其作为数组传递::

		$js = ['onClick' => 'some_function();'];
		echo form_checkbox('newsletter', 'accept', TRUE, $js);

.. php:function:: form_radio([$data = ''[, $value = ''[, $checked = FALSE[, $extra = '']]]])

	:param	array	$data: 字段属性数据
	:param	string	$value: 字段值
	:param	bool	$checked: 是否将单选按钮标记为 *checked* 状态
	:param	mixed	$extra: 额外属性以数组或文字字符串形式添加到标签中
	:returns:	HTML单选输入标签
	:rtype:	string

	 :php:func:`form_checkbox()` 除了使用 "radio" 输入类型外，此功能在所有方面均与上述功能相同。

.. php:function:: form_label([$label_text = ''[, $id = ''[, $attributes = []]]])

	:param	string	$label_text: 放在<label>标记中的文本
	:param	string	$id: 我们为其创建标签的表单元素的ID
	:param	string	$attributes: HTML属性
	:returns:	HTML字段 label 标记
	:rtype:	string

	使您生成<label>。简单示例::

		echo form_label('What is your Name', 'username');
		// 将产生：  <label for="username">What is your Name</label>

	与其他功能类似，如果您希望设置其他属性，则可以在第三个参数中提交关联数组。

	示例::

		$attributes = [
			'class' => 'mycustomclass',
			'style' => 'color: #000;'
		];

		echo form_label('What is your Name', 'username', $attributes);
		// 将产生：  <label for="username" class="mycustomclass" style="color: #000;">What is your Name</label>

.. php:function:: form_submit([$data = ''[, $value = ''[, $extra = '']]])

	:param	string	$data: 按钮名称
	:param	string	$value: 按钮值
	:param	mixed	$extra: 额外属性以数组或文字字符串形式添加到标签中
	:returns:	HTML输入提交标签
	:rtype:	string

	使您可以生成标准的提交按钮。简单示例::

		echo form_submit('mysubmit', 'Submit Post!');
		// 将产生：  <input type="submit" name="mysubmit" value="Submit Post!" />

	与其他函数类似，如果您希望设置自己的属性，则可以在第一个参数中提交关联数组。第三个参数使您可以向表单中添加其他数据，例如JavaScript。

.. php:function:: form_reset([$data = ''[, $value = ''[, $extra = '']]])

	:param	string	$data: 按钮名称
	:param	string	$value: 按钮值
	:param	mixed	$extra: 额外属性以数组或文字字符串形式添加到标签中
	:returns:	HTML输入重置按钮标签
	:rtype:	string

	让您生成标准的重置按钮。使用与 :func:`form_submit()` 相同。

.. php:function:: form_button([$data = ''[, $content = ''[, $extra = '']]])

	:param	string	$data: 按钮名称
	:param	string	$content: 按钮标签
	:param	mixed	$extra: 额外属性以数组或文字字符串形式添加到标签中
	:returns:	HTML按钮标记
	:rtype:	string

	使您可以生成标准按钮元素。您可以在第一个和第二个参数中最少传递按钮名称和内容::

		echo form_button('name','content');
		// 将产生： <按钮名称="name" type="button">Content</button>

	或者，您可以传递一个包含希望表单包含的任何数据的关联数组::

		$data = [
			'name'    => 'button',
			'id'      => 'button',
			'value'   => 'true',
			'type'    => 'reset',
			'content' => 'Reset'
		];

		echo form_button($data);
		// 将产生： <按钮名称="button" id="button" value="true" type="reset">Reset</button>

	如果您希望表单包含一些其他数据（例如JavaScript），则可以将其作为字符串传递给第三个参数::

		$js = 'onClick="some_function()"';
		echo form_button('mybutton', 'Click Me', $js);

.. php:function:: form_close([$extra = ''])

	:param	string	$extra: 闭合标记附加的任何字段, *as is*
	:returns:	HTML表单结束标记
	:rtype:	string

	产生一个结束</ form>标记。使用此功能的唯一好处是它允许您将数据传递给它，该数据将添加到标签下方。对于示例::

		$string = '</div></div>';
		echo form_close($string);
		// 将产生：  </form> </div></div>

.. php:function:: set_value($field[, $default = ''[, $html_escape = TRUE]])

	:param	string	$field: 字段名
	:param	string	$default: 默认值
	:param  bool	$html_escape: 是否关闭值的HTML转义
	:returns:	字段值
	:rtype:	string

	允许您设置输入表单或文本区域的值。您必须通过函数的第一个参数来提供字段名。第二个（可选）参数允许您设置表单的默认值。第三个（可选）参数允许您关闭值的HTML转义，以防您需要将此功能与ie结合使用 :php:func:`form_input()` 并避免重复转义。

	示例::

		<input type="text" name="quantity" value="<?php echo set_value('quantity', '0'); ?>" size="50" />

	The above form will show "0" when loaded for the first time.

.. php:function:: set_select($field[, $value = ''[, $default = FALSE]])

	:param	string	$field: 字段名
	:param	string	$value: 要检查的值
	:param	string	$default: 该值是否也是默认值
	:returns:	'selected' 属性或空字符串
	:rtype:	string

	如果使用<select>菜单，则此功能允许您显示所选的菜单项。

	第一个参数必须包含选择菜单的名称，第二个参数必须包含每个项目的值，第三个（可选）参数使您可以将一个项目设置为默认值（使用布尔值TRUE/FALSE）。

	示例::

		<select name="myselect">
			<option value="one" <?php echo  set_select('myselect', 'one', TRUE); ?> >One</option>
			<option value="two" <?php echo  set_select('myselect', 'two'); ?> >Two</option>
			<option value="three" <?php echo  set_select('myselect', 'three'); ?> >Three</option>
		</select>

.. php:function:: set_checkbox($field[, $value = ''[, $default = FALSE]])

	:param	string	$field: 字段名
	:param	string	$value: 要检查的值
	:param	string	$default: 该值是否也是默认值
	:returns:	'checked' 属性或空字符串
	:rtype:	string

	允许您以提交状态显示一个复选框。

	第一个参数必须包含复选框的名称，第二个参数必须包含其值，第三个（可选）参数使您可以将项目设置为默认值（使用布尔值TRUE/FALSE）。

	示例::

		<input type="checkbox" name="mycheck" value="1" <?php echo set_checkbox('mycheck', '1'); ?> />
		<input type="checkbox" name="mycheck" value="2" <?php echo set_checkbox('mycheck', '2'); ?> />

.. php:function:: set_radio($field[, $value = ''[, $default = FALSE]])

	:param	string	$field: 字段名
	:param	string	$value: 要检查的值
	:param	string	$default: 该值是否也是默认值
	:returns:	'checked' 属性或空字符串
	:rtype:	string

	允许您以提交状态显示单选按钮。该功能与上述 :php:func:`set_checkbox()` 功能相同。

	示例::

		<input type="radio" name="myradio" value="1" <?php echo  set_radio('myradio', '1', TRUE); ?> />
		<input type="radio" name="myradio" value="2" <?php echo  set_radio('myradio', '2'); ?> />

	.. note:: 如果使用的是Form Validation类，则必须始终为您的字段指定一个规则（即使为空），以使 ``set_*()`` 功能起作用。这是因为，如果定义了表单验证对象，则将 ``set_*()`` 移交给类的方法，而不是通用辅助函数。
