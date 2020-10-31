<center>模型类字段
</center>

1. 字段是被定义在django.db.model.fields中的，但是出于方便的目的，他们被导入django.db.models，标准的使用是from django.db import models, 并且使用models.Field来获取

2. 字段的选项

    + 通用选项(通用选项主要是每个类型的字段都可以使用的选项)

        + null:如果设置为True，会将数据库中该字段的值设置为null，默认是False。

            ```sql
            这里顺便说一下null值和空值的区别：
            	1. 空值是不占用空间的，null值是占用空间的。
              2. not null字段是不能插入null值的，只能插入空值，也证实了上面的说法
              3. 使用not null进行查询的时候，效率会比使用null进行查询高，这是因为null值需要占用空间，也会参与到字段比较中去，会影响效率。B树索引是不会存储null值的，
              4. 在进行count()统计某列的记录数的时候，如果采用的NULL值，系统会自动忽略掉，但是空值是会进行统计到其中的。
              5. 判断NULL 用IS NULL 或者 IS NOT NULL, SQL语句函数中可以使用ifnull()函数来进行处理，判断空字符用=''或者 <>''来进行处理
              6. 对于MySQL特殊的注意事项，对于timestamp数据类型，如果往这个数据类型插入的列插入NULL值，则出现的值是当前系统时间。插入空值，则会出现 0000-00-00 00:00:00
              7. 对于字段类型为auto_increment的列，如果插入空值，系统会插入一个正整数序列。而不是null。
            ```

        + balck: 如果设置为True，表明该字段允许为空，默认是False。
            + 注意和null做区分，black是数据验证方面的，但是null是涉及数据库存储方面的。
            + black设置为True的时候，在进行表单验证的时候，该字段可以传一个空值，设置为False的时候，该字段必须有值。

        + choices: 指定字段的可选范围
            + 是由两元素元祖组成的序列。
            + 第一个元素表示需要存储到数据库中的值，第二个元素表示的是一个在表单中展示的可读性的名称
            + 对于已经设置了choice的字段，django将会自动生成一个获取可读性名称的方法 get_foo_display,foo需要使用字段名进行替换。
            + 其他可用的方法还有: get_next_by_foo:获取下一个选项，get_pervious_by_foo:获取前一个选项。仅针对日期字段并且是不允许为null的。
        + db_column;指定数据库中该字段的名称，默认情况下使用的是字段名，可以通过db_column来指定自己需要的名称。
        + db_index: 是否创建索引，如果设置为True，则将为该字段创建一个索引。
        + db_tablespace: 如果此字段已经建立了索引，则用于该字段索引的数据库表空间的名称。
        
        + default: 指定字段的默认值，可以是一个值或者是一个可调用的对象，如果是个可调用的对象，每次实例化模型类的时候都会调用该对象。
        + editable: 如果设置为Fasle，将不会在admin站点的模型中展示，即使已经注册，也会跳过模型类的验证，默认是True。
        + error_messages: 设置该字段验证错误的错误信息。
        + help_text: 额外的帮助文本。
        + perimary_field: 设置为True，该字段将成为模型类的主键。Django默认提供自增主键id，如果指定的新的主键，将不会自动生成id主键。
        + unique: 设置为True，则必须保持唯一。例如主键
        + Unique_for_datel: 用于DateField或者是DateTimeField，保证时间的唯一值。
        + unique_for_month、unique_for_year和unique_for_date一样，只是针对的范围不一样。
        + verbose_name：指定数据库字段展示的时候使用的名称。
        + validators: 指定需要在该字段上运行的验证器。是一个可调用对象的序列。
    
3. 字段类型


    1. AutoField: 本质上是一个IntegerField，将会根据id值进行自动的递增
    2. BigAutoField：存储64位整数，从1到9223372036854775807
    3. BigIntegerField:存储64位整数，从-9223372036854775808到9223372036854775807
    4. BinaryField: 存储二进制数据，并且editable默认是False
    5. BooleanField: 存储true或者是false
    6. CharField: 存储字符串，必须指定最大长度，将在数据库层面和模型类验证层面强制执行。
    7. DateField: 存储日期，是datetime.date()的实例。
        + auto_now：在执行save保存的时候自动保存当前的时间，其余方法比如update被调用的时候，该字段不会进行更新。
        + auto_new_add: 首次创建对象的时候，将其值设置为当前时间。即使在创建的时候，指定了值，也会被忽略。
    8. DateTimeField: 存储日期和时间，datetime.datetime()的实例。
    9. DecimalField: 小数字段
        + max_digits： 指定最大的位数
        + decimal_places：小数点的位置
    10. DurationField: 存储时间差，timedelta的实例。
    11. TimeField：存储时间值，datetime.time实例
    12. TextField: 存储大文本，指定max_length会影响textarea的宽度，但是在数据库层面不会受影响。
    13. URLField: 存储字符串形式的url，可以指定max_length
    14. UUIDField: 存储uuid

4. 关联关系字段


    1. ForeignKey:多对一关系:需要两个参数，被关联的类和on_delete选项
        + 如果需要创建一个自关联的关系，被关联的类需要使用'self'
        + 也可以指定一个还未定义的被关联的类，需要使用类名的字符串形式。
        + 要引用在另一个应用中定义的类，可以使用完整的应用名称label.模型类名的方式
        + 外键字段将会被自动创建索引，可以将db_index设置为False来禁用。
        + 默认创建的外键字段名称会带有_id，可以使用db_cloumn指定你想要的名称。
        + on_delete选项：
            + models.CASCADE：级联删除，某个数据被删除的时候，同时删除与其关联的数据
            + models.PROTECT: 通过引发一场来阻止删除。
            + Models.SET_NULL：将关联的外键字段的值设置为null，只有在允许null=True的时候生效。
            + models.SET_DEFAULT：将关联外键字段设为默认值，只有允许默认值的时候生效。
            + models.SET():执行删除的时候，调用对应的函数。
            + models.DO_NOTHING:什么都不做。
        + limite_choice_to：在进行表单或者是admin站点展示的时候，限制该字段的选项。
        + related_name：用于关联对象引用该对象的时候使用的值。
        + related_query_name：