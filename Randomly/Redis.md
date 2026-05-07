## 介绍
Redis 将数据存储在内存中的数据库，提高数据IO速度，在服务和持久化数据库之间充当缓冲层(Cache)，默认以字符串存储数据

## 使用/语法
Redis的数据以键值对的形式存储

- Redis-cli：启动命令行界面
- Redis-cli --raw：启动并以原始形式展示内容(适合中文)

- SET KEY VALUE：声明一个键值对，String类型
- GET KEY：按键查询值
- DEL KEY：按键删除键值对
- EXISTS KEY： 按键查询键值对是否存在
 
- `KEYS *`： 查找所有键*尽量不要用，数据多直接爆炸*
- `KEYS *xx`: 查找以xx结尾的键
- `FLUSHALL`: 清除缓存(删除所有数据)

- TTL KEY：查看一个键的过期时间，-1表示没有设置
- EXPIRE KEY seconds： 设置一个键的过期时间
- SETEX KEY seconds VALUE：声明一个有过期时间的键值对

列表
- LPUSH\RPUSH：
- LPOP\RPOP：
- LRANGE：
- RRANGE：

