# 清除n天前的日志

```shell
find logs/ -type f -mtime +n -exec rm -f {} \;
```

 

