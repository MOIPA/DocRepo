title: OracleSql+mybatis
author: tr
tags:
  - Sql
  - Oracle
categories:
  - Sql
  - Oracle
date: 2020-05-26 16:43:00
---
### OracleSql+mybatis

oracle sql与mysql sql 区别还是有的，复习sql

<!--more-->

#### 查询

1. case when 列变量转为其他值

```sql
select t.obj_id,
       t.graph_id,
       t.graph_name,
       case
           when graph_type = '2' then '黑图'
           when graph_type
               = '1' then '历史图'
           else '' end                 as graph_type,
       t.create_time,
       t.graph_version,
       (select count(1)
        from t_pw_tm_singlegraph x
        where x.graph_id = t.graph_id) as graphcount
from t_pw_tm_singlegraph t
where t.graph_type = '2'
  and t.graph_name like '%' || '' || '%'
order by create_time desc
```

2. decode 

decode 列名，匹配的值，转为的值，匹配，转为,.........，其他情况的值

```sql
select bsb.OUTAGE_TYPE, decode(bsb.DATA_SRC, 1, -1, 2, -2, 3, -3, 4, 4, null) DATA_SRC from bsb) tt
```

3. nvl()

`nvl(table1.name, 'test')` 值是否为空，空则返回test

4. 日期模糊查询

`where to_char(t.INSERT_TIME,'yyyy-mm-dd hh24:mi:ss') like '%2020%'`

5. resultMap

```xml
    <resultMap id="resultMapId" type="com.nari.domain.testVO">
        <result property="name" column="name" jdbcType="VARCHAR" typeHandler = "com.nari.core.mp.utils.AESEncryptHandler"/>
        <result property="age" column="age"/>
    </resultMap>
    <!-- resultMap对应的查询-->
    <select id="getFunc" resultMap='resultMapId' resultType="java.util.Map">
            select * ....
    </select>
```