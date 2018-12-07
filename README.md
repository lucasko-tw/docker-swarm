
## Join Node

There are two nodes in this example

 * **Manager**: 192.168.56.101
 * **Worker**: 192.168.56.101


Initialize docker swarm at manager node.

```
[root@manager ~]# sudo systemctl stop firewalld

[root@manager ~]# docker swarm init


[root@manager ~]# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
izlurchkcl8jthbqsmrqkomwe *   manager             Ready               Active              Leader              18.09.0
```

Generate token for worker at manager node

```
[root@manager ~]# docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-52qsglury6q0ixsisno7urk1xwel1t7ltfk4m5bqb7raaba9ew-e73kjr78pqmp8ri05bvnf3g2e 192.168.56.101:2377
```

Copy above command `docker swarm join --token  XXX`, run it at worker node.

```sh
[root@worker ~]# docker swarm join --token SWMTKN-1-52qsglury6q0ixsisno7urk1xwel1t7ltfk4m5bqb7raaba9ew-e73kjr78pqmp8ri05bvnf3g2e 192.168.56.101:2377

This node joined a swarm as a worker.
```
 
Worker node has been joined.

```sh
[root@manager ~]# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
izlurchkcl8jthbqsmrqkomwe *   manager             Ready               Active              Leader              18.09.0
ti3gskbad53l8qchei918oolc     worker              Ready               Active                                  18.09.0

```

### Docker Stack

Create a yml file.
 
```sh
[root@manager Desktop]# vim tomcat.yml
``` 
content of `tomcat.yml` is following :

```sh
version: "3" # 'deploy' only be offerd in version 3.
services: 
  stack_tomcat: # self-definition
    image: tomcat:8  
    ports:  
      - "8080:8080"
    deploy:
      replicas: 1  
      restart_policy:
        condition: on-failure
      placement:
        constraints: [node.role == worker] # which node will you deploy
```

Run Stack

```sh
[root@manager Desktop]# docker stack deploy -c tomcat.yml mytomcat
```
> If you want to remove, you could run `docker stack rm mytomcat`
> 
> `mytomcat` is self-definition 
 
 
Check Service

```sh
[root@manager Desktop]# docker service ls
ID                  NAME                    MODE                REPLICAS            IMAGE               PORTS
m5re0cl9zi29        mytomcat_stack_tomcat   replicated          1/1                 tomcat:8            *:8080->8080/tcp


[root@manager Desktop]# docker service ps mytomcat_stack_tomcat
ID                  NAME                      IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
tvzao1uqesw0        mytomcat_stack_tomcat.1   tomcat:8            worker              Running             Running 8 minutes ago


[root@manager Desktop]# docker service ls
ID                  NAME                    MODE                REPLICAS            IMAGE               PORTS
m5re0cl9zi29        mytomcat_stack_tomcat   replicated          1/1                 tomcat:8            *:8080->8080/tcp

```


At worker node

```sh
[root@worker Desktop]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
06ef0b380d80        tomcat:8            "catalina.sh run"   14 minutes ago      Up 14 minutes       8080/tcp            mytomcat_stack_tomcat.1.tvzao1uqesw0r1fcaz1dikzc5
```


### Remove Stack

```
[root@manager Desktop]# docker service ls
ID                  NAME                    MODE                REPLICAS            IMAGE               PORTS
m5re0cl9zi29        mytomcat_stack_tomcat   replicated          1/1                 tomcat:8            *:8080->8080/tcp


[root@manager Desktop]# docker stack rm mytomcat
Removing service mytomcat_stack_tomcat
Removing network mytomcat_default


[root@manager Desktop]# docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
```
 
 
### Remove node
 
 From manager side
 
 ```sh
 docker node rm WORKER_NAME
 ``` 
 From worker side
 
 ```sh
 docker swarm leave
 ```
