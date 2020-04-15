################
User Agent 类
################

用户代理类提供帮助识别访问您网站的浏览器、移动设备或机器人信息的函数。

.. contents::
    :local:
    :depth: 2

**************************
使用用户代理类
**************************

初始化类
======================

用户代理类始终可以直接从当前的 :doc:`IncomingRequest </incoming/incomingrequest>` 实例直接获得。默认情况下，您的控制器中将有一个请求实例，您可以检索该实例来自的用户代理类::

	$agent = $this->request->getUserAgent();

用户代理定义
======================

用户代理名称定义位于: **app/Config/UserAgents.php** 配置文件中。 您可以将项目添加到您的用户代理数组（如果需要）。

示例
=======

当用户代理类初始化时，它将尝试确定浏览您网站的用户代理是否是web浏览器、移动设备设备或机器人。它还将收集平台信息（如果可用）::

	$agent = $this->request->getUserAgent();

	if ($agent->isBrowser())
	{
		$currentAgent = $agent->getBrowser().' '.$agent->getVersion();
	}
	elseif ($agent->isRobot())
	{
		$currentAgent = $this->agent->robot();
	}
	elseif ($agent->isMobile())
	{
		$currentAgent = $agent->getMobile();
	}
	else
	{
		$currentAgent = 'Unidentified User Agent';
	}

	echo $currentAgent;

	echo $agent->getPlatform(); // 平台信息 (Windows, Linux, Mac, 等等。)

***************
类参考
***************

.. php:class:: CodeIgniter\\HTTP\\UserAgent

	.. php:method:: isBrowser([$key = NULL])

		:param	string	$key: 可选浏览器名称
    		:returns:	如果用户代理是（指定的）浏览器，则为TRUE；如果不是，则为FALSE
    		:rtype:	bool

    		如果用户代理是已知的web浏览器，则返回TRUE/FALSE（布尔值）。
    		::

			if ($agent->isBrowser('Safari'))
			{
				echo 'You are using Safari.';
			}
			elseif ($agent->isBrowser())
			{
				echo 'You are using a browser.';
			}

		.. note:: 本例中的字符串 ``Safari`` 是浏览器定义列表中的一个数组键。如果要添加新的浏览器或更改字符串，可以在 **app/Config/UserAgents.php** 中找到此列表。

	.. php:method:: isMobile([$key = NULL])

		:param	string	$key: 可选移动设备名称
    		:returns:	如果用户代理是（指定的）移动设备，则为TRUE；如果不是，则为FALSE
    		:rtype:	bool

    		如果用户代理是已知的移动设备，则返回TRUE/FALSE（布尔值）。
    		::

			if ($agent->isMobile('iphone'))
			{
				echo view('iphone/home');
			}
			elseif ($agent->isMobile())
			{
				echo view('mobile/home');
			}
			else
			{
				echo view('web/home');
			}

	.. php:method:: isRobot([$key = NULL])

		:param	string	$key: 可选机器人名称
    		:returns:	如果用户代理是（指定的）robot，则为TRUE；如果不是，则为FALSE
    		:rtype:	bool

    		如果用户代理是已知的robot，则返回TRUE/FALSE（布尔值）。

    		.. note:: 用户代理库只包含最常见的robot定义。它不是一个完整的机器人列表。有成百上千个这样的搜索每一个都不是很有效。如果你发现一些机器人通常访问您的网站的列表中没有，您可以将其添加到 **app/Config/UserAgents.php** 文件。

	.. php:method:: isReferral()

		:returns:	如果用户代理是引用的（referral），则为TRUE；如果不是，则为FALSE
		:rtype:	bool

		如果用户代理是从其他站点引用的，则返回TRUE/FALSE（布尔值）。

	.. php:method:: getBrowser()

		:returns:	检测到浏览器或空字符串
		:rtype:	string

		返回包含查看网站的web浏览器名称的字符串。

	.. php:method:: getVersion()

		:returns:	检测到浏览器版本或空字符串
		:rtype:	string

		返回包含查看网站的web浏览器版本号的字符串。

	.. php:method:: getMobile()

		:returns:	检测到移动设备品牌或空字符串
		:rtype:	string

		返回包含查看网站的移动设备名称的字符串。

	.. php:method:: getRobot()

		:returns:	检测到的机器人名称或空字符串
		:rtype:	string
        
		返回包含查看网站的机器人名称的字符串。

	.. php:method:: getPlatform()

		:returns:	检测到OS或空字符串
		:rtype:	string

		返回包含查看网站的平台 (Linux, Windows, OS X, 等等。)。

	.. php:method:: getReferrer()

		:returns:	检测到引用或空字符串
		:rtype:	string

		如果用户代理是从其他站点引用的，则为引用者。通常情况下，您将按如下方式进行测试::

			if ($agent->isReferral())
			{
				echo $agent->referrer();
			}

	.. php:method:: getAgentString()

		:returns:	完整用户代理字符串或空字符串
		:rtype:	string

		返回包含完整用户代理字符串的字符串。通常是这样的::

			Mozilla/5.0 (Macintosh; U; Intel Mac OS X; en-US; rv:1.8.0.4) Gecko/20060613 Camino/1.0.2

	.. php:method:: parse($string)

		:param	string	$string: 自定义用户代理字符串
    		:rtype:	void

    		分析自定义用户代理字符串，该字符串不同于当前访问者报告的字符串。
