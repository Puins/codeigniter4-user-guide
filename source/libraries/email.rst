###########
Email 类
###########

CodeIgniter强大的电子邮件类支持以下功能:

-  多种协议: Mail, Sendmail, 和 SMTP
-  SMTP的TLS和SSL加密
-  多个收件人
-  抄送和密件抄送
-  HTML或纯文本 email
-  附件
-  自动换行
-  优先事项
-  密件抄送批处理模式，可将大型电子邮件列表拆分为小型密件抄送。
-  Email 调试工具

.. contents::
    :local:
    :depth: 2

.. raw:: html

  <div class="custom-index container"></div>

***********************
使用 Email 库
***********************

发送电子邮件
=============

发送电子邮件不仅简单，而且您可以随时对其进行配置，也可以在 **app/Config/Email.php** 文件中设置您的首选项。

这是一个基本示例，演示如何发送电子邮件::

	$email = \Config\Services::email();

	$email->setFrom('your@example.com', 'Your Name');
	$email->setTo('someone@example.com');
	$email->setCC('another@another-example.com');
	$email->setBCC('them@their-example.com');

	$email->setSubject('Email Test');
	$email->setMessage('Testing the email class.');

	$email->send();

设置电子邮件首选项
=========================

有21种不同的首选项可用于定制电子邮件的发送方式。您可以按照此处所述手动设置它们，也可以通过存储在配置文件中的偏好设置自动设置它们，如下所述:

通过将一组首选项值传递给电子邮件初始化方法来设置首选项。这是您如何设置某些首选项的示例::

	$config['protocol'] = 'sendmail';
	$config['mailPath'] = '/usr/sbin/sendmail';
	$config['charset']  = 'iso-8859-1';
	$config['wordWrap'] = true;

	$email->initialize($config);

.. note:: 大多数首选项均具有默认值，如果您未设置它们，则将使用它们。

在配置文件中设置电子邮件首选项
------------------------------------------

如果您不想使用上述方法设置首选项，则可以将其放入配置文件中。只需打开 **app/Config/Email.php** 文件，然后在Email属性中设置您的配置。然后保存文件，它将自动使用。如果您在配置文件中设置了首选项，则无需使用该方法 ``$email->initialize()``。

电子邮件首选项
=================

以下是发送电子邮件时可以设置的所有首选项的列表。

=================== ====================== ============================ =======================================================================
首选项               默认值                  选项                         描述
=================== ====================== ============================ =======================================================================
**userAgent**       CodeIgniter            None                         “用户代理”。
**protocol**        mail                   mail, sendmail, or smtp      邮件发送协议。
**mailpath**        /usr/sbin/sendmail     None                         Sendmail的服务器路径。
**SMTPHost**        No Default             None                         SMTP服务器地址。
**SMTPUser**        No Default             None                         SMTP用户名。
**SMTPPass**        No Default             None                         SMTP密码。
**SMTPPort**        25                     None                         SMTP端口。
**SMTPTimeout**     5                      None                         SMTP超时（以秒为单位）。
**SMTPKeepAlive**   FALSE                  TRUE or FALSE (boolean)      启用持久的SMTP连接。
**SMTPCrypto**      No Default             tls or ssl                   SMTP加密
**wordWrap**        TRUE                   TRUE or FALSE (boolean)      启用自动换行。
**wrapChars**       76                                                  要换行的字符数。
**mailType**        text                   text or html                 邮件类型。如果发送HTML电子邮件，则必须将其作为完整的网页发送。确保您没有任何相对链接或相对图像路径，否则它们将不起作用。
**charset**         utf-8                                               字符集设置 (utf-8, iso-8859-1, 等等).
**validate**        TRUE                   TRUE or FALSE (boolean)      是否验证电子邮件地址。
**priority**        3                      1, 2, 3, 4, 5                电子邮件优先级。1 =最高。5 =最低。3 =正常。
**CRLF**            \\n                    "\\r\\n" or "\\n" or "\\r"   换行符。 (使用 "\\r\\n" 符合 RFC 822).
**newline**         \\n                    "\\r\\n" or "\\n" or "\\r"   换行符。 (使用 "\\r\\n" 符合 RFC 822).
**BCCBatchMode**    FALSE                  TRUE or FALSE (boolean)      启用密件抄送批处理模式。
**BCCBatchSize**    200                    None                         每个密件抄送批次中的电子邮件数量。
**DSN**             FALSE                  TRUE or FALSE (boolean)      启用来自服务器的通知消息
=================== ====================== ============================ =======================================================================

重写换行
========================

