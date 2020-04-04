=====================
Honeypot 类
=====================

如果在 ``App\Config\Filters.php`` 文件中启用了Honeypot类，可以通过Honeypot类，确定Bot何时向CodeIgniter4应用程序发出请求。 这是通过将表单字段附加到任何表单来完成的，并且此表单域对人类是隐藏的，但Bot可以访问。 在字段中输入数据后，假设请求来自Bot，则可以引发 ``HoneypotException``。

.. contents::
    :local:
    :depth: 2

启用 Honeypot
=====================

要启用Honeypot，必须对 ``app/Config/Filters.php`` 进行更改。 只是取消 ``$globals`` 数组中的honeypot的注释即可，像这样...::

    public $globals = [
            'before' => [
                'honeypot'
                // 'csrf',
            ],
            'after'  => [
                'toolbar',
                'honeypot'
            ]
        ];

捆绑一个 ``Honeypot过滤器`` 示例，如 ``system/Filters/Honeypot.php``。如果不合适，请在 ``app/Filters/Honeypot.php`` 中创建您自己的文件，并在配置中适当地修改 ``$aliases``。

自定义 Honeypot
=====================

Honeypot 可以被自定义。 以下字段可以设置　``app/Config/Honeypot.php`` 或者 ``.env``　中。

* ``hidden``   - true|false 控制　``honeypot``　字段的可见性; 默认为 ``true``
* ``label``    - 　``honeypot``　字段的HTML标签, 默认为 'Fill This Field'
* ``name``     - 模板使用的HTML表单字段的名称; 默认为 'honeypot'
* ``template`` - 用于 ``honeypot`` 的表单字段模板; 默认为 '<label>{label}</label><input type="text" name="{name}" value=""/>'
