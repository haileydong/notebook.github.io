# mysql必知必会

# 一、概览

select子句顺序：

​	select>from>where>group by>having>order by >limit

使用mysql时：

- source .sql文件时，先创建脚本，再填充脚本
- `use ****(name)`选择要使用的数据库

               - `show databases;`：返回可用数据库的一个列表
               - `show tables;`:获得数据库内表的列表
               - `show columns from customers;`:返回表中的字段信息

> auto_increment自动增量：表定义的组成部分，添加后mysql会自动分配下一行的编号

# 二、检索语句（select）

- `select prod_name from products;`:检索某表中的某列

- `select prod_id, prod_name, prod_price from products;`:选择多列时，列名之间加逗号，但最后一个列名之后不加。

- `select prod_name from products;`显示所有列，能检索出名字未知的列，但会降低效率

- `SELECT DISTINCT vend_id FROM products;`distinct关键字只返回不同的值，必须放在列名的前面。distinct应用于所有列，不止它后面的一列；同时，它只能放在所有列的开头。

  >**限制语句limit**
  >
  >`SELECT prod_name FROM products LIMIT 5;`限制语句，返回不多于5行
  >
  >` SELECT prod_name FROM products LIMIT a,n;`从第a行开始，返回n行。
  >
  >note:行号从0开始


# 三、排序检索语句（order by) 

> 如果不明确规定排序顺序，则不应该假定检索出的数据的顺序有意义

- `SELECT prod_name FROM products ORDER BY prod_name;`排序

- `select prod_id, prod_price, prod_name from products order by prod_price, prod_name;`多行组合排序，先排第一个，再拍第二个

- ``SELECT prod_name FROM products ORDER BY prod_name asc;`升序（默认）

- ``SELECT prod_name FROM products ORDER BY prod_name desc;`降序。多列排序时，desc关键字只应用到直接位于其前面的列名。

  > `select prod_price from products order by prod_price desc limit 1;`用order by 和limit结合，可以找出最大最小值。
  >
  > order by 位于 from之后，limit之前，order by 是select语句中的最后一条子句

# 四、过滤语句（where)

## 1.where 子句

- 数据根据where子句中指定的搜索条件进行过滤，where子句在表名（from子句）之后给出
- `select prod_name, prod_price from products where prod_price = 2.50;`

> 数据也可以在应用层过滤，这种实现并不令人满意

| 操作符       | 说明               |
| ------------ | ------------------ |
| =            | 等于               |
| <>           | 不等于             |
| !=           | 不等于             |
| <            | 小于               |
| <=           | 小于等于           |
| >            | 大于               |
| >=           | 大于等于           |
| BETWEEN  AND | 在指定的两个值之间 |

- `select prod_name, prod_price from products where prod_price between 5 and 10;`

- `select prod_name from products where prod_price is null;`检查空值

> 一般要验证

## 2.组合where子句(and,or,in,not)

**操作符operator**：用来联结或改变where子句的关键字，也称为逻辑操作符

- `select prod_id, prod_price, prod_name from products where vend_id = 1003 and prod_price <=10;`

- `select prod_price, prod_name from products where vend_id = 1003 or vend_id = 1002;`

  > where可包含任意数目的and和or操作符，允许两者结合以进行复杂和高级的过滤，但是优先处理and操作符。任何时候使用具有and和or操作符的where子句，都应该使用圆括号明确计算顺序，不过分依赖默认计算次序，以消除歧义

- `select prod_price, prod_name from products where vend_id in (1002,1003) order by prod_name;`  in操作符用来指定条件范围，范围内的每个条件都可以进行匹配，in取值由逗号分隔，全都括在圆括号中

  >in 与 or 由相同的功能，但在多个条件时更清楚直观，且执行更快
  >
  >In最大的优点是可以包含其他select语句，使得能够更动态地建立where子句**（？）**

- `select prod_price, prod_name from products where vend_id not in (1002,1003) order by prod_name;` not用来否定后跟的条件 

> mysql支持使用not对in、between和exists取反

## 3.用通配符进行过滤（%_）

