# 安装

> 操作系统: CentOS 7.6
>
> MySQL: 5.7

```shell
# 安装依赖
yum install perl-DBI perl-DBD-MySQL perl-Digest-MD5 perl-IO-Socket-SSL perl-TermReadKey

# 上一步有可能报如下警告
warning: /var/cache/yum/x86_64/7/mysql57-community/packages/mysql-community-libs-compat-5.7.40-1.el7.x86_64.rpm: Header V4 RSA/SHA256 Signature, key ID 3a79bd29: NOKEY
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

The GPG keys listed for the "MySQL 5.7 Community Server" repository are already installed but they are not correct for this package.
Check that the correct key URLs are configured for this repository.

# 遇见如上告警执行如下语句
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022

# 再次安装依赖
yum install perl-DBI perl-DBD-MySQL perl-Digest-MD5 perl-IO-Socket-SSL perl-TermReadKey

# 下载 percona-toolkit
wget https://downloads.percona.com/downloads/percona-toolkit/3.4.0/binary/redhat/7/x86_64/percona-toolkit-3.4.0-3.el7.x86_64.rpm

# 安装
rpm -iv percona-toolkit-3.4.0-3.el7.x86_64.rpm

# 分析慢查询日志
pt-query-digest /var/lib/mysql/slow_query_log.log
```

# 使用

参考：https://blog.51cto.com/liang3391/1249348

```shell
pt-query-digest --report --since '2021-06-08 16:58:00' slow-query.log > slow_query_statistic20210608.log

pt-query-digest --report --since '2021-06-08 16:58:00' --limit 20 --filter '($event->{user} || "") =~ m/^wecomuser/i' slow-query06091010.log > slow_query_statistic.log
```







