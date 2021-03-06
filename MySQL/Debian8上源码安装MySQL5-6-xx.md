## Debian8上源码安装MySQL5.6.xx

Linux版本：Debian8.5<br />
MySQL版本：MySQL5.6.xx


1. 安装编译软件cmake,make

   ```shell
   # 安装cmake
   apt-get install -y cmake
   # 安装make
   apt-get install -y make
   ```

2. 获得MySQL所有版本(5.0.15-latest)地址传送门

   http://downloads.mysql.com/archives/community/

3. 创建MySQL根目录

   ```shell
   # 创建MySQL根目录
   mkdir -p /usr/local/mysql
   ```

4. 解压MySQL源码

   ```shell
   tar -zxvf mysql-5.6.xx.tar.gz
   ```

5. 创建mysql用户与用户组

   ```shell
   # 创建用户组
   groupadd mysql
   # 创建mysql用户，所属组为mysql
   useradd -s /bin/bash -m -g mysql mysql
   ```

6. 安装MySQL依赖包

   ```shell
   apt-get install -y libssl-dev libjemalloc-dev libncurses5-dev
   ```

7. 创建MySQL相关目录

   | 目录         | 含义                                     | 配置参数                                     |
   | :--------- | -------------------------------------- | ---------------------------------------- |
   | bin_log    | 二进制日志目录                                | log_bin_basename<br />log_bin_index      |
   | mydata     | 数据文件目录                                 | datadir                                  |
   | innodb_log | InnoDB重做日志目录                           | innodb_log_group_home_dir                |
   | innodb_ts  | InnoDB共享表空间目录                          | innodb_data_home_dir                     |
   | log        | 日志文件目录(error log+general log+slow log) | log_error<br />general_log_file<br />slow_query_log_file |
   | relay_log  | InnoDB中继日志目录                           | relay_log_basename<br />relay_log_index  |
   | tmpdir     | 临时文件目录                                 | tmpdir                                   |
   | undo_log   | InnoDB回滚日志目录                           | innodb_undo_directory                    |

   ```shell
   mkdir -p /data/mysql/3306/bin_log
   mkdir -p /data/mysql/3306/mydata
   mkdir -p /data/mysql/3306/innodb_log
   mkdir -p /data/mysql/3306/innodb_ts
   mkdir -p /data/mysql/3306/log
   mkdir -p /data/mysql/3306/relay_log
   mkdir -p /data/mysql/3306/tmpdir
   mkdir -p /data/mysql/3306/undo_log
   ```

8. 修改步骤6创建的目录所属用户与组为mysql

   ```shell
   chown -R mysql:mysql /data/mysql/3306
   ```

9. 将MySQL配置文件my-3306.cnf文件放置指定目录/etc

   ```shell
   # 将my.cnf文件放置指定目录
   mv ~/my.cnf /etc
   ```

10. 编译安装MySQL5.6.xx

```shell
   # 切换到源码目录
   cd ~/mysql-5.6.xx/
   # cmake
   cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql/3306/mydata -DSYSCONFDIR=/etc/my.cnf -DWITH_INNOBASE_STORAGE_ENGINE=1  -DWITH_PARTITION_STORAGE_ENGINE=1 -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DENABLE_DEBUG_SYNC=0 -DENABLED_LOCAL_INFILE=1 -DENABLED_PROFILING=1 -DMYSQL_TCP_PORT=3306 -DMYSQL_UNIX_ADDR=/data/mysql/3306/tmpdir/my-3306.sock -DWITH_DEBUG=0 -DWITH_SSL=yes -DCMAKE_EXE_LINKER_FLAGS="-ljemalloc" -DWITH_SAFEMALLOC=OFF
   # make
   make &&make install &&make clean
   #或者使用make -j2&&make install -j2&&make clean使用两个线程编译，数字"2"可以根据CPU核数进行指定
   # 修改MySQL根目录的所属用户与组
   chown -R mysql:mysql /usr/local/mysql  
```

11. 初始化MySQL 

```shell
   # 修改mysql_install_db脚本权限
   chmod 755 scripts/mysql_install_db
   # 初始化MySQL
   scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/data/mysql/3306/mydata --defaults-file=/etc/my-.cnf  --user=mysql
```

12. 添加MySQL环境变量

```shell
   vim ~/.bashrc

   # 在~/.bashrc文件下添加如下语句
   export MYSQL_HOME=/usr/local/mysql
   export PATH=${MYSQL_HOME}/bin:$PATH

   # 保存后，使环境变量生效
   source ~/.bashrc
```

13. 启动MySQL

```shell
   mysqld_safe --defaults-file=/etc/my.cnf &
```

14. 登陆MySQL

```shell
   mysql -uroot -S /data/mysql/3306/tmpdir/my-3306.sock -p
```

15. 处理MySQL安全漏洞

```shell
   # 删除匿名账户
   DELETE FROM mysql.user WHERE user='';

   # 更新root密码
   UPDATE mysql.user SET password=password('new_password') WHERE user='root';

   # 删除test数据库
   DROP DATABASE test；

   # 刷新权限
   FLUSH PRIVILEGES;
```

16. 关闭MySQL

```shell
   mysqladmin shutdown -uroot -S /data/mysql/3306/tmpdir/my-3306.sock -p
```

   ​

