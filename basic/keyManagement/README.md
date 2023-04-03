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

### 2.  key 過期

<br>

除了 `expire` 與 `ttl` 命令以外，Redis 還提供了 `expireat`、 `pexpire`、`pexpireat`、`pttl`、`persist` 等命令：

<br>

expire [key] [seconds]：key 在 seconds 秒後過期。

expireat [key] [timestamp]：key 在時間戳 timestamp 後過期。

* 將 hello 這個 key 設定某一特定時間過期，例如 2023年4月9日 上午6:31:51（timestamp = 1680993111）

    ```
    expireat hello 1680993111
    ```

<br>

ttl [key]：查詢 key 剩餘過期秒數。

pttl [key]：同 ttl 一樣，但是返回的值精度為毫秒。

* 回傳值 >= 0 代表 key 剩餘保質期 (`ttl` 是 秒，`pttl` 是毫秒)

* 回傳值 = -1 代表 key 沒有設定過期時間

* 回傳值 = -2 代表 key 不存在

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