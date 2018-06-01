```
#进入mysql数据库控制台，
mysql -u root -p 
mysql>use 数据库
mysql>set names utf8; （先确认编码，如果不设置可能会出现乱码，注意不是UTF-8） 
#然后使用source命令，后面参数为脚本文件（如这里用到的.sql）
mysql>source d:\wcnc_db.sql
```

mysql中的dump导出数据

mysql中的source导入数据

需要解决的问题是乱码的问题，需要提前使用set names utf8解决乱码的问题。

 mysqldump -h 10.200.169.92 -P6306 -u juwan -p juwan tb_juwan_item_brand --skip-lock-tables >   test.sql

mysqldump的用法。。。！！！导入，导出数据，方便快捷！