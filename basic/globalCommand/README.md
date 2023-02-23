# 6 個基本全域指令

<br>

---

<br>

## 查找所有 key

<br>

```redis
keys *
```

<br>


```
127.0.0.1:6380> keys *
1) "key"
2) "1"
3) "hello"
4) "GNR"
127.0.0.1:6380> 
```

<br>
<br>
<br>
<br>

## 查目前 key 總數

<br>

```
dbsize
```

<br>

```
127.0.0.1:6380> dbsize
(integer) 4
127.0.0.1:6380> 
```

<br>
<br>
<br>
<br>

## 查 key 是否存在

<br>

```
exists key
```

<br>

0 是不存在，1 是存在

<br>

```
127.0.0.1:6380> exists ACDC
(integer) 0
127.0.0.1:6380> exists GNR
(integer) 1
127.0.0.1:6380> 
```

<br>
<br>
<br>
<br>

## 刪除 key (可以一次多筆)

<br>

```
del key1 key2 key3
```

<br>

```
127.0.0.1:6380> exists GNR
(integer) 1
127.0.0.1:6380> del GNR
(integer) 1
127.0.0.1:6380> exists GNR
(integer) 0
127.0.0.1:6380> 
```

<br>
<br>
<br>
<br>

## key 過期時間

<br>

設定 key 過期秒數，時間到自動刪除

<br>

```
expire key seconds
```

<br>

查看 key 剩餘有效期秒數

<br>

```
ttl key seconds
```

<br>

```
127.0.0.1:6380> set hello world
OK
127.0.0.1:6380> expire hello 20
(integer) 1
127.0.0.1:6380> ttl hello
(integer) 17 # 剩餘 17 秒
127.0.0.1:6380> 
```

<br>
<br>
<br>
<br>

## key 的值類型

<br>

回傳 key 存的值的類型

<br>

```
type key
```

<br>

```
127.0.0.1:6380> set A 'this is a string'
OK
127.0.0.1:6380> rpush B a b c d e f g h i j k l m n
(integer) 14
127.0.0.1:6380> type A
string
127.0.0.1:6380> type B
list
127.0.0.1:6380> 
```
