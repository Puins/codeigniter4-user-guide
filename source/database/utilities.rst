########################
实用工具
########################

数据库实用程序类包含有助于您管理数据库的方法。

.. contents::
    :local:
    :depth: 2

*******************
从结果中获取XML
*******************

**getXMLFromResult()**

此方法从数据库结果返回xml结果。您可以这样做::

    $model = new class extends \CodeIgniter\Model {
        protected $table      = 'foo';
        protected $primaryKey = 'id';
    };
    $db = \Closure::bind(function ($model) {
        return $model->db;
    }, null, $model)($model);

    $util = (new \CodeIgniter\Database\Database())->loadUtils($db);
    echo $util->getXMLFromResult($model->get());

它将获得以下xml结果::

    <root>
        <element>
            <id>1</id>
            <name>bar</name>
        </element>
    </root>
