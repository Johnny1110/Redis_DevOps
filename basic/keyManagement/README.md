# key 管理 (單一key，key 遷移，走訪 key，資料庫管理)

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

`migrate` 是將 `dump` `restore` `del` 三個命令組合。__且具備原子性，不需要開啟多個 redis-cli。__，此外 `migrate` 命令的資料傳輸直接在源 Redis 和目標 Redis 上完成，目標 Redis 完成之後會發送 ok 給源 Redis，源 Redis 接收後根據 `migrate` 對應的選項來決定是否在源 Redis 上刪除對應的 key。

<br>

`migrate host port key|"" destination-db timeout [copy] [replace] [keys key [key ...]]`

<br>

參數介紹 ( `[]` 項目為選填 )

`host` : 目標 Redis IP 位置。

`port` : 目標 Redis port。

`key|""` : 這個意思是要馬給一個 key，要馬給空字串。Redis 3.0.6 前只支援一次遷移一個 key，之後的版本可以一次遷移多個 key，若要一次遷移多個 key，`""` 內留空就可以，後面可以指定多個 key (看範例 2)。

`destination-db` : 目標資料庫，例如要遷移到 0 號資料庫，就填入 0。

`timeout` : 遷移超時時間(單位毫秒)。

`[copy]` : 如果添加此項，遷移後源 Redis 不會刪除 key。

`[replace]` : 如果添加此項，__目標 Redis 不管之前有沒有該 key，直接覆蓋__。

`[keys key [key ...]]` : 遷移多個 key，例如要遷移 k1, k2, k3 則填入 `keys k1 k2 k3`。

<br>

範例 1 : 

 把 hello 這個 key 遷移到 127.0.0.1:6381 的 0 號資料庫，超時時間 1 秒 

```
127.0.0.1:6380> migrate 127.0.0.1 6381 hello 0 1000
```

<br>

範例 2 : 

把 k1 k2 k3 這幾個 key 遷移到 127.0.0.1:6381 的 0 號資料庫，超時時間 5 秒 


```
127.0.0.1:6380> migrate 127.0.0.1 6381 "" 0 5000 keys k1 k2 k3
```


<br>
<br>
<br>
<br>

---

<br>
<br>
<br>
<br>

## 走訪(遍歷)

<br>

Redis 有 2 個走訪 key 指令，分別是 `keys` 與 `scan`。

<br>

### 1. 全量 key 走訪 (`keys`)

<br>

`keys`  指令示範 : 
```
127.0.0.1:6380> keys *
 1) "python"
 2) "c"
 3) "key"
 4) "B"
 5) "A"
 6) "counter"
 7) "testDumpKey"
 8) "1"
 9) "hello"
10) "java"
```

<br>

`keys` 後面跟的是 pattern，pattern 可以有如下幾種：

* `*` 代表匹配任何字元

* `?` 代表匹配一個字元

* `[ ]` 代表匹配部份字元，例如 `[1, 3]` 代表匹配 1, 3 ，`[1-10]` 代表匹配 1 ~ 10  任意數字。

* `\x` 用來做轉義，例如要匹配字元 *，就可以輸入 `\*`

<br>

示範：

```
127.0.0.1:6380> mset video_1 aaa video_2 bbb video_3 ccc
OK
127.0.0.1:6380> keys video*
1) "video_1"
2) "video_3"
3) "video_2"
```

<br>

__如果今天 Redis 是只有少量的 key，要走訪 key 時使用 `keys` 指令 是 OK 的，但是遇到 Redis 有大量 key 存在時，執行 `keys` 命令會造成組塞。為了避免阻塞，可以改用 `scan` 指令。__ 

<br>
<br>

### 2. 漸進式 key 走訪 (`scan`)

<br>

Redis 2.8 版本後，提供了 `scan` 指令，解決了大量 key 走訪時阻塞問題。

`scan` 採用漸進式 key 走訪，要真正用 `scan` 實現 `keys` 的效果需要多次執行。Redis  儲存 key pair 實際使用的是 HashTable，簡化模型如下圖：

<br>

![p1](./imgs/p1.jpg)

<br>

<br>
<br>
<br>
<br>

---

<br>
<br>
<br>
<br>

## 資料庫管理

<br>ˊˊ