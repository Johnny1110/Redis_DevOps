# Docker 啟動 Redis

<br>

---

<br>

在已經安裝了 Docker 的條件下，安裝啟動 Redis 也很簡單。

<br>

<br>

__先 pull redis 鏡像__

<br>

```
sudo docker pull redis
```

<br>

結果：

<br>

```
[johnny@krb5]: Redis_DevOps -> sudo docker pull redis
Using default tag: latest
latest: Pulling from library/redis
bb263680fed1: Pull complete 
ac509f65c3e9: Pull complete 
51afc2cce3df: Pull complete 
817f7e347ebd: Pull complete 
ab1a1215d5f9: Pull complete 
db7c27bf3552: Pull complete 
Digest: sha256:6a59f1cbb8d28ac484176d52c473494859a512ddba3ea62a547258cf16c9b3ae
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
[johnny@krb5]: Redis_DevOps -> 
```

<br>
<br>
<br>
<br>

__啟動鏡像__

<br>

這裡因為我本機的 6379 port 被佔了，我改開本機的 6380 port 去對應容器的 6379 port。

<br>

```
sudo docker run -itd --name redis-test -p 6380:6379 redis
```

<br>

結果

<br>

```
[johnny@krb5]: Redis_DevOps -> sudo docker run -itd --name redis-test -p 6380:6379 redis
b3a08239ad8e03c5b29d643528e52911fccd9875848b4e3a0b14be5fc0c114e0
```

<br>
<br>
<br>
<br>

__查看鏡像運行狀態__

<br>

```
sudo docker ps
```

<br>

結果

<br>

```
[johnny@krb5]: Redis_DevOps -> sudo docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                    NAMES
b3a08239ad8e   redis     "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:6380->6379/tcp   redis-test
```

<br>
<br>
<br>
<br>

__測試 docker redis__

<br>

要測這一段還是需要有 redis-cli 客戶端，不然就是下載 Medis （GUI）軟體來連。

<br>

```
[johnny@krb5]: Redis_DevOps -> redis-cli -p 6380
127.0.0.1:6380> set hello world
OK
127.0.0.1:6380> get hello
"world"
127.0.0.1:6380> 
```

<br>
<br>
<br>
<br>

__關閉 docker redis 服務__

<br>

```
sudo docker stop redis-test
```

<br>

<br>

<br>

<br>

__重啟 docker redis 服務__

<br>

```
sudo docker start redis-test
```

<br>






