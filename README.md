# docker nginx vhost

![image](https://github.com/pySatellite/docker-nginx-vhost/assets/87309910/878eaf6a-18bc-4467-8b3f-5086de8ff3a1)
![image](https://github.com/INAUGURATE-Ryong/docker-nginx-vhost/assets/62015109/b763ba5c-38e0-4b8c-89ff-d6057fe40c7b)


# load-balancing
https://www.nginx.com/resources/glossary/load-balancing/

# step 1
- docker rm * rmi
```
$ sudo docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
$ sudo docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

# step 2
```
$ docker run -itd -p 8002:80 --name serv-a nginx
$ docker run -itd -p 8003:80 --name serv-b nginx
$ docker run -itd -p 8001:80 --name lb nginx:latest
```

# step 3
```
vi config/default.conf

upstream serv {
    server serv-a:80;
    server serv-b:80;
}
server {
    listen 80;

    location /
    {
        proxy_pass http://serv;
    }
}
```
# step 4
```
$ sudo docker exec -it lb bash 에 들어가서.
root@2127fcb00f0d:/# cd etc/nginx/conf.d 폴더 안에 있는 default.conf 를 step3에 만든 default.conf로 바꾸기.
$ sudo docker cp config/default.conf lb:/etc/nginx/conf.d/

```

# step5
```
$ mkdir lb serv-a serv-b //폴더 3개 만들기
$ mv config lb // config폴더를 lb폴더로 옮기기
$ tree
.
├── README.md
├── lb
│   └── config
│       └── default.conf
├── serv-a
└── serv-b
```

# step6
```
$ vi serv-a/index.html  <h1>A</h1>
$ cp serv-a/index.html serv-b
$ vi serv-b/index.html  <h1>A</h1>
각각 파일 만들어서 컨테이너 /usr/share/nginx/html/ 밑에 각각 cp하기
$ sudo docker cp serv-a/index.html serv-a:/usr/share/nginx/html/
$ sudo docker cp serv-b/index.html serv-b:/usr/share/nginx/html/
```

# step7
$ docker network ls

bridge 네트워크는 하나의 호스트 컴퓨터 내에서 여러 컨테이너 연결
host 네트워크는 컨터이너를 호스트 컴퓨터와 동일한 네트워크에서 컨테이너를 돌리기 위해서 사용
overlay 네트워크는 여러 호스트에 분산되어 돌아가는 컨테이너들 간에 네트워킹을 위해서 사용
```
네트워크 생성 후 붙이기.
create
$ docker network create ablb
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
dfff2f701263   ablb      bridge    local
b49cfa625b8a   bridge    bridge    local
dd33952430ff   host      host      local
bd86b65b416b   none      null      local

inspect
$ docker network inspect ablb //내용은 이슈 확인

connect
$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED       STATUS         PORTS                                   NAMES
563b52d28965   nginx          "/docker-entrypoint.…"   3 hours ago   Up 3 hours     0.0.0.0:8003->80/tcp, :::8003->80/tcp   serv-b
28bdc543cf50   nginx          "/docker-entrypoint.…"   3 hours ago   Up 3 hours     0.0.0.0:8002->80/tcp, :::8002->80/tcp   serv-a

$ sudo docker network connect ablb serv-a
$ sudo docker network connect ablb serv-b
$ sudo docker network connect ablb lb

$ docker network inspect ablb 다시 했을 때 컨테이너 부분에 lb가 안떠있으면 //내용은 이슈 확인
$ docker start lb 실행 후 확인
$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED       STATUS         PORTS                                   NAMES
2127fcb00f0d   nginx:latest   "/docker-entrypoint.…"   3 hours ago   Up 2 minutes   0.0.0.0:8001->80/tcp, :::8001->80/tcp   lb
563b52d28965   nginx          "/docker-entrypoint.…"   3 hours ago   Up 3 hours     0.0.0.0:8003->80/tcp, :::8003->80/tcp   serv-b
28bdc543cf50   nginx          "/docker-entrypoint.…"   3 hours ago   Up 3 hours     0.0.0.0:8002->80/tcp, :::8002->80/tcp   serv-a
```
