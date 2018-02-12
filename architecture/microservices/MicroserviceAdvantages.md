
# Microservice Advantages


1. language independent
2. fast interations
3. small teams
4. fault isolation
5. pair well with containers
6. SCALABLE

# Disadvantage
1. complex networks
2. over head： databases and servers
	

Docker vs VM


Container Orchestration：
1. Azure Container Services
2. Amazon ECS
3. MARATHOn
4. docker
5. kubernetes
6. Google GKE


Kubernets Vocab
Node ： kubelet ， communicates with master ； run pods
pods : runs 1+ containers,  exist on a pod
service:  handle request; usually a load balancer
Deployment: Defines desired state - kubernet handle the rest



# docker practices:

## install docker on centos
wget https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-17.09.0.ce-1.el7.centos.x86_64.rpm
