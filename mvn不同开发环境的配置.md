```
mvn clean && mvn package -P$1 && java -jar target/juwanEurekaServer.jar --spring.profiles.active=$2
```



mvn package -P(resources目录下的文件),决定了从哪个文件中导入资源文件



spring.profiles.active=peer2,制定了导入不同的开发环境下的配置文件



两者结合或者单独都可以实现，不同的开发环境的配置