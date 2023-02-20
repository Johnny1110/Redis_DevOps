# 配置 啟動 操作 關閉 Redis

<br>

---

<br>


Redis 安裝好後，有一些基本的 shell script 可以使用：

<br>

| shell script | 說明 |
|  --| -- |
|redis-server | 啟動 redis|
|redis-cli| redis 命令客戶端|
|redis-benchmark| redis 基準測試工具|
|redis-check-aof| redis AOF 持久化文件檢測修復工具
|redis-check-dump| redis RDB 持久化文件檢測修復工具
|redis-sentinel| 啟動 redis sentinel


<br>
<br>
<br>
<br>

## 啟動 Redis

<br>

1. 默認配置啟動

    使用 Redis 默認配置啟動

    ```
    redis-server
    ```

<br>
<br>

2. 運行指令配置

    啟動時間入配置訊息（不建議）

    ```
    redis-server --configKey1 configValue1 --configKey2 configValue2
    ```

    ex: 改啟動 port

    ```
    redis-server --port 6380
    ```

<br>
<br>

3. 配置文件啟動

    啟動時，指定配置文件位址，假設我們把文件放在 /opt/redis/redis.conf

    啟動時下指令：

    ```
    redis-server /opt/redis/redis/conf
    ```

    <br>

    預設的設定檔目錄如下：

    `/etc/redis/redis.conf`

    如果不指定特別設定，則會自動使用這個官方預設的設定檔

    <br>

    我們需要進行客製化設定時，需要 copy 一份官方設定來進行修改

<br>
<br>

啟動！

<br>

```
[johnny@krb5]: redis -> sudo redis-server --port 6380
10712:C 20 Feb 23:08:42.640 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
10712:C 20 Feb 23:08:42.640 # Redis version=4.0.9, bits=64, commit=00000000, modified=0, pid=10712, just started
10712:C 20 Feb 23:08:42.640 # Configuration loaded
10712:M 20 Feb 23:08:42.642 * Increased maximum number of open files to 10032 (it was originally set to 1024).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 4.0.9 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6380
 |    `-._   `._    /     _.-'    |     PID: 10712
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

10712:M 20 Feb 23:08:42.643 # Server initialized
10712:M 20 Feb 23:08:42.644 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
10712:M 20 Feb 23:08:42.644 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
10712:M 20 Feb 23:08:42.644 * Ready to accept connections
```


<br>
<br>
<br>
<br>

## 操作 Redis

<br>

cli 客戶端連線 redis

```
redis-cli -h 127.0.0.1 -p 6380
```

<br>

set 值 get 值

<br>

```
[johnny@krb5]: redis -> redis-cli -h 127.0.0.1 -p 6380
127.0.0.1:6380> set hello world
OK
127.0.0.1:6380> get hello
"world"
127.0.0.1:6380> 
```

<br>
<br>

直接執行一次指令

<br>

```
[johnny@krb5]: redis -> redis-cli -h 127.0.0.1 -p 6380 get hello
"world"
```


<br>
<br>
<br>
<br>

## 停止 Redis

<br>

```
[johnny@krb5]: redis -> redis-cli -h 127.0.0.1 -p 6380 shutdown
[johnny@krb5]: redis -> redis-cli -h 127.0.0.1 -p 6380 get hello
Could not connect to Redis at 127.0.0.1:6380: Connection refused
[johnny@krb5]: redis -> 
```

<br>

關閉時是否要生成持久化文件也可以選：

```
redis-cli -h 127.0.0.1 -p 6380 shutdown nosave|save
```