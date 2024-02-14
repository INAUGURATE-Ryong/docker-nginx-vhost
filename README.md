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
$ docker run -itd -p 8003:80 --name serv-a nginx
$ docker run -itd -p 8001:80 --name lb nginx:latest
```
