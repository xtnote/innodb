
### CREATE DATABASE
```
CREATE DATABASE <database_name> [WITH [DURATION <duration>] [REPLICATION <n>] [SHARD DURATION <duration>] [NAME <retention-policy-name>]]
DROP DATABASE <database_name>
```
创建数据库

### SHOW DATABASES
显示所有数据库

### USE <db-name>
设置当前使用数据库


### Line Protocol
```
<measurement>[,<tag_key>=<tag_value>[,<tag_key>=<tag_value>]] <field_key>=<field_value>[,<field_key>=<field_value>] [<timestamp>]

cpu,host=serverA,region=us_west value=0.64
payment,device=mobile,product=Notepad,method=credit billed=33,licenses=3i 1434067467100293230
stock,symbol=AAPL bid=127.46,ask=127.48
temperature,machine=unit42,type=assembly external=25,internal=37 1434067467000000000
```  
空格敏感，不要加多余的空格  
measurement 至少有一个fields，可以有0到多个tags  
Timestamp,Measurements, tag keys, tag values, field keys不要打引号，他们总是字符串类型  
Field values字符串类型打引号，floats, integers, Booleans不要打引号  


### SELECT
```
SELECT <field_key>[,<field_key>,<tag_key>] FROM <measurement_name>[,<measurement_name>]
SELECT <field_key>::field,<tag_key>::tag`  #如果field和tag有相同的名字
FROM <database_name>.<retention_policy_name>.<measurement_name> #全限定名
```  
必须至少有一个field_key  
字符串最好都加上引号

### WHERE
```
SELECT_clause FROM_clause WHERE <conditional_expression> [(AND|OR) <conditional_expression> [...]]
```

### GROUP BY
```
SELECT_clause FROM_clause [WHERE_clause] GROUP BY [* | <tag_key>[,<tag_key]]
SELECT <function>(<field_key>) FROM_clause WHERE <time_range> GROUP BY time(<time_interval>),[tag_key] [fill(<fill_option>)]
SELECT <function>(<field_key>) FROM_clause WHERE <time_range> GROUP BY time(<time_interval>,<offset_interval>),[tag_key] [fill(<fill_option>)]
```

### LIMIT
```
SELECT_clause [INTO_clause] FROM_clause [WHERE_clause] [GROUP_BY_clause] [ORDER_BY_clause] LIMIT <N>
```
