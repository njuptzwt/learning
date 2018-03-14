Maven和Ant的打包区别：



首先项目的需求都是：

需要编译，将编译的文件放在指定的文件目录下，同时需要将资源文件载入，因为.class文件运行的时候需要资源文件。



/src/main/java 存放java的源代码，指定编译的源文件<sourceDirectory>

Maven的约定大于配置和Ant的手动打包机制还是有区别的，实现了半自动化。



```
<build>
   <finalName>juwanSyncService</finalName>
   <outputDirectory>bin</outputDirectory>
   <sourceDirectory>src/main/java</sourceDirectory>
   <testSourceDirectory>src/test/java</testSourceDirectory>
   <resources>
      <resource>
         <directory>src/main/resources/${env}</directory>
      </resource>
      <resource>
         <directory>src/main/resources</directory>
         <excludes>
            <exclude>test/*</exclude>
            <exclude>product/*</exclude>
            <exclude>dev/*</exclude>
         </excludes>
      </resource>
      <resource>
         <directory>src/main/java/</directory>
         <includes>
            <include>**/*.yml</include>
            <include>**/*.xml</include>
         </includes>
      </resource>
   </resources>

   <plugins>
      <plugin>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
   </plugins>
</build>

```

可以手工的去设置编译的目录和资源文件的目录，来实现编译，编译是把源文件编译成.class文件，并且载入了资源文件。通过测试之后，进行打包。



mvn clean  compile test  package最后生成的jar包在服务端部署。



Maven的通俗理解:



运行一条mvn clean package命令，Maven会帮你清除target目录，重新建一个空的，编译src/main/java类至target/classes，复制src/main/resources的文件至target/classes，编译src/test/java至target/test-classes，复制src/test/resources的文件至target/test-classes；然后运行所有测试；测试通过后，使用jar命令打包，存储于target目录。Maven做的事情一点也不少，只是都对用户隐蔽起来了，它只要求你遵循它的约定。



http://juvenshun.iteye.com/blog/293975.一篇容易理解的文章。