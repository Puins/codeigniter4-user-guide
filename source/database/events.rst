###############
数据库事件
###############

数据库类包含一些 :doc:`Events </extending/events>`，您可以利用这些事件来了解有关数据库执行期间正在发生的事情的更多信息。这些事件可用于收集数据以进行分析和报告。在 :doc:`调试工具栏 </testing/debugging>` 使用此收集查询在工具条显示。

==========
事件
==========

**DBQuery**

每当运行新查询（无论是否成功）时，都会触发此事件。唯一的参数是当前查询的 :doc:`查询 </database/queries>` 实例。您可以使用它在STDOUT中显示所有查询，或记录到文件，甚至创建工具进行自动查询分析，以帮助您发现可能丢失的索引，查询速度慢等。示例用法可能是::

    // 在 Config\Events.php 文件中
    Events::on('DBQuery', 'CodeIgniter\Debug\Toolbar\Collectors\Database::collect');

    // 收集所有的查询语句以备后来所需
    public static function collect(CodeIgniter\Database\Query $query)
    {
        static::$queries[] = $query;
    }
