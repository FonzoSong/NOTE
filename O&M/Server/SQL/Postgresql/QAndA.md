# 连接到postgresql for AWS RDS 时报错:FATAL: no pg_hba.conf entry for host 未配置允许远程连接

**错误原因 :**

未在 `pg_hba.conf` 中允许主机进行链接;

因为在RDS实例中无法直接更改postgresql的配置文件,但AWS提供类参数组来进行配置的修改

**解决方法:**
1. 创建一个新的参数组;
2. 更改`rds.force_ssl` 参数,确保值为 `0` (允许非SSl链接)
3. 更改参数`rds.allowed_hosts` 添加允许的ip地址范围
