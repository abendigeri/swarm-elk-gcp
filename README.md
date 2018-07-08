

# How to setup ELK Stack on Google Cloud Platform

## Environment

```
35.231.92.68 => 10.142.0.9
35.237.165.181 => 10.142.0.2
35.237.187.239 => 10.142.0.4
```

## Remove old Docker 1.13 version from your GCP instances


```
[root@docker-3 docker]# yum remove docker docker-common docker-selinux docker-engine
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror
No Match for argument: docker-engine
Resolving Dependencies
--> Running transaction check
---> Package container-selinux.noarch 2:2.55-1.el7 will be erased
---> Package docker.x86_64 2:1.13.1-63.git94f4240.el7.centos will be erased
---> Package docker-common.x86_64 2:1.13.1-63.git94f4240.el7.centos will be erased
--> Processing Dependency: docker-common for package: 2:docker-client-1.13.1-63.git94f4240.el7.centos.x86_64
--> Running transaction check
---> Package docker-client.x86_64 2:1.13.1-63.git94f4240.el7.centos will be erased
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package            Arch    Version                              Repository
                                                                           Size
================================================================================
Removing:
 container-selinux  noarch  2:2.55-1.el7                         @extras   36 k
 docker             x86_64  2:1.13.1-63.git94f4240.el7.centos    @extras   57 M
 docker-common      x86_64  2:1.13.1-63.git94f4240.el7.centos    @extras  4.5 k
Removing for dependencies:
 docker-client      x86_64  2:1.13.1-63.git94f4240.el7.centos    @extras   13 M

Transaction Summary
================================================================================
Remove  3 Packages (+1 Dependent package)

Installed size: 70 M
Is this ok [y/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Erasing    : 2:docker-1.13.1-63.git94f4240.el7.centos.x86_64              1/4
warning: /etc/sysconfig/docker-storage saved as /etc/sysconfig/docker-storage.rpmsave
  Erasing    : 2:docker-client-1.13.1-63.git94f4240.el7.centos.x86_64       2/4
  Erasing    : 2:docker-common-1.13.1-63.git94f4240.el7.centos.x86_64       3/4
  Erasing    : 2:container-selinux-2.55-1.el7.noarch                        4/4
  Verifying  : 2:docker-common-1.13.1-63.git94f4240.el7.centos.x86_64       1/4
  Verifying  : 2:docker-1.13.1-63.git94f4240.el7.centos.x86_64              2/4
  Verifying  : 2:docker-client-1.13.1-63.git94f4240.el7.centos.x86_64       3/4
  Verifying  : 2:container-selinux-2.55-1.el7.noarch                        4/4

Removed:
  container-selinux.noarch 2:2.55-1.el7
  docker.x86_64 2:1.13.1-63.git94f4240.el7.centos
  docker-common.x86_64 2:1.13.1-63.git94f4240.el7.centos

Dependency Removed:
  docker-client.x86_64 2:1.13.1-63.git94f4240.el7.centos

Complete!

```

## Installing the latest Docker version on all the nodes

```
curl -sSL https://test.docker.com/ | sh
```

## Starting Docker Service on all the nodes

```
systemctl restart docker
```

## Verify the Docker version

```
[root@docker-2 swarm-elk]# docker version
Client:
 Version:      18.06.0-ce-rc2
 API version:  1.38
 Go version:   go1.10.3
 Git commit:   a4637b4
 Built:        Sat Jul  7 00:01:52 2018
 OS/Arch:      linux/amd64
 Experimental: false

Server:
 Engine:
  Version:      18.06.0-ce-rc2
  API version:  1.38 (minimum version 1.12)
  Go version:   go1.10.3
  Git commit:   a4637b4
  Built:        Sat Jul  7 00:04:14 2018
  OS/Arch:      linux/amd64
  Experimental: false
[root@docker-2 swarm-elk]
```

## Setting up Swarm Manager Node (10.142.0.2)

```
docker swarm init --advertise-addr 10.142.0.2 --listen-addr 10.142.0.2:2377
Swarm initialized: current node (f2ldl9dxry8rz3fdv5mjgd0qr) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join \
    --token SWMTKN-1-2icnkcdi2jb30z0zuayf8bfgtjm8ias0e94vqnydvr79pjg3qx-6okgpi9znojie4mzuap9wf10m \
    10.142.0.2:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

## Addng Worker Node

Run the below command on both the worker node

```
docker swarm join \
    --token SWMTKN-1-2icnkcdi2jb30z0zuayf8bfgtjm8ias0e94vqnydvr79pjg3qx-6okgpi9znojie4mzuap9wf10m \
    10.142.0.2:2377
```

## Listing out Docker Swarm Node Cluster Nodes

```
[root@docker-2 swarm-elk]# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
7vexu33mevck5pj98c4sh52nw     docker-1            Ready               Active                                  18.06.0-ce-rc2
jeh0qsb04iby7ldp6i8dr4qr3 *   docker-2            Ready               Active              Leader              18.06.0-ce-rc2
jw6nqxap0cyycjhmgmqg8bukx     docker-3            Ready               Active                                  18.06.0-ce-rc2
[root@docker-2 swarm-elk]#
```

## 

```
[root@docker-2 swarm-elk]# docker stack deploy -c docker-compose.yml myelk
Ignoring unsupported options: ulimits

Creating network myelk_elk
Creating service myelk_elasticsearch
Creating service myelk_kibana
Creating service myelk_logstash
[root@docker-2 swarm-elk]# docker stack ls
NAME                SERVICES            ORCHESTRATOR
myelk               3                   Swarm

