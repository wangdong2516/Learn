关联查询
过滤器的链式调用


N+1问题
执行原生的sql
模型继承
抽象基类




Manager.raw(raw_query, params=None, translations=None)  --> django.db.models.query.RawQuerySet实例。这个 RawQuerySet 能像普通的 QuerySet 一样被迭代获取对象实例

raw() 字段将查询语句中的字段映射至模型中的字段。

如果你需要执行参数化的查询，可以使用 raw() 的 params 参数:params 是一个参数字典。你将用一个列表替换查询字符串中 %s 占位符，或用字典替换 %(key)s 占位符（其中， key 理所应当由字典 key 替换

有时候，甚至 Manager.raw() 都无法满足需求：你可能要执行不明确映射至模型的查询语句，或者就是直接执行 UPDATE， INSERT 或 DELETE 语句。

这些情况下，你总是能直接访问数据库，完全绕过模型层。

对象 django.db.connection 代表默认数据库连接。要使用这个数据库连接，调用 connection.cursor() 来获取一个指针对象。然后，调用 cursor.execute(sql, [params]) 来执行该 SQL 和 cursor.fetchone()，或 cursor.fetchall() 获取结果数据。