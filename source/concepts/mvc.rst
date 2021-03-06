##############################
模型，视图和控制器
##############################

当创建一个应用的时候，我们需要有一种便捷的代码结构。和很多 Web 框架类似， CodeIgnite 框架也使用了模型、视图、控制器结构，即 MVC 模式，来组织结构代码文件。这种方式可以将数据，展示部分和流程部分分别作为单独的部分存放在我们的应用中。需要注意的是，可能每个人会对某个元素所担任的角色有不同的看法。如果你有不同的想法，你可以自由使用自己的方式修改你需要的每个文件。

**Models** 管理应用程序的数据，并帮助实施应用程序可能需要的任何特殊业务规则。

**Views** 是一个几乎没有逻辑的简单的文件，只负责将数据展示给用户。

**Controllers** 充当拼接代码的功能, 它主要在视图层（或正在查看它的用户）和数据存储之间来回的处理并整合数据。

从根本上讲，控制器和模型只是具有特定工作的类。 他们虽然不是你可以使用的唯一类的类型，但他们是构成整个框架的核心。 他们甚至已在 **/app** 目录中指定了用于存储它们的目录，尽管您可以自由存储它们，只要您有适当的命名空间，就可以在任何需要的地方使用。 我们将在下面更详细地讨论。

让我们仔细看一下这三个主要组成部分。

**************
组成部分
**************

Views
=====

视图是最简单的文件，一个视图文件通常是一个HTML文件加入少量的PHP代码。视图中的PHP代码应该尽可能的简单，一般只是显示一个变量内容，或者通过循环语句将数据输出在表格中展示出来。

视图从控制器中获取数据并展示——控制器将数据发送给视图，视图通过简单的 ``echo`` 调用将数据展示出来。你也可以在一个视图中插入展示其他视图，这样可以很简单的在每个页面上展示出公共的页眉和页脚。

视图文件通常存放在 **/app/Views** 目录下，如果在创建文件时不按照一定的规则创建的话，会显得我们的代码杂乱无章。 CodeIgnite 框架虽然没有规定任何的规则，但通过经验我们规定在 **Views** 目录下创建一个新的目录对应每个控制器。然后通过方法名来命名视图。这样就会使我们之后查找起来更加容易。例如：``用户配置`` 可能会显示在一个名为 ``User`` 的控制器中,并且方法名称为 ``profile`` ，你就可以将该视图文件保存在 **/app/Views/User/Profile.php** 这个路径下，并这样命名。

这种良好的组织代码方式建议养成一个习惯。可能有些时候，你有一些其他需求需要以其他方式来组织代码，没关系，只要CodeIgnite框架可以找到这个文件，这个视图就会被显示。

:doc:`了解有关视图的更多信息 </outgoing/views>`

Models
======

模型的主要任务是给应用维护单一类型的数据。比如：用户，博客内容，交易信息等。所以，模型的工作有以下两种，对数据进行采集或者放入数据库中执行业务规则；检索数据并将数据库中的数据读取出来。也就是进行数据的增删改查的操作。

对于许多开发人员而言，在确定执行哪些业务规则时会产生混乱。 数据的任何限制和要求都由模型层承担，包括在保存数据前将原始数据初始化，或者在数据传给控制器前将数据格式化。这样可以保证你可以不用在多个控制器中出现重复代码，或者遗漏更新区域。


模型类型的文件保存在 **/app/Models** 这个目录下，虽然他们也可以使用一个命名空间分组，但是还是建议你将模型文件放在这个目录下。

:doc:`了解有关模型的更多信息 </models/model>`

Controllers
===========

控制器扮演着两个不同的角色。 最明显的是，他们会接收用户的请求，然后判断这个请求应该执行什么样的操作。 而这一过程通常会涉及到将数据发送给模型层保存，或者去请求模型层的数据返回给视图。控制器也会用来加载其他应用程序请求的除模型参与的任务。

控制器的另一职责是处理与HTTP请求有关的所有内容-重定向，身份验证，网络安全，编码等。总而言之，控制器是你的应用程序的入口。在那里，他们才可以到达指定的地方并获取他们想要的数据使用格式。

控制器通常会保存在 **/app/Controllers** 这个路径下, 虽然你也可以使用命名空间分组，但是还是建议你将控制器存放在该目录下。

:doc:`了解有关控制器的更多信息 </incoming/controllers>`
