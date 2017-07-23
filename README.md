# spring-cloud-config-eureka-docker

Repository that is consisted of 3 modules 
* Eureka 
* Config
* Client (uses remote configuration and is a Eureka client)

Idea is to demonstrate how to create a docker swarm and deploy all modules in highly available way. 

TODO: Add spring cloud buss and auto refresh all configurations.



#Docker swarm instructions


## Create Docker Machines
docker-machine create manager
docker-machine create worker1
docker-machine create worker2

docker-machine ssh manager
swarm init --advertise-addr=192.168.99.100

## Copy the swarm join command
docker swarm join --token SWMTKN-1-0p7hmpnatnqtexz9ewa4hcb4iost4u6sp578e177831d8ec6nl-34j14812r8wgdrzjps7rfdhdz 192.168.99.100:2377

docker-machine ssh worker1
paste
docker-machine ssh worker2
paste

eval $(docker-machine env manager)



# Create docker images
    cd client
    mvn package docker:build -Dmaven.test.skip=true
    cd ../eureka
    mvn package docker:build -Dmaven.test.skip=true
    cd ../config
    mvn package docker:build -Dmaven.test.skip=true

# Create network
    docker network create  --driver overlay network-us-west-2


# Create docker service
    docker service create --replicas 2 --name config  --reserve-memory=100Mb --publish 8888:8888 --update-delay 10s  --network network-us-west-2 springio/config
    docker service create --replicas 1 --name client  --reserve-memory=100Mb --publish 8080:8080 --update-delay 10s  --network network-us-west-2 springio/client


    docker service create --replicas 1 --name eureka  --reserve-memory=100Mb --publish 8761:8761 --update-delay 10s   --network network-us-west-2  -e spring_application_name=eureka --constraint  'node.role == manager' springio/eureka
    docker service create --replicas 1 --name eureka-backup  --reserve-memory=100Mb --publish 8762:8762 --update-delay 10s  --network network-us-west-2 -e spring_application_name=eureka-backup --constraint 'node.role != manager' springio/eureka

# Portener
    docker service create \
        --network network-us-west-2 \
        --name portainer \
        --publish 9000:9000 \
        --constraint 'node.role == manager' \
        --mount type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock \
        portainer/portainer \
        -H unix:///var/run/docker.sock



# Docker update Service

## Update image

docker service update --image springio/client --force client
docker service update --image springio/eureka --force eureka
docker service update --image springio/eureka --force eureka-backup

## Update replicas

docker service update --replicas 2 client
docker service update --replicas 2 config
