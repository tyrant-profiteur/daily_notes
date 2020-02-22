## Oracle

## 1.to_char,to_number,to_date

to_number():将字符串转换为数值型的格式

- 使用TO_NUMBER()函数的时候，一定要确保所转换字段是可转换为数字的，比如字符串“20151008”是可以转换为数字20151008的，但是字符串“2015-10-08”不可以。如果你的字段中包含了字符串“2015-10-08”，而你还直接使用了TO_NUMBER()函数进行操作的话就会报“invalid number”的错。

to_char():

to_date():

## 2.根据查询结果显示自己要的内容

```sql
SELECT id,title,type,
	case type 
		WHEN 1 THEN 'type1' 
		WHEN 2 THEN 'type2' 
		ELSE 'type error' 
	END as newType 
FROM table;  
```

case 后面跟的列名得是表中的列名，when的条件严格区分数据类型

## 3.数据在同一列展示

使用 union 和 union all ，要求查询的字段个数相同