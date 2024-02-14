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