如果您启用了自动换行（建议遵循 RFC 822），并且电子邮件中的链接很长，则该链接也可能被自动换行，从而导致收信人无法点击它。CodeIgniter允许您手动覆盖部分消息中的自动换行，如下所示::

	The text of your email that
	gets wrapped normally.

	{unwrap}http://example.com/a_long_link_that_should_not_be_wrapped.html{/unwrap}

	More text that will be
	wrapped normally.


将不需要的单词放在以下各项之间: {unwrap} {/unwrap}

***************
类参考
***************

.. php:class:: CodeIgniter\\Email\\Email

	.. php:method:: setFrom($from[, $name = ''[, $returnPath = null]])

		:param	string	$from: “发件人”电子邮件地址
		:param	string	$name: “发件人”名称
		:param	string	$returnPath: 可选的电子邮件地址，用于将未发送的电子邮件重定向到
		:returns:	CodeIgniter\\Email\\Email 实例 (方法链)
		:rtype:	CodeIgniter\\Email\\Email

		设置电子邮件地址和发送电子邮件的人的姓名::

			$email->setFrom('you@example.com', 'Your Name');

		您还可以设置返回路径，以帮助重定向未送达的邮件::

			$email->setFrom('you@example.com', 'Your Name', 'returned_emails@example.com');

		.. note:: 如果您已将“smtp”配置为协议，则无法使用 ``$returnPath``。

	.. php:method:: setReplyTo($replyto[, $name = ''])

		:param	string	$replyto: 回复的电子邮件地址
		:param	string	$name: 显示回复电子邮件地址的名称
		:returns:	CodeIgniter\\Email\\Email 实例 (方法链)
		:rtype:	CodeIgniter\\Email\\Email

		设置回复地址。如果未提供信息，则使用 `setFrom <#setFrom>`_ 方法中的信息。示例::

			$email->setReplyTo('you@example.com', 'Your Name');

	.. php:method:: setTo($to)

		:param	mixed	$to: 以逗号分隔的字符串或电子邮件地址数组
		:returns:	CodeIgniter\\Email\\Email 实例 (方法链)
		:rtype:	CodeIgniter\\Email\\Email

		设置收件人的电子邮件地址。可以是一封电子邮件，逗号分隔的列表或数组::

			$email->setTo('someone@example.com');

		::

			$email->setTo('one@example.com, two@example.com, three@example.com');

		::

			$email->setTo(['one@example.com', 'two@example.com', 'three@example.com']);

	.. php:method:: setCC($cc)

		:param	mixed	$cc: 逗号分隔的字符串或电子邮件地址数组
		:returns:	CodeIgniter\\Email\\Email 实例 (方法链)
		:rtype:	CodeIgniter\\Email\\Email

		设置抄送电子邮件地址。就像“收件人”一样，可以是一封电子邮件，一个逗号分隔的列表或一个数组。

	.. php:method:: setBCC($bcc[, $limit = ''])

		:param	mixed	$bcc: 逗号分隔的字符串或电子邮件地址数组
		:param	int	$limit: 每批发送的最大电子邮件数
		:returns:	CodeIgniter\\Email\\Email 实例 (方法链)
		:rtype:	CodeIgniter\\Email\\Email

		设置密件抄送电子邮件地址。就像 ``setTo()`` 方法一样，可以是一封电子邮件，一个逗号分隔的列表或一个数组。

		如果 ``$limit`` 已设置，则将启用“批处理模式”，它将按批次发送电子邮件，每批不超过指定的 ``$limit``。

	.. php:method:: setSubject($subject)

		:param	string	$subject: 电子邮件主题行
		:returns:	CodeIgniter\\Email\\Email 实例 (方法链)
		:rtype:	CodeIgniter\\Email\\Email

		设置电子邮件主题::

			$email->setSubject('This is my subject');

	.. php:method:: setMessage($body)

		:param	string	$body: 电子邮件正文
		:returns:	CodeIgniter\\Email\\Email 实例 (方法链)
		:rtype:	CodeIgniter\\Email\\Email

		设置电子邮件正文::

			$email->setMessage('This is my message');

	.. php:method:: setAltMessage($str)

		:param	string	$str: 备用电子邮件正文
		:returns:	CodeIgniter\\Email\\Email 实例 (方法链)
		:rtype:	CodeIgniter\\Email\\Email

		设置备用电子邮件正文::

			$email->setAltMessage('This is the alternative message');

		这是一个可选的消息字符串，如果您发送HTML格式的电子邮件，则可以使用它。它使您可以指定不带HTML格式的替代消息，该消息将添加到不接受HTML电子邮件的人的标题字符串中。如果您未设置自己的消息，CodeIgniter将从HTML电子邮件中提取消息并剥离标签。

	.. php:method:: setHeader($header, $value)

		:param	string	$header: 标头名称
		:param	string	$value: 标头值
		:returns:	CodeIgniter\\Email\\Email 实例 (方法链)
		:rtype: CodeIgniter\\Email\\Email

		将其他标头附加到电子邮件::

			$email->setHeader('Header1', 'Value1');
			$email->setHeader('Header2', 'Value2');

	.. php:method:: clear($clearAttachments = false)

		:param	bool	$clearAttachments: 是否清除附件
		:returns:	CodeIgniter\\Email\\Email 实例 (方法链)
		:rtype: CodeIgniter\\Email\\Email

		将所有电子邮件变量初始化为空状态。如果您循环运行电子邮件发送方法，则允许使用该方法，从而允许在两个周期之间重置数据。

		::

			foreach ($list as $name => $address)
			{
				$email->clear();

				$email->setTo($address);
				$email->setFrom('your@example.com');
				$email->setSubject('Here is your info '.$name);
				$email->setMessage('Hi ' . $name . ' Here is the info you requested.');
				$email->send();
			}

		如果将参数设置为TRUE，则所有附件也会被清除::

			$email->clear(true);

	.. php:method:: send($autoClear = true)

		:param	bool	$autoClear: 是否自动清除消息数据
		:returns:	成功则为TRUE，失败则为FALSE
		:rtype:	bool

		电子邮件发送方法。根据成功或失败返回布尔值TRUE或FALSE，使其有条件地使用::

			if (! $email->send())
			{
				// 生成错误
			}

		如果请求成功，此方法将自动清除所有参数。要停止此行为，请通过FALSE::

			if ($email->send(false))
			{
				// 参数不会被清除
			}

		.. note:: 为了使用 ``printDebugger()`` 方法，您需要避免清除电子邮件参数。

		.. note:: 如果 ``BCCBatchMode`` 启用，并且 ``BCCBatchSize`` 接收者多于此，则此方法将始终返回 ``TRUE``。

	.. php:method:: attach($filename[, $disposition = ''[, $newname = null[, $mime = '']]])

		:param	string	$filename: 文件名
		:param	string	$disposition: 附件的“处置”。无论此处使用的MIME规范如何，大多数电子邮件客户端都会自行决定。https://www.iana.org/assignments/cont-disp/cont-disp.xhtml
		:param	string	$newname: 在电子邮件中使用的自定义文件名
		:param	string	$mime: 要使用的MIME类型（对缓冲的数据很有用）
		:returns:	CodeIgniter\\Email\\Email 实例 (方法链)
		:rtype:	CodeIgniter\\Email\\Email

		使您可以发送附件。将文件路径/名称放在第一个参数中。对于多个附件，请多次使用该方法。示例::

			$email->attach('/path/to/photo1.jpg');
			$email->attach('/path/to/photo2.jpg');
			$email->attach('/path/to/photo3.jpg');

		要使用默认处置方式（附件），请将第二个参数留空，否则请使用自定义处置方式::

			$email->attach('image.jpg', 'inline');

		您还可以使用以下网址::

			$email->attach('http://example.com/filename.pdf');

		如果要使用自定义文件名，则可以使用第三个参数::

			$email->attach('filename.pdf', 'attachment', 'report.pdf');

		如果需要使用缓冲字符串而不是实际文件，则可以将第一个参数用作缓冲区，将第三个参数用作文件名，第四个参数用作mime-type::

			$email->attach($buffer, 'attachment', 'report.pdf', 'application/pdf');

	.. php:method:: setAttachmentCID($filename)

		:param	string	$filename: 现有附件文件名
		:returns:	附件Content-ID或FALSE（如果找不到）
		:rtype:	string

		设置并返回附件的Content-ID，使您可以将内嵌（图片）附件嵌入HTML。第一个参数必须是已经附加的文件名。
		::

			$filename = '/img/photo1.jpg';
			$email->attach($filename);
			foreach ($list as $address)
			{
				$email->setTo($address);
				$cid = $email->setAttachmentCID($filename);
				$email->setMessage('<img src="cid:'. $cid .'" alt="photo1" />');
				$email->send();
			}

		.. note:: 必须重新创建每个电子邮件的Content-ID，以使其唯一。

	.. php:method:: printDebugger($include = ['headers', 'subject', 'body'])

		:param	array	$include: 消息的哪一部分打印出来
		:returns:	格式化的调试数据
		:rtype:	string

		返回包含任何服务器消息，电子邮件标题和电子邮件消息的字符串。对于调试很有用。

		您可以选择指定应打印消息的哪些部分。有效选项为: **headers**, **subject**, **body**。

		示例::

			// You need to pass FALSE while sending in order for the email data
			// to not be cleared - if that happens, printDebugger() would have
			// nothing to output.
			$email->send(false);

			// Will only print the email headers, excluding the message subject and body
			$email->printDebugger(['headers']);

		.. note:: 默认情况下，将打印所有原始数据。
