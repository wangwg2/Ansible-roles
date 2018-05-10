## Ansible Role: Redis
安装redis

###### 介绍
redis（Remote Dictionary Server）是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

官方地址： https://redis.io/
官方文档地址：https://redis.io/documentation

###### 要求
此角色仅在RHEL及其衍生产品、Ububtu及衍生产品上运行。



###### 测试环境
* ansible `2.5.2`
* os `Centos 7.0 X64` / `Ubuntu 16.0.4`
* python `2.7.5`

###### 角色变量
* `redis_version`: `4.0.9`
* `redis_data_path`: `/opt/data/redis`
* `redis_dir_path`: `{{ redis_data_path }}/{{redis_port}}`
* `redis_logfile`: `{{ redis_dir_path }}/redis-server.log`
* `redis_pidfile`: `{{ redis_dir_path }}/redis.pid`
* `redis_dbfilename`: `dump.rdb`
* `redis_dbdir`: `{{ redis_dir_path }}/data`
* `{{ redis_dir_path }}/redis.conf`
  src: `templates/redis.conf.j2`
* `/etc/systemd/system/{{ redis_service }}.service`
  src: `templates/redis.service.j2`

@import "defaults/main.yml"

###### 依赖
java ruby

###### github地址
https://github.com/kuailemy123/Ansible-roles/tree/master/redis

###### Example Playbook
```yml
## 单实例安装
- hosts: node1
  roles:
    - { role: redis }

## 主从配置
- hosts: node1
  vars:
    - redis_master_host: '127.0.0.1'
    - redis_master_port: '6380'
  roles:
    - { role: redis, redis_port: 6380}
    - { role: redis, redis_port: 6381, redis_slave: true}
    - { role: redis, redis_port: 6382, redis_slave: true}

## 哨兵模式
- hosts: node1
  vars:
    - redis_master_host: '127.0.0.1'
    - redis_master_port: '6383'
  roles:
    - { role: redis, redis_port: 6383, redis_sentinel_port: 26383}
    - { role: redis, redis_port: 6384, redis_sentinel_port: 26384, redis_slave: true}
    - { role: redis, redis_port: 6385, redis_sentinel_port: 26385, redis_slave: true}

## 伪集群模式
- hosts: node1
  vars:
  - redis_cluster: true
  - redis_requirepass: '123456'
  roles:
  - { role: redis, redis_port: 6481}
  - { role: redis, redis_port: 6482}
  - { role: redis, redis_port: 6483}
  - { role: redis, redis_port: 6484}
  - { role: redis, redis_port: 6485}
  - { role: redis, redis_port: 6486, redis_cluster_replicas: '1 127.0.0.1:6481 127.0.0.1:6482 127.0.0.1:6483 127.0.0.1:6484 127.0.0.1:6485 127.0.0.1:6486'}

## 集群分布式模式
- hosts: node1
  vars:
  - redis_cluster: true
  - redis_requirepass: '123456'
  roles:
  - { role: redis, redis_port: 7000}
  - { role: redis, redis_port: 7003}

- hosts: node2
  vars:
  - redis_cluster: true
  - redis_requirepass: '123456'
  roles:
  - { role: redis, redis_port: 7001}
  - { role: redis, redis_port: 7004}

- hosts: node3
  vars:
  - redis_cluster: true
  - redis_requirepass: '123456'
  roles:
  - { role: redis, redis_port: 7002}
  - { role: redis, redis_port: 7005, redis_cluster_replicas: '1 172.19.204.246:7000 172.19.204.245:7001 172.19.204.244:7002 172.19.204.246:7003 172.19.204.245:7004 172.19.204.244:7005'}
```


----
@import "./redis_configure.md"