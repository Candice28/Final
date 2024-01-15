# SQL（MGTF495）Conclusion

1.从“Airbnb_listings”表中选择d, name, space, city, price，条件是room_type为“Private room”,
且space不为null，且price在去除美元符号和逗号后，转换为数值类型是小于100的。

select id, name, space, city, price
from "Airbnb_listings"
where room_type = 'Private room' and
space is not null and ltrim(regexp_replace(price, ',', ''), '$')::numeric < 100

