# SQL（MGTF495）Conclusion

1.从“Airbnb_listings”表中选择d, name, space, city, price，条件是room_type为“Private room”,
且space不为null，且price在去除美元符号和逗号后，转换为数值类型是小于100的。

select id, name, space, city, price
from "Airbnb_listings"
where room_type = 'Private room' and
  space is not null and ltrim(regexp_replace(price, ',', ''), '$')::numeric < 100

说明
regexp_replace(price, ',', '')：这是一个正则表达式替换函数，作用是将price字段中的所有逗号(,)替换为空字符，也就是删除所有逗号。

ltrim(…, '$')：ltrim函数用来去除字符串左侧的特定字符，在这里是用来去除字符串左侧的美元符号($)。

::numeric：这是 PostgreSQL 中的类型转换操作符，用于将字符串转换成数值类型。如果转换前的字符串不是有效的数值表示，转换会失败。

用函数来达到相同效果【创建一个名为 conv_price 的函数，它接受一个 text 类型的参数，并返回一个 numeric 类型的结果】
CREATE FUNCTION conv_price(text) RETURNS numeric
  AS 'select ltrim(regexp_replace($1, ',', ''), ''$'')::numeric;'
  LANGUAGE SQL
  IMMUTABLE
  RETURNS NULL ON NULL INPUT;

select distinct conv_price(weekly_price)
from "Airbnb_listings"

说明
IMMUTABLE：规定函数在相同的输入值上总是返回相同的结果


2.括号的重要性，SQL中，默认AND运算符的优先级高于OR运算符。这意味着，如果没有使用括号，此查询将会先评估AND条件，然后再将OR条件应用于整个结果集

select id, name, space, city, state, price
from "Airbnb_listings"
where room_type = 'Private room' and
  space is not null and (state = 'MD' OR state = 'DC')

3.substring

SELECT substring(listing_url from 35)    从 listing_url 字段的第35个字符开始截取直到字段的末尾
from "Airbnb_listings"       
where summary is null;

select substring(host_url from 35 for 4)   从 `host_url` 字段的第35个字符开始，截取长度为4的子串
from "Airbnb_listings"
where substring(host_url,35,2) = '12';     从 host_url 字段中从第35个字符开始截取长度为2的字符串，并且要等于12

4.Compare strings and substrings
（1）String可以使用等于（=）和不等于（!=）运算符来进行比较
（2）为了进行部分匹配，我们使用 LIKE 操作符。有两个保留字符用于此操作：

（3）'%' 用于匹配任意数量的字符
    '_' 用于匹配单个任意字符
如果这些字符出现在你的string value中，你需要使用（'\'）来转义

select id, description, notes, city, price from "Airbnb_listings"
where (description ilike '%bedroom%' OR description ilike '%br%') and
      (description ilike '%smoking%' OR notes ilike '%smoking%');


description like '%bedroom%'：这个条件匹配任何在 description 字段中包含 "bedroom" 文本的记录，不论 "bedroom" 前后是否有其他字符。例如，"Nice bedroom in the center" 和 "The room for rent is a part of a larger bedroom apartment" 都会匹配

description like 'bedroom%'："bedroom" 后可以有任意数量的字符，但之前不能有其他字符。例如，"Bedroom available for rent" 会匹配，但 "Nice bedroom in the center" 则不会匹配，因为 "bedroom" 前面有其他文字。

description like  '%bedroom'："bedroom" 前可以有任意数量的字符，但之后不能有其他字符

select id, description, notes, city, price 
from "Airbnb_listings" 
where description like '%walk%museum%';   这个条件用于筛选出那些 description 字段中含有 "walk" 和 "museum" 这两个词的记录，且这两个词之间可以有任意字符

select id, description, weekly_price 
from "Airbnb_listings" 
where weekly_price like '$_,%';           这个条件用于筛选 weekly_price 字段的值。$：代表货币金额的美元符号。'_'：代表一个任意字符，这里用来代表价格中千位的数字
                                          ','：直接表示价格中的逗号，用于格式上的逗号，如美元的表示法中的千位分隔符。 %：代表任意数量的字符，这里用来匹配价格的百位、十位和个位数字

5. 使用了连接运算符(||)来将不同的字段和字符串文字连接起来，以创建一个格式化的报告字符串。AS report将这个长字符串的结果重命名为report。

select 'Listing ID ' || id || ', located at ' || street || ', has a daily rate of ' || price ||
       ', a weekly rate of ' || weekly_price ||
       ' and a monthly rate of ' || monthly_price AS report
from "Airbnb_listings"
where weekly_price is not null and monthly_price is not null

6.函数
数值函数 (Numeric functions):
sqr(): 用于计算某个值的平方。
sqrt(): 计算给定数值的平方根。
abs(): 返回给定数值的绝对值。
round(): 将数值四舍五入到指定的小数位数。例如 SELECT round(3.14159, 2) 将返回 3.14。

字符串函数 (String functions):
trim(): 去除字符串两端的空格。例如 SELECT trim(' hello ') 将返回 'hello'。
lower(): 将字符串中的所有字符转换为小写。例如 SELECT lower('Hello') 将返回 'hello'。
upper(): 将字符串中的所有字符转换为大写。例如 SELECT upper('Hello') 将返回 'HELLO'。

类型转换函数 (Casting functions):
::numeric: 将数据转换为数值类型。例如，将字符串 '123' 转换为数值类型可以使用 SELECT '123'::numeric。

日期函数 (Date functions):
date '2001-09-28' + interval '1 hour': 将日期和时间间隔相加。这个例子将给2001年9月28日增加一个小时，所以结果会是2001年9月28日的1小时后。

用户自定义函数 (User defined functions):
conv_price(): 这可能是一个在上一张幻灯片中定义的函数，用于转换或计算价格。使用方式会根据函数的具体定义而变化，但一般会是类似 SELECT conv_price(price) 的形式，其中 price 是传递给函数的参数。

聚合函数 (Aggregate functions):
count(): 计算表中符合特定条件的行数。例如 SELECT count(*) FROM table_name 会返回表中的总行数。
sum(): 计算某列数值的总和。例如 SELECT sum(column_name) FROM table_name 会返回该列所有行的数值总和。
avg(): 计算某列数值的平均值。例如 SELECT avg(column_name) FROM table_name 会返回该列所有行的平均数值。
这些函数通常在 SELECT 语句中使用，可以直接作用于列名或者表达式。聚合函数特别的地方在于它们通常用在包含 GROUP BY 子句的查询中，来计算分组的结果

7.select city, property_type, room_type, count(*), round(avg(conv_price(price)),3) as avgPrice, round(stddev(conv_price(price)),3) as stddevPrice,
       max(conv_price(price)) as maxPrice, min(conv_price(price)) as minPrice
from "Airbnb_listings"
where price is not null
group by city, property_type, room_type
having count(*) > 5                           这个HAVING子句用于筛选那些分组后记录数大于5的组，这意味着只有当某个特定的城市、房产类型和房间类型组合拥有超过5个列表时，才会包含在结果中
order by city asc, property_type desc;

  
