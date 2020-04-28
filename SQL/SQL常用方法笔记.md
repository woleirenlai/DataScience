# SQL常用方法记录

## 1.json解析

**情景**

表中存储的优惠详情字段PROMOTION_DETAIL为：

[{"id":"1201202004070935","desc":"云闪付满100立减50（新户专享）","type":"CP01","spnsrId":"00010000","offstAmt":"5000"}]

存储为varchar 但是实际内容是json数组，可能是多个json数组，这里只有一个json数组，desc代表用券类型想取出desc以及offstAmt

**查询语句**

```sql
select       
s.PROMOTION_DETAIL,
REPLACE(REPLACE(REPLACE(JSON_EXTRACT (s.PROMOTION_DETAIL, '$[*].desc'), '"', ''),'[',''),']','') AS '用券类型',
REPLACE(REPLACE(REPLACE(JSON_EXTRACT (s.PROMOTION_DETAIL, '$[*].offstAmt'), '"', ''),'[',''),']','')/100 AS '用券金额'
from a_trace_his_supply s
where s.order_no = '1860062113252020042013010684'
```

**输出结果**

![01](https://github.com/woleirenlai/Images/blob/master/SQLnote/01.png)

**思路**

使用mysql自带的JSON_EXTRACT函数提取json数组，JSON_EXTRACT提取特定的key  
对于desc，使用(JSON_EXTRACT (s.PROMOTION_DETAIL, '$[*].desc')，但是提取出来为json值格式为["value"]，需要替换一个双引号  " 与两个中括号 [  ]  
对于offstAmt，与desc提取方法相同，但存储金额单位为分，需要/100.

**方法2**

```sql
SELECT
(detail ->> '$.item[0].desc') as name,
(detail ->> '$.item[0].offstAmt')/100 AS amt
FROM
  (SELECT order_no,
          CONCAT('{"item":', PROMOTION_DETAIL, '}') AS detail
   FROM a_trace_supply) s
WHERE s.order_no = '1860062113252020042013010684'
```

**输出结果**

![02](https://github.com/woleirenlai/Images/blob/master/SQLnote/02.png)

**思路**

将PROMOTION_DETAIL里的json数组，拼接成{"item":[{"id":"1201202004070935","desc":"云闪付满100立减50（新户专享）","type":"CP01","spnsrId":"00010000","offstAmt":"5000"}]} 的json对象，里面item对应一个json数组，里面包含了一个json对象，将大的json对象命名为detail然后解析里面的对应值  
使用 detail ->> '$.item[0].desc' 解析 'desc' 对应的值，同理offstAmt解析后需要转换单位为元

item[0] 表示第一个json数组里的第一个数组，.desc 表示取desc 对应的值

使用这个思路可以解决一个json对象里多个json数组的情况

对于存储多个json数组的数据：还是先拼接成一个大的json对象，变更item[0]中json数组的位置，item[1]取第二个json数组的内容，item[2]取json数组的内容。

但是不能在select中用concat(str) ->> '$.item[0].key'直接取值，同时取其他字段的时候，嵌套子查询然后连接大表取数。

**拓展链接**

[Json教程](https://www.runoob.com/json/json-tutorial.html)

[mysql解析json数组](https://www.sunjs.com/article/detail/4152f6fc6c1e426d9288d7197a3467f0.html)

## 2.SQL优化

优化查询：

使用explain分析

mysql联合索引 有多个索引时，最好使用两个一起关联，使用第二个索引一般不会走  
查询差别不大时，mysql不会走索引，例如一张表10w条数据，从中取3w，不会走索引，相当于从10个男的里面找男的出来一样

磁盘会根据数据量存储不同文件里，可以加一个子查询，走索引把关键数据查出来再关联sql会内部优化查询 根据数据量

## 3.空值

数据为空字符串 ‘“ 和 null 或者 [null]

## 4.日期时间规范化

DATE_FORMAT(date, '%Y/%m/%d')

输出：2020/4/30

TIME_FORMAT(time, '%H:%I:%s')

输出：13:01:03