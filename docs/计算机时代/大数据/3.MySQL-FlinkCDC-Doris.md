# #MySQL -> Flink CDC -> Doris

​`查看mysql时区`

​​![image](assets/image-20240104100543-fnhl5ph.png)​

```sql
# 展示当前数据库时区
show variables like '%time_zone%';
```

当前配置：

​![image](assets/image-20240104100637-mssp6rm.png)​

修改同步任务，将时区设置为`Asia/Shanghai`​.
