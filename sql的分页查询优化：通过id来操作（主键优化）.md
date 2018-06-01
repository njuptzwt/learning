sql的分页查询优化：通过id来操作（主键优化）

```
 <select id="queryByPage" resultMap="ItemMap">
   SELECT * FROM tb_juwan_item AS t1
   JOIN (SELECT id FROM tb_juwan_item ORDER BY id desc LIMIT
   ${(pagenum-1)*pagesize}, 1) AS t2
   WHERE t1.id <![CDATA[ <= ]]>
   t2.id ORDER BY t1.id desc LIMIT ${pagesize};
</select>
```





商品更新的时候，应该要把缓存去掉。