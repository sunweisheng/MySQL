# 安装PostgreSQL

操作系统是Ubuntu 24.04.2 LTS桌面版

````shell
# 安装
sudo apt install postgresql postgresql-contrib

# 启动服务
sudo systemctl start postgresql

# 设置开机自动启动
sudo systemctl enable postgresql

# 查看服务状态
sudo systemctl status postgresql

# 验证是否安装成功
sudo -u postgres psql -c "SELECT version();"
                                                               version                                                                
--------------------------------------------------------------------------------------------------------------------------------------
 PostgreSQL 16.8 (Ubuntu 16.8-0ubuntu0.24.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0, 64-bit
(1 row)

# 切换到 postgres 用户（默认管理员）
sudo -i -u postgres

# 进入 PostgreSQL 交互终端
psql

# 修改 postgres 用户密码（在 psql 终端内）
ALTER USER postgres WITH PASSWORD 'your_password';

# 退出 psql
\q

# 退出 postgres 用户
exit

# 进入配置文件目录修改配置允许远程访问
cd /etc/postgresql/16/main/

# 修改配置文件
sudo nano postgresql.conf 
````

修改内容
````ini
listen_addresses = '*'  # 默认是 'localhost'
````

````shell
# 修改另一个配置文件
sudo nano pg_hba.conf 
````
添加内容
````ini
host    all             all             0.0.0.0/0               md5
````

````shell
# 重启服务
sudo systemctl restart postgresql
````