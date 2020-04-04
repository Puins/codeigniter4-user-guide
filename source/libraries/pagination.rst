##########
分页类
##########

CodeIgniter提供了一个非常简单但灵活的分页库，该库主题简单，可与模型一起使用，并能够在单个页面上支持多个分页器。

*******************
加载类库
*******************

与CodeIgniter中的所有服务一样，可以通过进行 ``Config\Services`` 加载，尽管通常不需要手动加载::

    $pager = \Config\Services::pager();

***************************
分页数据库结果
***************************

在大多数情况下，将使用分页器库对从数据库中检索到的结果进行分页。使用 :doc:`Model </models/model>` 类时，可以使用其内置 ``paginate()`` 方法自动检索当前批次的结果，并设置Pager库，以便可以在控制器中使用它。它甚至通过 ``page=X`` 查询变量从当前URL读取应显示的当前页面。

为了在您的应用程序中提供用户的分页列表，控制器的方法应类似于::

    <?php namespace App\Controllers;

    use CodeIgniter\Controller;

    class UserController extends Controller
    {
        public function index()
        {
            $model = new \App\Models\UserModel();

            $data = [
                'users' => $model->paginate(10),
                'pager' => $model->pager
            ];

            echo view('users/index', $data);
        }
    }

在此示例中，我们首先创建UserModel的新实例。然后，我们填充数据以发送到视图。第一个元素是来自数据库 **users** 的结果，将针对正确的页面进行检索，每页返回10个用户。必须发送到视图的第二项是Pager实例本身。为了方便起见，模型将保留所使用的实例，并将其存储在公共类变量 **$pager** 中。因此，我们将其获取并将其分配给视图中的 **$pager** 变量。

然后，在视图内，我们需要告诉它在哪里显示结果链接::

    <?= $pager->links() ?>

这就是全部。Pager类将为当前页面两侧超过两个页面的任何页面呈现“首页”和“末页”链接，以及“下一页”和“上一页”链接。

如果您更喜欢简单的输出，则可以使用 ``simpleLinks()`` 仅使用“较旧”和“较新”链接而不是详细信息分页链接的方法::

    <?= $pager->simpleLinks() ?>

在后台，该库加载了一个视图文件，该文件确定链接的格式，从而可以轻松地根据需要进行修改。有关如何完全自定义输出的详细信息，请参见下文。

分页多个结果
===========================

如果需要提供来自两个不同结果集的链接，则可以将组名传递给多个分页方法，以使数据分开::

    // 在控制器中
    public function index()
    {
        $userModel = new \App\Models\UserModel();
        $pageModel = new \App\Models\PageModel();

        $data = [
            'users' => $userModel->paginate(10, 'group1'),
            'pages' => $pageModel->paginate(15, 'group2'),
            'pager' => $userModel->pager
        ];

        echo view('users/index', $data);
    }

    // 在视图中
    <?= $pager->links('group1') ?>
    <?= $pager->simpleLinks('group2') ?>

手动分页
=================

您可能会发现只需要根据已知数据创建分页的时间。您可以使用 ``makeLinks()`` 方法手动创建链接，该方法分别将当前页面，每页的结果数和项目总数作为第一个，第二个和第三个参数::

    <?= $pager->makeLinks($page, $perPage, $total) ?>

默认情况下，这将以正常方式将链接显示为一系列链接，但是您可以通过将模板名称作为第四个参数传入来更改使用的显示模板。在以下各节中可以找到更多详细信息。::

    <?= $pager->makeLinks($page, $perPage, $total, 'template_name') ?>

也可以使用URI片段作为页码，而不是页面查询参数。只需指定片段号用作 ``makeLinks()`` 的第五个参数。然后，由分页器生成的URI看起来像 *https://domain.tld/model/[pageNumber]* 而不是 *https://domain.tld/model?page=[pageNumber]*。::

<?= $pager->makeLinks($page, $perPage, $total, 'template_name', $segment) ?>

Please note: ``$segment`` 值不能大于URI片段的数量加1。

如果您需要在一页上显示很多分页器，那么定义组的其他参数可能会有所帮助::

	$pager = service('pager');
	$pager->setPath('path/for/my-group', 'my-group'); // 此外，还可以为每个组定义路径。
	$pager->makeLinks($page, $perPage, $total, 'template_name', $segment, 'my-group'); 

仅对预期查询进行分页
=====================================

默认情况下，所有GET查询都显示在分页链接中。

例如，当访问URL *http://domain.tld?search=foo&order=asc&hello=i+am+here&page=2* 时，可以生成页面3链接以及其他链接，如下所示::

    echo $pager->links();
    // Page 3 link: http://domain.tld?search=foo&order=asc&hello=i+am+here&page=3

``only()`` 方法允许您将其限制为仅已预期的查询::

    echo $pager->only(['search', 'order'])->links();
    // Page 3 link: http://domain.tld?search=foo&order=asc&page=3

*page* 查询默认情况下启用。并且 ``only()`` 在所有分页链接中起作用。

