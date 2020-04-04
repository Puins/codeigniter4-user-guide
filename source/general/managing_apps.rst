##########################
管理多个应用
##########################

默认情况下，假定您仅打算使用CodeIgniter来管理一个应用程序，该应用程序将在您的 **app** 目录中构建。但是，可以有多个应用程序共享一个CodeIgniter安装，甚至可以重命名或重定位应用程序目录。

重命名或重定位应用程序（app）目录
================================================

如果您想重命名应用程序目录，或者甚至将其移动到服务器上除项目根目录之外的其他位置，请打开主 **app/Config/Paths.php** 并在 ``$appDirectory`` 变量中设置 *完整的服务器路径* （大约在第38行）::

	public $appDirectory = '/path/to/your/application';

您将需要在项目根目录中修改另外两个文件，以便它们可以找到 ``Paths`` 配置文件: 

- ``/spark`` 运行命令行应用程序；路径是在第36行或附近指定的::

        require 'app/Config/Paths.php';
        // ^^^ 如果移动应用程序文件夹，请更改此项


- ``/public/index.php`` 是您的Web应用程序的前端控制器；配置路径是在第16行或附近指定的:: 

        $pathsPath = FCPATH . '../app/Config/Paths.php';
        // ^^^ 如果移动应用程序文件夹，请更改此项


一个CodeIgniter安装程序运行多个应用程序
===============================================================

如果您想共享一个常见的CodeIgniter框架安装，以管理多个不同的应用程序，只需将位于应用程序目录内的所有目录放入其自己的（子）目录。

例如，假设您要创建两个应用程序，分别名为 "foo" 和 "bar" 。您可以这样构建应用程序项目目录::

	foo/app, public, tests and writable
        bar/app/, public, tests and writable
        codeigniter/system and docs

这将有两个应用程序 "foo" 和 "bar"，两个应用程序都有标准的应用程序目录和一个public文件夹，并共享一个公共的Codeigniter框架。

每一个应用程序内的 ``index.php`` 将指向自己的配置， ``.../app/Config/Paths.php`` 以及 ``$systemDirectory`` 内部的每个的那些变量将被设置来指向共享的公共 "system" 文件夹中。

如果任何一个应用程序都具有命令行组件，那么您还需要按照上面的指示在每个应用程序的项目文件夹中进行修改 ``spark``。
