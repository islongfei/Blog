 ```
 ID列：数据为一组数字表示执行select语句的顺序 
      id值相同的时候执行顺数由上至下
      id值越大优先级越高，越先被执行
      ID列中的如果数据为一组数字，表示执行SELECT语句的顺序；如果为NULL，则说明这一行数据是由另外两个SQL语句
      进行 UNION操作后产生的结果集
      
 SELECT_TYPE |_ SIMPLE不包含子查询或者是UNION操作的查询
             |_ PRIMARY 查询中如果包含任何子查询，那么最外层的查询则被标记为 PRIMARY
             |_ SUBQUERY select 列表中的子查询
             |_ DEPENDENT 依赖外部结果的子查询
             |_ SUBQUERY 依赖外部结果的子查询 
             |_ UNION Union操作的第二个或是之后的查询的值为union
             |_ DEPENDENT UNION 当UNION作为子查询时，第二或是第二个后的查询的select_type值
             |_ UNION RESULT UNION产生的结果集
             |_ DERIVED  出现在FROM子句中的子查询
 
 TABLE  |_ 所在表的名称，如果表取了别名，则显示的是别名
        |_ <union M,N>： 由ID为M,N查询union产生的结果集
        |_ <derived N>/<subquery N> ：由ID为N的查询产生的结果
        
 PARTITIONS 分区表 
            查询匹配的记录来自哪一个分区
            对于分区表，显示查询的分区ID
            对于非分区表，显示为NULL
 
 TYPE列
      |_ system 	这是const联接类型的一个特例，当查询的表只有一行时使用 
      |_ const 	表中有且只有一个匹配的行时使用，如对主键或是唯一索引的查询，这是效率最高的联接方式
      |_ eq_ref 	唯一索引或主键索引查询，对应每个索引键，表中只有一条记录与之匹配
      |_ ref 	非唯一索引查找，返回匹配某个单独值的所有行
      |_ ref_or_null 	类似于ref类型的查询，但是附加了对NULL值列的查询
      |_ index_merge 	该联接类型表示使用了索引合并优化方法
      |_ range 	索引范围扫描，常见于between、>、<这样的查询条件
      |_ index 	FULL index Scan 全索引扫描，同ALL的区别是，遍历的是索引树
      |_ ALL 	FULL TABLE Scan 全表扫描，这是效率最差的联接方式
      

        
  POSSIBLE_KEYS列
       |_    指出MySQL能使用哪些索引来优化查询
       |_    查询列所涉及到的列上的索引都会被列出，但不一定会被使用

  KEY列
       |_查询优化器优化查询实际所使用的索引
       |_如果表中没有可用的索引，则显示为NULL
       |_如果查询使用了覆盖索引，则该索引仅出现在Key列中

  KEY_LEN列
       |_ 显示MySQL索引所使用的字节数，在联合索引中如果有3列，假如3列字段总长度为100个字节
          Key_len显示的可能会小于100字节，比如30字节
          这就说明在查询过程中没有使用到联合索引的所有列，只是利用到了前面的一列或2列
       |_ 表示索引字段的最大可能长度 
       |_ Key_len的长度由字段定义计算而来，并非数据的实际长度
       
  Ref列
       |_ 表示当前表在利用Key列记录中的索引进行查询时所用到的列或常量
      
       
  rows列
  
       |_ 表示MySQL通过索引的统计信息，估算出来的所需读取的行数（关联查询时，显示的是每次嵌套查询时所需要的行数
       |_ Rows值的大小是个统计抽样结果，并不十分准确
  
  Filtered列 
       |_ 表示返回结果的行数占需读取行数的百分比
       |_ Filtered列的值越大越好（值越大，表明实际读取的行数与所需要返回的行数越接近）
       |_ Filtered列的值依赖统计信息，所以同样也不是十分准确，只是一个参考值  
       
       
  Extra列
      |_ Distinct 	优化distinct操作，在找到第一个匹配的元素后即停止查找
      |_ Not exists 	使用not exists来优化查询
      |_ Using filesort 	使用额外操作进行排序，通常会出现在order by或group by查询中
      |_ Using index 	使用了覆盖索引进行查询
      |_ Using temporary 	MySQL需要使用临时表来处理查询，常见于排序，子查询，和分组查询
      |_ Using where 	需要在MySQL服务器层使用WHERE条件来过滤数据
      |_ select tables optimized away 	直接通过索引来获得数据，不用访问表，这种情况通常效率是最高的
       
  执行计划的限制
      |_ 无法展示存储过程，触发器，UDF对查询的影响
      |_ 无法使用EXPLAIN对存储过程进行分析
      |_ 早期版本的MySQL只支持对SELECT语句进行分析
   ```



上面说了 SQL 的执行计划，通过 `Show Profile 语句分析 SQL 执行性能`,可以显示 SQL 各种开销所花的时间，注意是5.0.37版本之后才支持。
