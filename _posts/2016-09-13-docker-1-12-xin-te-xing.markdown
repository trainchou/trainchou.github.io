---
layout: post
title: Docker 1.12 新特性
date: '2016-09-13 07:28:40'
---

### docker 1.12 版本新特性
- docker swarm：集群管理，子命令有init, join, leave, update

- docker service：服务创建，子命令有create, inspect, update, remove

- docker node：节点管理，子命令有accept, promote, demote, inspect, update, tasks, ls, rm

- docker stack/deploy：试验特性，用于多应用部署， 类似与 docker-compose 中的特性。

### 如何升级？

```
$ curl -fsSL https://get.docker.com/ | sh
$ systemctl restart docker
```

### 集群管理

docker swarm

- **init** Initialize a Swarm

- **join** Join a Swarm as a node and/or manager

- **leave** Leave a Swarm

- **update** Update the Swarm

#### docker swarm init

初始化一个 swarm 集群：

```
$ docker swarm init
Swarm initialized: current node (0vwpni05mew2j84i6gjet44iu) is now a manager.
```

如果你的机器有多块网卡，则需要指定 ip 地址：

```
$ docker swarm init --advertise-addr 192.168.96.187
```

命令执行过后会返回其他节点加入时需要的 token，这个 token 也可以使用 `docker swarm join-token` 命令查看。

参数：

```
      --advertise-addr string           Advertised address (format: <ip|interface>[:port])
      --cert-expiry duration            Validity period for node certificates (default 2160h0m0s)
      --dispatcher-heartbeat duration   Dispatcher heartbeat period (default 5s)
      --external-ca value               Specifications of one or more certificate signing endpoints
      --force-new-cluster               Force create a new cluster from current state.
      --help                            Print usage
      --listen-addr value               Listen address (format: <ip|interface>[:port]) (default 0.0.0.0:2377)
      --task-history-limit int          Task history retention limit (default 5)
```

#### docker swarm join

添加节点：

使用 init 时生成的 token 进行节点的添加， 添加 worker 和 manager 节点的 token 是不同的。
具体可以在初始节点上通过 `docker swarm join-token (worker|manager)` 命令查询。
具体命令形如：

```
docker swarm join \
    --token SWMTKN-1-1pk90rlt0iaoqfr7kvq8qyi38n8it7tthzfv8e19l545e6uzql-1l1mhad32fcchvuhz4kjap2zd \
    192.168.96.187:2377
```

