mybatis和springboot的配置



mybatis_config.xml配置的是mybatis的基础配置



三大配置：

一个是type package aliase,包的别名



第二个是数据层Dao对象的映射Mapper。使用@MapperScan(“包名”)映射包，使得sqlsession对象可以通过代理直接使用dao对象进行数据库的操作



第三个是mapper.xml的配置，配置文件。需要映射dao的数据库操作，通过在mybatis_config.xml中配置

<mapper>

<package>

</package>

</mapper>

来实现mapper.xml的发现。



强大的返回值resulttype 和resultMap

数据源的配置。