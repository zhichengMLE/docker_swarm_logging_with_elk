#Docker Swarm Monitoring
===

Docker Stack compose file to deploy dynamic logging for Docker Swarm. This stack deploys the following services:

* `Elasticsearch`: Database to store the log data and query for it.
* `Logstash`: Log processor that ingests data from multiple sources and feeds it to Elasticsearch.
* `Logspout`: Logspout is deployed to all hosts to collect the logs from docker daemon and feed it to logstash.
* `Kibana`: Web UI to display Elasticsearch data.

## Run Instructions
---
### Setup a Docker Swarm with the Docker Swarm Mode.

#### Create docker machine
```
docker-machine create manager
docker-machine create agent1
docker-machine create agent2
```

#### Create docker swarm and join the swarm
```
docker swarm init --advertise-addr `docker-machine ip manager`
docker-machine ssh agent1 docker swarm join --token `docker swarm join-token -q worker` `docker-machine ip manager`:2377
docker-machine ssh agent2 docker swarm join --token `docker swarm join-token -q worker` `docker-machine ip manager`:2377
```

#### Set maximal memory (otherwise, elasticsearch doesn't work)
```
docker-machine ssh manager sudo sysctl -w vm.max_map_count=262144
docker-machine ssh agent1 sudo sysctl -w vm.max_map_count=262144
docker-machine ssh agent2 sudo sysctl -w vm.max_map_count=262144
```


### From the swarm manager, deploy the stack with the command,
```
docker stack deploy -c docker-stack.yml elk
```

### Open Kibana in browser
```
open http://`docker-machine ip manager`
```

In Kibana, create a new index `logstash-*` with `@timestamp` as the Time-field name.

### Run Nginx services

```
docker stack deploy -c nginx.yml nginx
```

### Open Nginx in browser

```
open http://`docker-machine ip manager:8000`
```



### Elastic Search

```
curl http://127.0.0.1:9200/_cat/health
```



### Remove all docker swarm services

```
docker service rm $(docker service ls -q)
```

### Remove all docker machines

```
docker-machine rm -f $(docker-machine ls -q)
```



### Reference
- https://medium.com/@yoanis_gil/running-an-elasticsearch-cluster-on-docker-1-9-with-swarm-and-compose-efabe110c675
- https://botleg.com/stories/log-management-of-docker-swarm-with-elk-stack/