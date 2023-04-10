# key 管理

<br>

----

<br>

這一篇，針對 __單一 key__，__走訪(遍歷)__，__資料庫管理__ 進行介紹。

<br>
<br>
<br>
<br>

## 單一 key

<br>

### 1. rename key

```redis
rename key newKeyName
```

<br>

__如果在 rename 前，key 已經存在，那麼已存在的 key 的值會被覆蓋 :__

```
127.0.0.1:6380> set a b
OK
127.0.0.1:6380> set c d
OK
127.0.0.1:6380> rename a c
OK
127.0.0.1:6380> get a
(nil)
127.0.0.1:6380>  get c
"b"
```

<br>

為了防止上述問題發生，redis 建議使用 `renamenx` 指令，確保只有 newKeyName 不存在時，才可以改成功：

```
127.0.0.1:6380> set java jedis
OK
127.0.0.1:6380> set python redis-py
OK
127.0.0.1:6380> renamenx java python
(integer) 0 # 未修改成功
127.0.0.1:6380> get java
"jedis"
127.0.0.1:6380> get python
"redis-py"
```

<br>

__注意！：重命名期間，實際上會執行 `del` 命令刪除舊的 key，如果 key 對應的值很大，會存在阻塞 Redis 的可能。__


<br>
<br>

### 2. 隨機返回 key

<br>

返回當前資料庫中的任意一個 key

```
127.0.0.1:6380> randomkey
"B"
127.0.0.1:6380> randomkey
"python"
127.0.0.1:6380> randomkey
"c"
127.0.0.1:6380> randomkey
"hello"
```

<br>
<br>

### 3.  key 過期

<br>

除了 `expire` 與 `ttl` 命令以外，Redis 還提供了 `expireat`、 `pexpire`、`pexpireat`、`pttl`、`persist` 等命令：

<br>

`expire` [key] [seconds]：key 在 seconds 秒後過期。

`expireat` [key] [timestamp]：key 在時間戳 timestamp 後過期。

* 將 hello 這個 key 設定某一特定時間過期，例如 2023年4月9日 上午6:31:51（timestamp = 1680993111）

    ```
    expireat hello 1680993111
    ```

* __如果過期時間設定為負數，該鍵立即刪除。__

<br>

`ttl` [key]：查詢 key 剩餘過期秒數。

`pttl `[key]：同 ttl 一樣，但是返回的值精度為毫秒。

* 回傳值 >= 0 代表 key 剩餘保質期 (`ttl` 是 秒，`pttl` 是毫秒)

* 回傳值 = -1 代表 key 沒有設定過期時間

* 回傳值 = -2 代表 key 不存在

<br>

`persist ` [key]  ：可以將 key 過期時間清除

```
127.0.0.1:6380> set key aabbcc
OK
127.0.0.1:6380> expire key 100
(integer) 1
127.0.0.1:6380> ttl key
(integer) 95
127.0.0.1:6380> persist key
(integer) 1
127.0.0.1:6380> ttl key
(integer) -1
```

<br>
<br>
<br>

__對於字串類型的 key，執行 `set` 命令會移除過期時間，這一點要注意：__

```
127.0.0.1:6380> expire hello 100
(integer) 1
127.0.0.1:6380> ttl hello
(integer) 95
127.0.0.1:6380> set hello hell
OK
127.0.0.1:6380> ttl hello
(integer) -1
```

<br>

__Redis 不支援 Hash、Set 內部元素的過期功能（要過期就是整個 key 過期）__


<br>

__`setex` 命令是 `set` + `expire` 的組合，不但是原子執行，同時可以減少一次網路連線__

<br>
<br>
<br>
<br>

### 4.  遷移 key 

<br>

遷移 key 可以把部份資料從一個 Redis 移轉到另一個 Redis（例如把 stage 資料移轉到 prod環境）。

遷移 key 有 3 種作法：

1. `move`
2. `dump` + `restore` 
3. `migrate`

<br>
<br>

### `move` 方法

<br>

```
move key db
```

<br>
<br>

`move` 命令適合用在 Redis 內部進行資料遷移，Redis 內部可以有多個 db，Redis 內的不同 db 相互隔離的，`move key db` 可以把指令 key 從該資料庫移轉到目標資料庫，（不建議使用，不太會在一個 redis 內建多個 db）。


<br>

### `dump` + `restore`

<br>

```
dump key
restore key ttl value
```

<br>

`dump` + `restore` 可以實現在不同 Redis 實例之間進行資料遷移，整個遷移過程分為 2 步驟：

1. 在來源 Redis 上，`dump` 命令將 key 序列化，格式使用 RDB 格式。

2. 在目標 Redis 上，`restore` 命令將上面序列化的值復原，__`ttl` 如果沒有要設定就給 0__。

<br>

為了做示範，我這邊使用 docker 啟動 2 個 redis container。

```
[johnny@krb5]: Redis_DevOps -> sudo docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                    NAMES
02ebfd3e1bf1   redis     "docker-entrypoint.s…"   16 seconds ago   Up 15 seconds   0.0.0.0:6381->6379/tcp   redis-test2
b3a08239ad8e   redis     "docker-entrypoint.s…"   6 weeks ago      Up 30 minutes   0.0.0.0:6380->6379/tcp   redis-test
```

<br>

開啟兩個 redis-cli 客戶端分別連接兩個 redis：

cli1:
```
[johnny@krb5]: Redis_DevOps -> redis-cli -p 6380
127.0.0.1:6380> set testDumpKey abc
OK
127.0.0.1:6380> dump testDumpKey
"\x00\x03abc\n\x00\xd9\xac0=Ar\xa2\xe2"
127.0.0.1:6380> 
```

<br>

cli2:
```
[johnny@krb5]: Redis_DevOps -> redis-cli -p 6381
127.0.0.1:6381> get testDumpKey
(nil)
127.0.0.1:6381> restore testDumpKey 0 "\x00\x03abc\n\x00\xd9\xac0=Ar\xa2\xe2"
OK
127.0.0.1:6381> get testDumpKey
"abc"
127.0.0.1:6381> 
```

<br>
<br>

### `migrate`

<br>

`migrate host port key | "" destination-db timeout [copy] [replace] [keys key [key ...]]`

<br>
<br>
<br>
<br>



<br>
<br>
<br>
<br>

## 走訪(遍歷)

<br>

<br>
<br>
<br>
<br>

## 資料庫管理

<br>