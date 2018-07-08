

# How to setup ELK Stack on Google Cloud Platform

## Environment

```
35.231.92.68 => 10.142.0.9
35.237.165.181 => 10.142.0.2
35.237.187.239 => 10.142.0.4
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
docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
3jed9xgpj0ypqxlue3itpa8yt    docker-3  Ready   Active
f2ldl9dxry8rz3fdv5mjgd0qr *  docker-2  Ready   Active        Leader
jia581rkhqckbibfu33z5jx84    docker-1  Ready   Active
[root@docker-2 docker]#
```

## 