*********************
自定义链接
*********************

视图配置
==================

当链接呈现到页面时，它们使用视图文件来描述HTML。您可以通过编辑 **app/Config/Pager.php** 轻松地更改使用的视图::

    public $templates = [
        'default_full'   => 'CodeIgniter\Pager\Views\default_full',
        'default_simple' => 'CodeIgniter\Pager\Views\default_simple'
    ];

此设置存储应使用的视图的别名和 :doc:`命名空间路径 </outgoing/views>` 。*default_full* 和 *default_simple* 视图被分别用于 ``links()`` 和 ``simpleLinks()`` 方法。要更改在整个应用程序范围内显示的方式，您可以在此处分配一个新视图。

例如，假设您创建一个与Foundation CSS框架一起使用的新视图文件，然后将该文件放置在 **app/Views/Pagers/foundation_full.php** 中。由于应用程序目录的命名空间为 ``App``，并且其下的所有目录都直接映射到命名空间的各个部分，因此您可以通过其命名空间找到视图文件::

    'default_full'   => 'App\Views\Pagers\foundation_full',

但是，由于它位于标准的 **app/Views** 目录下，因此不需要命名空间，因为 ``view()`` 方法可以按文件名定位它。在这种情况下，您只需提供子目录和文件名::

    'default_full'   => 'Pagers/foundation_full',

创建视图并将其设置为配置后，将自动使用它。您不必替换现有模板。您可以在配置文件中根据需要创建任意数量的其他模板。常见的情况是您的应用程序的前端和后端需要不同的样式。::

    public $templates = [
        'default_full'   => 'CodeIgniter\Pager\Views\default_full',
        'default_simple' => 'CodeIgniter\Pager\Views\default_simple',
        'front_full'     => 'App\Views\Pagers\foundation_full',
    ];
 
配置完成后，你可以指定它作为 ``links()``, ``simpleLinks()``, 和 ``makeLinks()`` 方法中最后的参数::

    <?= $pager->links('group1', 'front_full') ?>
    <?= $pager->simpleLinks('group2', 'front_full') ?>
    <?= $pager->makeLinks($page, $perPage, $total, 'front_full') ?>

创建视图
=================

创建新视图时，只需创建分页链接本身所需的代码。您不应创建不必要的包装divs，因为它可能会在多个地方使用，并且限制了它们的用途。通过向您显示现有的default_full模板，最容易演示创建新视图::

    <?php $pager->setSurroundCount(2) ?>

    <nav aria-label="Page navigation">
        <ul class="pagination">
        <?php if ($pager->hasPrevious()) : ?>
            <li>
                <a href="<?= $pager->getFirst() ?>" aria-label="First">
                    <span aria-hidden="true">First</span>
                </a>
            </li>
            <li>
                <a href="<?= $pager->getPrevious() ?>" aria-label="Previous">
                    <span aria-hidden="true">&laquo;</span>
                </a>
            </li>
        <?php endif ?>

        <?php foreach ($pager->links() as $link) : ?>
            <li <?= $link['active'] ? 'class="active"' : '' ?>>
                <a href="<?= $link['uri'] ?>">
                    <?= $link['title'] ?>
                </a>
            </li>
        <?php endforeach ?>

        <?php if ($pager->hasNext()) : ?>
            <li>
                <a href="<?= $pager->getNext() ?>" aria-label="Previous">
                    <span aria-hidden="true">&raquo;</span>
                </a>
            </li>
            <li>
                <a href="<?= $pager->getLast() ?>" aria-label="Last">
                    <span aria-hidden="true">Last</span>
                </a>
            </li>
        <?php endif ?>
        </ul>
    </nav>

**setSurroundCount()**

在第一行中，``setSurroundCount()`` 方法指定了我们要显示到当前页面链接两侧的两个链接。它接受的唯一参数是要显示的链接数。

**hasPrevious()** & **hasNext()**

如果根据传递给 ``setSurroundCount`` 的值，如果当前页面的任何一侧上可以显示更多链接，则这些方法将返回布尔值true。例如，假设我们有20页数据。当前页面是第3页。如果周围的计数是2，则以下链接将显示在列表中：1、2、3、4和5。由于显示的第一个链接是第1页，``hasPrevious()`` 因此从此处返回 **false**。没有零页。但是，``hasNext()`` 将返回true，因为在第5页之后还有15个额外的结果页。

**getPrevious()** & **getNext()**

这些方法返回编号链接两侧上一页或下一页结果的URL。有关完整说明，请参见上一段。

**getFirst()** & **getLast()**

与 ``getPrevious()`` 和 ``getNext()`` 类似，这些方法返回指向结果集中第一页和最后一页的链接

**links()**

返回有关所有编号链接的数据数组。每个链接的数组都包含该链接的uri，标题（仅是数字）和一个布尔值，该布尔值指示该链接是否为当前/活动链接::

	$link = [
		'active' => false,
		'uri'    => 'http://example.com/foo?page=2',
		'title'  => 1
	];
