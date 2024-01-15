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
  
