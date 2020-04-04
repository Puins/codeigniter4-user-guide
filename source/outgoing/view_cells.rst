##########
子视图
##########

子视图允许你插入在控制器以外生成的 HTML 片段，但它只能调用指定类的方法，且该方法只能返回有效的 HTML 字符串内容。这个可调用方法可以是在 项目中自动加载器可以定位到的任何可访问类的任何方法，唯一的限制是该类的构造方法不可有必须传入的参数。使用这个功能后，对模块化代码有很好的帮助。::

    <?= view_cell('\App\Libraries\Blog::recentPosts') ?>

在这个示例中，会自动运行 ``App\Libraries\Blog`` 类的 ``recentPosts()`` 方法，该方法必须返回有效的 HTML 字符串。该方法可以是 静态方法，也可以是非静态方法。

子视图参数
---------------

您可以通过将第二个参数中的参数列表传递给方法来进一步优化调用。传递的值可以是键/值对的数组，或键/值对的逗号分隔字符串::

    // 传递数组参数
    <?= view_cell('\App\Libraries\Blog::recentPosts', ['category' => 'codeigniter', 'limit' => 5]) ?>

    // 传递字符串参数
    <?= view_cell('\App\Libraries\Blog::recentPosts', 'category=codeigniter, limit=5') ?>

    public function recentPosts(array $params=[])
    {
        $posts = $this->blogModel->where('category', $params['category'])
                                 ->orderBy('published_on', 'desc')
                                 ->limit($params['limit'])
                                 ->get();

        return view('recentPosts', ['posts' => $posts]);
    }

此外，可以在方法中使用与参数变量匹配的参数名称，以提高可读性。当以这种方式使用它时，必须始终在视图调用方法中指定所有参数::

    <?= view_cell('\App\Libraries\Blog::recentPosts', 'category=codeigniter, limit=5') ?>

    public function recentPosts(int $limit, string $category)
    {
        $posts = $this->blogModel->where('category', $category)
                                 ->orderBy('published_on', 'desc')
                                 ->limit($limit)
                                 ->get();

        return view('recentPosts', ['posts' => $posts]);
    }

子视图缓存
------------

您可以通过传递缓存数据的秒数作为第三个参数来缓存子视图的调用结果，默认将使用当前配置的缓存引擎。::

    // 视图将缓存 5 分钟
    <?= view_cell('\App\Libraries\Blog::recentPosts', 'limit=5', 300) ?>

您可以通过传递新名称作为第四个参数来提供自定义名称，以代替自动生成的名称::

    // 视图将缓存 5 分钟
    <?= view_cell('\App\Libraries\Blog::recentPosts', 'limit=5', 300, 'newcacheid') ?>
