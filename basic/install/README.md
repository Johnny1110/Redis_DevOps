# 安裝 Redis

<br>

---

<br>


Redis 在 Unix Base 的 OS 上操作比較簡單，如果要在 Windows 上安裝需要用 docker，這裡示範使用 Ubuntu 下載 Redis


<br>

下載

<br>

```cli
apt-get install redis
```

<br>

查看版本資訊

<br>

```cli
redis-cli -v
```

<br>
我安裝的當下 redis 版本為 redis-cli 4.0.9

<br>

透過 apt-get 自動安裝的話我們無法指定安裝目錄，所以 redis (包括其他軟體都一樣) 的幾個重要目錄預設如下：

<br>

可執行文件 : `/usr/bin /usr/sbin`

配置文件 : ` /etc`

lib文件 : `/usr/lib`

<br>
<br>
<br>
<br>

__Redis 版號偶數為穩定版，基數為測試版。__