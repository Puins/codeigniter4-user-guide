****************
CLIRequest 类
****************

如果请求来自命令行调用，则请求对象实际上是 ``CLIRequest`` 。它的行为与 :doc:`conventional request </incoming/request>` 一样。但为了方便起见，添加了一些访问器方法。

====================
其他访问器
====================

**getSegments()**

返回被视为路径一部分的命令行参数数组::

    // 命令行: php index.php users 21 profile -foo bar
    echo $request->getSegments();  // ['users', '21', 'profile']

**getPath()**

以字符串形式返回重建的路径::

    // 命令行: php index.php users 21 profile -foo bar
    echo $request->getPath();  // users/21/profile

**getOptions()**

返回被视为选项的命令行参数数组::

    // 命令行: php index.php users 21 profile -foo bar
    echo $request->getOptions();  // ['foo' => 'bar']

**getOption($which)**

返回被视为选项的特定命令行参数的值::

    // 命令行: php index.php users 21 profile -foo bar
    echo $request->getOption('foo');  // bar
    echo $request->getOption('notthere'); // NULL

**getOptionString()**

返回选项的重建命令行字符串::

    // 命令行: php index.php users 21 profile -foo bar
    echo $request->getOptionPath();  // -foo bar
