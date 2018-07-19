```
mvn clean && mvn package -P$1 && java -jar target/juwanEurekaServer.jar --spring.profiles.active=$2
```



mvn package -P(resources目录下的文件),决定了从哪个文件中导入资源文件



spring.profiles.active=peer2,制定了导入不同的开发环境下的配置文件



两者结合或者单独都可以实现，不同的开发环境的配置





在Resources文件夹下建立自己的resource文件夹，不同的开发环境：

![1520994938(1)](C:\Users\zhengwentian\Desktop\知识图集\1520994938(1).png)

)

然后需要配置在pom文件中配置profiles属性:

![1520994979(1)](C:\Users\zhengwentian\Desktop\知识图集\1520994979(1).png)

同时还需要在build中加入以下的build信息:

![1520994397(1)](C:\Users\zhengwentian\Desktop\知识图集\1520994397(1).png)

本地run jar包的时候使用:



Java -jar -Ptest ....jar运行

在某个环境下继续使用某个配置文件使用：

--spring.profiles.active=dev来设置跑的环境！



https://blog.csdn.net/MassiveStars/article/details/53510586，不同环境下的配置方式。资源文件