前面介绍的所有操作符都是针对已知值进行过滤的，不管是匹配一个还是多个值，测试大于还是小于已知值，共同点是过滤中使用的值都是已知的，但是如果未知时呢？

**通配符wildcard**：用来匹配值的一部分的特殊字符

**搜索模式search pattern**：由值、通配符或两者组合构成的搜索条件

为在搜索子句中使用通配符，必须使用like操作符，like指示mysql，后跟的搜索模式利用通配符匹配而不是直接相等匹配

- `select prod_id prod_name from products where prod_name like 'jet%';`百分号%表示任何字符出现任意次数，除了一个或多个字符，%还能匹配0个字符。在结尾：匹配xx开头的词。在开头：匹配xx结尾的词。在开头加结尾：匹配含有xx的词。在中间：匹配xx开头，xxx结尾的词

  > 不能匹配值为NULL的行

- `select prod_id, prod_name from products where prod_name like '_ ton anvil';`下划线_只匹配单个字符，不能多也不能少。

  > 使用通配符处理的时间比其他搜索时间更长，能有其他方法就不用。
  >
  > 通配符位于搜索模式的开始处是最慢的

## 4.用正则表达式进行搜索

mysql支持正则表达式的部分，不区分大小写

- `select prod_name from products where prod_name regexp '1000' order by prod_name;`REGEXP后所跟的东西作为正则表达式处理

| 命令    | 说明                                              |
| ------- | ------------------------------------------------- |
| ^       | 在字符的开启处进行匹配                            |
| $       | 在字符的末尾处进行匹配                            |
| .       | 匹配任何字符（包括回车和新行）                    |
| [….]    | 匹配括号内的任意单个字符                          |
| [m-n]   | 匹配m到n之间的任意单个字符，例如[0-9],[a-z],[A-Z] |
| [^..]   | 不能匹配括号内的任意单个字符                      |
| a*      | 匹配0个或多个a,包括空,可以作为占位符使用.         |
| a+      | 匹配一个或多个a,不包括空                          |
| a?      | 匹配一个或0个a                                    |
| a1\| a2 | 匹配a1或a2                                        |
| a{m}    | 匹配m个a                                          |
| a{m,}   | 匹配m个或者更多个a                                |
| a{m,n}  | 匹配m到n个a                                       |
| a{,n}   | 匹配0到n个a                                       |

> like匹配整个字符串，regexp匹配任意子集
>
> mysql转义符是两个斜杠"\\\\\"

# 四、计算字段

> 只有数据知道哪些列是实际的表列，哪些列是计算字段。从客户机（应用程序）的角度来看，计算字段的数据是以与其他列的数据相同的方式返回的。

## 1.拼接字段（concat()函数）

**拼接(concatenate)**:将值联结到一起构成单个值

- `select concat(vend_name, '(', vend_country, ')') from vendors order by vend_name;`concat()拼接串，即把多个串连接起来形成一个较长的串，各个串之间用逗号分隔。

- `select concat(rtrim(vend_name), '(', rtrim(vend_country), ')') from vendors order by vend_name;`

  > RTrim()函数去掉值右边的所有空格
  >
  > LTrim()函数去掉串左边的所有空格
  >
  > Trim()函数去掉串左右两边的空格

## 2.算术计算

| 操作符 | 说明 |
| ------ | ---- |
| +      | 加   |
| -      | 减   |
| *      | 乘   |
| /      | 除   |

- `select prod_id, quantity, item_price, quantity*item_price as expanded_price from orderitems where order_num = 20005;`

## 3.使用数据处理函数

> 如果决定使用函数，做好代码注释，因为函数可移植性不强 

- 文本处理函数

| 函数        | 说明             |
| :---------- | ---------------- |
| left()      | 返回串左边的字符 |
| length()    | 返回串的长度     |
| locate()    | 找出串的一个子串 |
| lower()     | 将串转换为小写   |
| ltrim       | 去掉串左边的空格 |
| right()     | 返回串右边的字符 |
| rtrim()     | 去掉串右边的空格 |
| upper()     | 将串转换为答谢   |
| substring() | 返回子串的字符   |

