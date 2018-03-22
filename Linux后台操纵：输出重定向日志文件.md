Linux后台操纵：输出重定向日志文件：

nohup sh run.sh >../juwan_log/syncNohup.out &



nohup+ 启动命令+ 输出重定向到+指定文件。../回到父目录....

nohup启动的是后台的服务

tail -f ../juwan_log/syncNohup.out 

查看文件的目录