```


## Listing out ELK services

```
ID                  NAME                  MODE                REPLICAS            IMAGE               PORTS
6fknkzsdbrke        myelk_elasticsearch   replicated          1/1                 elasticsearch:5
vzijhhjih1sk        myelk_kibana          replicated          1/1                 kibana:latest       *:5601->5601/tcp
joak42efqt1f        myelk_logstash        replicated          2/2                 logstash:alpine     *:10514->10514/tcp, *:10514->10514/udp, *:12201->12201/udp
[root@docker-2 swarm-elk]# curl 10.142.0.2:9200

```

## Scaling Elastisearch to 3 instances

```
[root@docker-2 swarm-elk]# docker service ls
ID                  NAME                  MODE                REPLICAS            IMAGE               PORTS
6fknkzsdbrke        myelk_elasticsearch   replicated          1/1                 elasticsearch:5
vzijhhjih1sk        myelk_kibana          replicated          1/1                 kibana:latest       *:5601->5601/tcp
joak42efqt1f        myelk_logstash        replicated          2/2                 logstash:alpine     *:10514->10514/tcp, *:10514->10514/udp, *:12201->12201/udp

[root@docker-2 swarm-elk]# docker service update --replicas=3 myelk_elasticsearch
myelk_elasticsearch
overall progress: 3 out of 3 tasks
1/3: running   [==================================================>]
2/3: running   [==================================================>]
3/3: running   [==================================================>]
verify: Service converged
[root@docker-2 swarm-elk]# docker service ls
ID                  NAME                  MODE                REPLICAS            IMAGE               PORTS
6fknkzsdbrke        myelk_elasticsearch   replicated          3/3                 elasticsearch:5
vzijhhjih1sk        myelk_kibana          replicated          1/1                 kibana:latest       *:5601->5601/tcp
joak42efqt1f        myelk_logstash        replicated          2/2                 logstash:alpine     *:10514->10514/tcp, *:10514->10514/udp, *:12201->12201/udp
[root@docker-2 swarm-elk]#
```

## Testing The Logstash

Run the following container:

```
docker run --rm -it --log-driver=gelf --log-opt gelf-address=udp://<logstash-host>:12201 alpine ping 8.8.8.8
```

Login to Kibana and you should see traffic coming into Elasticsearch.


![alt text](https://github.com/dockerworxinc/swarm-elk-gcp/blob/master/kibanaui.png)


## 


```
[root@docker-2 swarm-elk]# docker run -d --name nginx-with-syslog --log-driver=syslog --log-opt syslog-address=udp://10.142.0.2:12201 -p 80:80 nginx:alpine

Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
911c6d0c7995: Already exists
f0ee8b734032: Pull complete
f8840ac64af5: Pull complete
9f8c19504ab0: Pull complete
Digest: sha256:56a9367b64eaef37894842a6f7a19a0ef8e7bd5de964aa844a70b3e2d758033c
Status: Downloaded newer image for nginx:alpine
d6b5712045261399249eb2e35d5229cb87b608c519b48517fd034b2d75f15289
```

```
[root@docker-2 swarm-elk]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
d6b571204526        nginx:alpine        "nginx -g 'daemon of…"   3 seconds ago       Up 2 seconds        0.0.0.0:80->80/tcp   nginx-with-syslog
bdf20caf883d        elasticsearch:5     "/docker-entrypoint.…"   19 minutes ago      Up 19 minutes       9200/tcp, 9300/tcp   myelk_elasticsearch.1.cqa5992tty4xkgq10w6c9z7hq
742f8e93d89c        logstash:alpine     "/docker-entrypoint.…"   19 minutes ago      Up 19 minutes                            myelk_logstash.1.2g55of9anr9xum33g6wovcxud
[root@docker-2 swarm-elk]#
```


## Pushing data into Logstash:


Let us push NGINX web server logs into logstash and see if Kibana is able to detect it:


```
[root@docker-2 swarm-elk]# docker run -d --name nginx-with-syslog --log-driver=syslog --log-opt syslog-address=udp://10.142.0.2:12201 -p 80:80 nginx:alpine
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
911c6d0c7995: Already exists
f0ee8b734032: Pull complete
f8840ac64af5: Pull complete
9f8c19504ab0: Pull complete
Digest: sha256:56a9367b64eaef37894842a6f7a19a0ef8e7bd5de964aa844a70b3e2d758033c
Status: Downloaded newer image for nginx:alpine
d6b5712045261399249eb2e35d5229cb87b608c519b48517fd034b2d75f15289
[root@docker-2 swarm-elk]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
d6b571204526        nginx:alpine        "nginx -g 'daemon of…"   3 seconds ago       Up 2 seconds        0.0.0.0:80->80/tcp   nginx-with-syslog
bdf20caf883d        elasticsearch:5     "/docker-entrypoint.…"   19 minutes ago      Up 19 minutes       9200/tcp, 9300/tcp   myelk_elasticsearch.1.cqa5992tty4xkgq10w6c9z7hq
742f8e93d89c        logstash:alpine     "/docker-entrypoint.…"   19 minutes ago      Up 19 minutes                            myelk_logstash.1.2g55of9anr9xum33g6wovcxud
[root@docker-2 swarm-elk]#
```

![alt text](https://github.com/dockerworxinc/swarm-elk-gcp/blob/master/kibanaui1.png)