- 日期和时间处理函数

  | 函数          | 说明                         |
  | ------------- | ---------------------------- |
  | adddate()     | 增加一个日期（天、周等）     |
  | addtime()     | 增加一个时间（时、分等）     |
  | date_add()    | 高度灵活的日期运算函数？     |
  | now()         | 返回当前的日期和时间         |
  | curdate()     | 返回当前日期                 |
  | curtime()     | 返回当前时间                 |
  | date()        | 返回日期时间的日期部分       |
  | time()        | 返回一个日期时间的时间部分   |
  | datediff()    | 计算两个日期之差             |
  | date_format() | 返回一个格式化的日期或时间串 |
  | year()        | 返回一个日期的年份部分       |
  | mouth()       | 返回日期的月份部分           |
  | day()         | 返回一个日期的天数部分       |
  | day0fweek()   | 返回一个日期的星期几         |
  | hour()        | 返回时间的小时部分           |
  | minute()      | 返回时间的分钟部分           |
  | second()      | 返回一个时间的秒部分         |

  > mysql插入的日期格式yyyy-mm-dd

`select cust_id,order_num from orders where date(order_date) = '2005-09-01';`比较日期时用date(),以防止值中不止包含日期

- 数值处理函数：仅处理数值数据

  | 函数   | 说明           |
  | ------ | -------------- |
  | abs()  | 返回绝对值     |
  | sqrt() | 返回平方根     |
  | exp()  | 返回指数值     |
  | mod()  | 返回余数       |
  | pi()   | 返回圆周率     |
  | rand() | 返回一个随机数 |
  | sin()  | 返回正弦       |
  | cos()  | 返回余弦       |
  | tan()  | 返回正切       |

## 4.聚集函数（高效）

**聚集函数(aggregate function)**:运行在行组上，计算和返回单个值的函数

| 函数    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| avg()   | 返回某列的平均值（忽略值为Null的行）                         |
| count() | 返回某列的行数（count*计算总行数，不忽略null；count（column）忽略null) |
| max()   | 返回某列的最大值（对文本数据，返回最后一行；忽略Null）       |
| min()   | 返回某列的最小值（对文本数据，返回第一行；忽略Null）         |
| sum()   | 返回某列值之和（忽略null)                                    |

# 五、分组数据

## 1.group by

分组允许把数据分为多个逻辑组，以便对每个组进行聚集计算

- `select vend_id, count(*) as num_prods from products group by vend_id;`
  * group by子句可以包含任意数目的列，即分组可以进行嵌套。如果进行了嵌套，则数据在最后规定的分组上进行汇总。
  * group by子句中列出的所有列必须是检索列或者有效的表达式，不能是聚集函数，如果select中使用了表达式，group by也要指定相同的表达式，不能使用别名。
  * 除聚集计算语句外，select中的每个列，都要在group by语句中给出
  * 如果有null，则null将作为一个分组返回，多个Null会分为一组
  * group by 必须在where之后，order by之前

## 2.过滤分组（having)

> where没有分组的概念,where在分组前过滤，having在分组后过滤
>
> having支持所有的where操作符，语法相同，可能关键字有差别

- `select cust_id, count(*) as orders from orders group by cust_id having count(*) >=2;`

# 六、子查询

## 1.利用子查询进行过滤

```
select order_num from orderitems where prod_id = 'TNT2';
select cust_id from orders where order_num in (20005,20007);

select cust_id from orders where order_num in (select order_num from orderitems where prod_id = 'TNT2');

select cust_name, cust_contact from customers where cust_id in (select cust_id from orders where order_num in (select order_num from orderitems where prod_id = 'TNT2'));
```

列必须匹配：在where 子句中使用子查询时，应该保证select语句具有与where子句中相同数目的列。通常，子查询将返回单个列与单个列匹配，但如果需要也可以使用多个列。

## 2.作为计算字段进行子查询

`select cust_name, cust_state, (select count(*) from orders where orders.cust_id = customers.cust_id)as orders from customers order by cust_name;`

# 七、联结

## 1.内部联结

`select vend_name, prod_name, prod_price from vendors, products where vendors.vend_id = products.vend_id order by vend_name, prod_name;`

