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

select substring(host_url from 35 for 4)
from "Airbnb_listings"
where substring(host_url,35,2) = '12';


  
