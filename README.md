# SQL（MGTF495）Conclusion

1.从“Airbnb_listings”表中选择d, name, space, city, price，条件是room_type为“Private room”,
且space不为null，且price在去除美元符号和逗号后，转换为数值类型是小于100的。

select id, name, space, city, price
from "Airbnb_listings"
where room_type = 'Private room' and
space is not null and ltrim(regexp_replace(price, ',', ''), '$')::numeric < 100

regexp_replace(price, ',', '')：这是一个正则表达式替换函数，作用是将price字段中的所有逗号(,)替换为空字符，也就是删除所有逗号。

ltrim(…, '$')：ltrim函数用来去除字符串左侧的特定字符，在这里是用来去除字符串左侧的美元符号($)。

::numeric：这是 PostgreSQL 中的类型转换操作符，用于将字符串转换成数值类型。如果转换前的字符串不是有效的数值表示，转换会失败。