用where 子句建立联结关系

> 笛卡尔积：由没有联结条件的表关系返回的结果称为笛卡尔积，检索出的行的数目就是第一个表的行数乘以第二个表的行数

`select vend_name, prod_name, prod_price from vendors inner join products on vendors.vend_id = products.vend_id;`与上面的语句效果相同，称为内部联结(等值联结)。

> 首选inner join on 语法进行内部联结

## 2.高级联结

- 自联结：自己联结自己
  * `select p1.prod_id, p1.prod_name from products as p1, products as p2 where p1.vend_id = p2.vend_id and p2.prod_id = 'DTNTR'`妙呀
- 自然联结
  * 自然联结是这样一种联结，你只能选择那些唯一的列
- 外部联结：包含在相关表中没有关联行的行
  * `select customers.cust_id, orders.order_num from customers left outer join orders on customers.cust_id = orders.cust_id;`左外部联结：从左边表中选择所有行。右外部联结：从右边表中选择所有行

# 八、组合查询

```
      select vend_id, prod_id, prod_price from products where prod_price <= 5
      union
      select vend_id, prod_id, prod_price from products where vend_id in (1001,1002);
```

union规则：

* 必须由两条或两条以上的select语句组成，语句之间用关键字union分隔
* 每个查询必须包含相同的列、表达式或聚集函数
* 列数据类型必须兼容
* union 自动去重，union all不去重
* 不允许使用多条order by 语句，只能用一条，一起排序

# 九、全文本搜索

## 1.fulltext索引

> 并非所有的引擎都支持全文本搜索。两个最常使用的引擎:MyISAM和InnoDB，前者支持全文本搜索，而后者不支持

- 启用全文本搜索支持：fulltext(xxx)在创建表时指定
- 进行全文本搜索：match()指定被搜索的列，against()指定要使用的搜索表达式

` select note_text from productnotes where match(note_text) against ('rabbit');`

- with query expansion:查询范围更广，还包含可能有关的词against ('rabbit' with query expansion)

## 2.布尔文本搜素

> 布尔方式即使没有fulltext索引也可以使用，但这是一种非常缓慢的操作

 `select note_text from productnotes where match(note_text) against ('heavy -rope*'in boolean mode);`匹配包含heavy但不包含任意以rope开始的词的行

- 全文本布尔操作符

  | 布尔操作符 | 说明                                                         |
  | ---------- | ------------------------------------------------------------ |
  | +          | 包含，词必须在                                               |
  | -          | 排除，词必须不出现                                           |
  | >          | 包含，而且增加等级值                                         |
  | <          | 包含，且减少等级值                                           |
  | ()         | 把词组成子表达式（允许这些子表达式作为一个组被包含、排序、排列等 ） |
  | ~          | 取消一个词的排序值                                           |
  | *          | 词尾的通配符                                                 |
  | ""         | 定义一个短语（与单个词的列表不一样，它匹配整个短语以便包含或排除这个短语） |

# 十、增删改

## 1.插入语句（insert）

- `insert into customers(cust_name, cust_address, cust_city, cust_state, cust_zip, cust_country, cust_contact, cust_email) values('Pep E. LaPew', '100 Main Street', 'Los Angeles', 'CA', '90046', 'USA', NULL, NULL);`

该语法可以省略允许空值和默认值的列

> insert 操作可能很耗时，可以用`insert low priority into`降低insert语句的优先级，update和delete语句也可以

插入多行：`values(),values();`

插入检索出的数据：insert select

## 2.更新数据（update）

`update customers set cust_email = 'elmer@fudd.com' where cust_id = 10005;`

更新多条语句用‘列=值’，逗号隔开

列=NULL则删除某列的值

## 3.删除数据（delete)

`delete from customers where cust_id = 10006;`delete不需要列名，删除整行。如果要删除整列，用update

> delete语句删除表的内容而不是表本身
>
> truncate table删除表，并新建一个表，要删除表中所有行，并该语法更快
>
> 使用强制实施引用完整性的数据库，mysql不允许删除具有与其他表相关联的数据的行

