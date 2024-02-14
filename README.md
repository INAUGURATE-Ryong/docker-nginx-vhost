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

- bridge 네트워크는 하나의 호스트 컴퓨터 내에서 여러 컨테이너 연결
- host 네트워크는 컨터이너를 호스트 컴퓨터와 동일한 네트워크에서 컨테이너를 돌리기 위해서 사용
- overlay 네트워크는 여러 호스트에 분산되어 돌아가는 컨테이너들 간에 네트워킹을 위해서 사용
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

# step8
```
각 serv-a,b 컨테이너에 들어가서 ping, telnet 설치
$ sudo docker exec -it serv-b bash
$ sudo docker exec -it serv-a bash

root@28bdc543cf50:/# apt update
root@28bdc543cf50:/# apt install iputils-ping
root@28bdc543cf50:/# apt install telnet

serv-a에 다시 접속해서
root@28bdc543cf50:/# ping 172.18.0.2(serv-b IP)  또는 ping serv-b

PING serv-b (172.18.0.3) 56(84) bytes of data.
64 bytes from serv-b.ablb (172.18.0.3): icmp_seq=1 ttl=64 time=0.767 ms
64 bytes from serv-b.ablb (172.18.0.3): icmp_seq=2 ttl=64 time=0.095 ms
64 bytes from serv-b.ablb (172.18.0.3): icmp_seq=3 ttl=64 time=0.070 ms
64 bytes from serv-b.ablb (172.18.0.3): icmp_seq=4 ttl=64 time=0.106 ms

root@28bdc543cf50:/# telnet serv-b 80  // 8002,8003포트가 아닌 docker 내부 포트 이용 network설정을 하면 8002,8003포트가 필요가 없어진다.
Trying 172.18.0.3...
Connected to serv-b.
Escape character is '^]'.
```

# step9
```
serv-a 와 serv-b의 포트번호인 8002,8003 을 없애고, 8001 포트인 lb만을 통해서 접속하는 실습
$ sudo docker commit serv-a memento12/serv-a
$ sudo docker commit serv-b memento12/serv-b

$ docker rm serv-a
$ docker rm serv-b

$ docker run --name serv-a -d memento12/serv-a  // -p옵션 없이 run 하기
$ docker run --name serv-b -d memento12/serv-b  // -p옵션 없이 run 하기

$ docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED         STATUS          PORTS                                   NAMES
3cbb0d8cb116   memento12/serv-a   "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes    80/tcp                                  serv-a
3522b218f63e   memento12/serv-b   "/docker-entrypoint.…"   7 minutes ago   Up 7 minutes    80/tcp                                  serv-b
2127fcb00f0d   nginx:latest       "/docker-entrypoint.…"   4 hours ago     Up 55 minutes   0.0.0.0:8001->80/tcp, :::8001->80/tcp   lb

$ docker network connect ablb serv-a
$ docker network connect ablb serv-b

$ sudo docker network inspect ablb // 컨테이너 확인 
```

# step10(Dockerfile 생성 후 build, run)
```
도커 파일 생성 
$ vi lb/Dockerfile
From nginx
COPY config/default.conf /etc/nginx/conf.d/

$ vi serv-a/Dockerfile 
FROM nginx
COPY index.html /usr/share/nginx/html

$ vi serv-b/Dockerfile 
FROM nginx
COPY index.html /usr/share/nginx/html

build
$ sudo docker build -t serv-b:\n0.1.0 .
$ sudo docker build -t serv-b:0.1.0 .
$ sudo docker build -t lb:0.1.0 .

run
$ sudo docker run -d --name serv-a serv-a:0.1.0
$ sudo docker run -d --name serv-b serv-b:0.1.0
$ sudo docker run -d --name lb -p 8001:80 lb:0.1.0

network
$ docker network create dockerfileNW
$ sudo docker network connect dockerfileNW serv-a
$ sudo docker network connect dockerfileNW serv-b
$ sudo docker network connect dockerfileNW lb
```