添加过以后可以通过 `docker node ls` 查看当前集群节点状态。
![docker node ls](https://storage.googleapis.com/b.swipeusercontent.com/5bvVh-uzyfc2C4a_jwb-ihW8wXt6KZ-609.png)

#### docker swarm leave

删除节点：

在某个节点上使用 `docker swarm leave` 命令，可使得当前节点离开 swarm 集群。
离开后，通过 ‘ docker node ls` 命令查看节点状态时，该节点的状态变为 **down**。


#### docker swarm update

更新 swarm 集群参数，具体参数和 init 命令的参数一致。

参数：

```
      --cert-expiry duration            Validity period for node certificates (default 2160h0m0s)
      --dispatcher-heartbeat duration   Dispatcher heartbeat period (default 5s)
      --external-ca value               Specifications of one or more certificate signing endpoints
      --help                            Print usage
      --task-history-limit int          Task history retention limit (default 5)
```

### 服务管理

docker service

  - **create**      Create a new service
  - **inspect**     Display detailed information on one or more services
  - **ps**          List the tasks of a service
  - **ls**          List services
  - **rm**          Remove a service
  - **scale**       Scale one or multiple services
  - **update**      Update a service


#### docker service create

创建一个服务：

```
Options:
      --constraint value               Placement constraints (default [])
      --container-label value          Container labels (default [])
      --endpoint-mode string           Endpoint mode (vip or dnsrr)
  -e, --env value                      Set environment variables (default [])
      --help                           Print usage
  -l, --label value                    Service labels (default [])
      --limit-cpu value                Limit CPUs (default 0.000)
      --limit-memory value             Limit Memory (default 0 B)
      --log-driver string              Logging driver for service
      --log-opt value                  Logging driver options (default [])
      --mode string                    Service mode (replicated or global) (default "replicated")
      --mount value                    Attach a mount to the service
      --name string                    Service name
      --network value                  Network attachments (default [])
  -p, --publish value                  Publish a port as a node port (default [])
      --replicas value                 Number of tasks (default none)
      --reserve-cpu value              Reserve CPUs (default 0.000)
      --reserve-memory value           Reserve Memory (default 0 B)
      --restart-condition string       Restart when condition is met (none, on-failure, or any)
      --restart-delay value            Delay between restart attempts (default none)
      --restart-max-attempts value     Maximum number of restarts before giving up (default none)
      --restart-window value           Window used to evaluate the restart policy (default none)
      --stop-grace-period value        Time to wait before force killing a container (default none)
      --update-delay duration          Delay between updates
      --update-failure-action string   Action on update failure (pause|continue) (default "pause")
      --update-parallelism uint        Maximum number of tasks updated simultaneously (0 to update all at once) (default 1)
  -u, --user string                    Username or UID
      --with-registry-auth             Send registry authentication details to swarm agents
  -w, --workdir string                 Working directory inside the container
```


#### service 和 task 的区别

在使用 `docker service create` 命令的时候如果添加了 `--replicas` 参数，也就是副本数大于1时，每个实例就是一个 task 。

```
$docker service create --name nginx --replicas 2 -p 80:80/tcp nginx

$docker service ls

ID            NAME   REPLICAS  IMAGE  COMMAND
1b9a58mlz330  nginx  1/2       nginx  

$docker service tasks nginx
ID                         NAME     SERVICE  IMAGE  LAST STATE           DESIRED STATE  NODE
56er48j3hin9ysdi3sb1chbn1  nginx.1  nginx    nginx  Preparing 2 minutes  Running        swarm-node-1
e7vtvpkbstznoi8ogihaao1f5  nginx.2  nginx    nginx  Running 2 minutes    Running        swarm-manager
```

#### 创建一个带 rolling update 策略的服务

```
$ docker service create \
  --replicas 10 \
  --name redis \
  --update-delay 10s \
  --update-parallelism 2 \
  redis:3.0.6
```

当服务更新时，则按照每10秒钟更新两个实例的策略进行更新。

#### 设置环境变量

```
$ docker service create --name redis_2 --replicas 5 --env MYVAR=foo redis:3.0.6
```

#### 为服务设置标签

```
$ docker service create \
  --name redis_2 \
  --label com.example.foo="bar"
  --label bar=baz \
  redis:3.0.6
```

#### 设置模式

```
$ docker service create --name redis_2 --mode global redis:3.0.6
```

#### 设定调度规则

|node|attribute matches|example|
|----|----|----|
|node.id|node ID|node.id == 2ivku8v2gvtg4|
|node.hostname|node hostname|node.hostname != node-2|
|node.role|node role: manager|node.role == manager|
|node.labels|user defined node labels|node.labels.security == high|
|engine.labels|Docker Engine’s labels	|engine.labels.operatingsystem == ubuntu 14.04|

例如：
```
$ docker service create \
  --name redis_2 \
  --label com.example.foo="bar"
  --label bar=baz \
  redis:3.0.6
```

#### 具备 reschedule 的功能

task 挂掉以后可自动调配重新启动。


#### docker service scale

通过命令 `docker service scale [servicename]=n` 的方式来扩展服务；
也可通过该命令缩容，被缩减的 task （容器）实际是被 stop 了，而非 rm。

#### docker service rm

通过命令 `docker service rm [servicename]` 可删除一个 service 下的全部容器。

#### docker service update

更新 service 参数。

参数：

```
      --args string                    Service command args
      --constraint-add value           Add or update placement constraints (default [])
      --constraint-rm value            Remove a constraint (default [])
      --container-label-add value      Add or update container labels (default [])
      --container-label-rm value       Remove a container label by its key (default [])
      --endpoint-mode string           Endpoint mode (vip or dnsrr)
      --env-add value                  Add or update environment variables (default [])
      --env-rm value                   Remove an environment variable (default [])
      --help                           Print usage
      --image string                   Service image tag
      --label-add value                Add or update service labels (default [])
      --label-rm value                 Remove a label by its key (default [])
      --limit-cpu value                Limit CPUs (default 0.000)
      --limit-memory value             Limit Memory (default 0 B)
      --log-driver string              Logging driver for service
      --log-opt value                  Logging driver options (default [])
      --mount-add value                Add or update a mount on a service
      --mount-rm value                 Remove a mount by its target path (default [])
      --name string                    Service name
      --network-add value              Add or update network attachments (default [])
      --network-rm value               Remove a network by name (default [])
      --publish-add value              Add or update a published port (default [])
      --publish-rm value               Remove a published port by its target port (default [])
      --replicas value                 Number of tasks (default none)
      --reserve-cpu value              Reserve CPUs (default 0.000)
      --reserve-memory value           Reserve Memory (default 0 B)
      --restart-condition string       Restart when condition is met (none, on-failure, or any)
      --restart-delay value            Delay between restart attempts (default none)
      --restart-max-attempts value     Maximum number of restarts before giving up (default none)
      --restart-window value           Window used to evaluate the restart policy (default none)
      --stop-grace-period value        Time to wait before force killing a container (default none)
      --update-delay duration          Delay between updates
      --update-failure-action string   Action on update failure (pause|continue) (default "pause")
      --update-parallelism uint        Maximum number of tasks updated simultaneously (0 to update all at once) (default 1)
  -u, --user string                    Username or UID
      --with-registry-auth             Send registry authentication details to swarm agents
  -w, --workdir string                 Working directory inside the container

```

### 节点管理

docker node

  - **demote**      Demote a node from manager in the swarm
  - **inspect**     Display detailed information on one or more nodes
  - **ls**          List nodes in the swarm
  - **promote**     Promote a node to a manager in the swarm
  - **rm**          Remove a node from the swarm
  - **ps**          List tasks running on a node
  - **update**      Update a node

