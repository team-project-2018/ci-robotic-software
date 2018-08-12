# Continuous Integration of Robotic Software

Project by: Lukas Graber, Dorotea Lleshaj, (insert your name here) under Prof. Dr. Giovanni Beltrame

## TODO for Team

- insert your names at top of readme file  
- you can make an additional text file, that contains the steps to reproduce and send it to the Professor  
- Maybe develop turtlesim bash file (you would need to create bash file, include this file in Dockerfile and add all the necessary files to the repository) and you can put it on here and add in steps to reproduce where to use this bash file and what it will do BEING WORKED ON BY DOROTEA 
- Try to reproduce the system yourself by following the steps and change the readme file if something is unclear  
- If you can reproduce, you can even make a tutorial video and put it on youtube and put the link in the readme, but that might be overkill  
- If everyone wants to contribute to this repository, maybe one can make a text file containing the steps to reproduce, another one can create a turtlesim application in bash file and the third can create the youtube video  
- Someone can create a file with definitions of certain words like: Docker engine, Docker-compose, Kubernetes, Docker Swarm, kubectl, kubeadm,...  DONE
- Before deadline remove these TODO lines from README

## Introduction
This project may be a first step on designing a system for Continuous Integration of Robotic Software for professional use cases. In this README we collect all the necessary information for our system, important files, commands and a list of steps to reproduce our system.

## Prerequisites 
- You should have Ubuntu installed or at least simulated on a virtual machine for our system to work properly.  
- If you want to use Kubernetes you also need enough RAM, because you are supposed to swap off the memory for Kubernetes to work.  
- connect all machines of the cluster to the same router

## Done
- Gitlab registry and gitlab-runner in Docker container  
- Gitlab CI pipeline  
- Gitlab registry  
- Using docker swarm for deployment in Gitlab  
- monitoring different machines in cluster with Prometheus (swarmprom)

## Future Projects
- make turtlesim work  
- use real robotic software  
- building our system with Kubernetes instead of Docker Swarm

## Important files 
There are a few essential files that bring our system into being:  
- /etc/hosts				-> file that maps ip to artificial name, important for deployment  
- /etc/docker/daemon.json        	-> lists insecure registries that should be allowed, set experimental to use Prometheus  
- /srv/gitlab/config/gitlab.rb 	  	-> contains all configurations of gitlab (on host machine)  
- docker-compose.yml      		-> docker-compose will spin up essential containers like gitlab, gitlab-runner  
- /srv/gitlab-runner/config/config.toml	-> contains all configurations of runners  
- .gitlab-ci.yml 			-> defines pipeline that runners should execute  
- Dockerfile				-> defines our application as image  

## Important tools 
- Docker					-> Engine to build the containers and therefore our whole system  
- Docker-compose				-> can set up multiple containers in a specified file and put them on the same network  
- Docker Swarm					-> cluster orchestration tool integrated with docker engine (easier set up than Kubernetes)  
- Kubernetes with kubeadm,kubectl, kubelet	-> more sophisticated yet more difficult cluster orchestration tool  

## Steps to reproduce:
The following steps document the process of setting up our system as exactly as possible:  
1. Install docker  
https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce-1  
2. Install docker-compose  
https://docs.docker.com/compose/install/  
3. Find out your ip that router assigns to your machine: use nmcli and look for the line that starts with inet4. Note the ip that follows after inet4 - this is the ip that router assigns to pc  
4. Map this ip to some artificial name (in our case master) in /etc/hosts (see hosts file in repository) - later we want to access gitlab instance over this ip but instead of using ip we want to type artificial name in browser  
5. Create docker-compose.yml file from our repository and start containers with docker-compose up -d(you have to be in same folder as docker-compose.yml file when typing this command, -d for background)  
6. type: master:7070 in browser and log in to Gitlab for first time (may take a while to pull images and start containers)  
7. Create a project and initialize a git repository to pull and push to remote gitlab repository, just follow the advice of gitlab page to initialize and configure git on local machine  
8. Register runner  
    - Go inside gitlab-runner container with: docker exec -it "name of container" bash (to find out name of container type docker ps and look for some name with gitlab-runner in it)  
    - once inside container type: gitlab-runner register and follow advice: use image gitlab:dind for service dind, you can find token in gitlab container under Admin Area -> Runners or Settings -> CI/CD -> Runner settings  
    - If runner is not available in gitlab check clone_url, network_mode, volume in .docker-compose.yml and /etc/gitlab-runner/config.toml inside gitlab
    - The registration process should look like this:  
    
	root@8a6bdc06e08b:/# gitlab-runner register  
	Running in system-mode.                            

	Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):  
	http://gitlab:7070  

	Please enter the gitlab-ci token for this runner:  
	iGeFrZz7UrncT4oQDEg6  

	Please enter the gitlab-ci description for this runner:  
	[8a6bdc06e08b]: my-runner  

	Please enter the gitlab-ci tags for this runner (comma separated):  
	my-runner  

	Registering runner... succeeded                     runner=iGeFrZz7  
	Please enter the executor: docker-ssh, ssh, docker, shell, virtualbox, docker+machine, docker-ssh+machine, kubernetes, parallels:  
	docker  

	Please enter the default Docker image (e.g. ruby:2.1):  
	gitlab/dind  

	Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!  

9. Now a runner is registered. But to use it in Gitlab, we still need to change some configuration for the registered runner (see config.toml file in repository):  
    - Find out the network on which gitlab is sitting with docker network ls and search for network whose first part is called like the folder where docker-compose file is sitting, make docker network inspect this_network and look if gitlab and gitlab-runner container is in there -> this is the network the registered runner needs to be registered to  
    - change configuration for runner to work in ci pipeline  
    clone_url: clone_url = "http://gitlab:7070"  
    network: network_mode = "tutorial_default" (in our case network is called tutorial_default)  
    volume: volumes = ["/cache","/var/run/docker.sock:/var/run/docker.sock"]  
10. Integrate gitlab image registry  
- The registry url is already set in the docker-compose file (you could also change line in gitlab.rb file)  
- Add insecure registries in /etc/docker/daemon.json (see daemon.json file in repository)  
- Restart docker engine: sudo service docker restart  
11. Other machines can connect to Gitlab instance over browser:  
Type master:7070 in browser and register to gitlab -> master can add people to project and assign rank (As administrator: Go to project -> Settings -> Members -> Add registered machines)  
12. Push .gitlab-ci.yml, Dockerfile and other files (e.g. hello.txt) used in Dockerfile to Gitlab and check pipeline out: Pipeline will fail on second job but image should be pushed on first stage -> check out image registry in Gitlab  
13. Initialize Docker Swarm cluster: docker swarm init  
14. Find out join command for other machines:  
docker swarm join-token worker (on master) -> use this command on worker to join cluster (you can copy this command in readme in gitlab, so that other people on team can join the cluster)  
15. If this is not working, maybe there is a firewall issue, so you have to open ports:  
https://www.digitalocean.com/community/tutorials/how-to-configure-the-linux-firewall-for-docker-swarm-on-ubuntu-16-04  
16. Log in to gitlab registry to pull and push to image:  
docker login master:4567  
17. Deploy/Create service on swarm leader onto the cluster (we will update service from within the .gitlab-ci.yml file):  
docker service create --mode global --with-registry-auth --name ello master:4567/root/tutorial roscore  
For more information on Docker Swarm: https://docs.docker.com/engine/swarm/swarm-tutorial/create-swarm/ and follow links at bottom of page  
18. Restart pipeline again and check if both jobs are completed  
19. Congratulations: You achieved Continuous Integration!  
20. Develop your own application with robotic software and you achieve Continuous Integration of Robotic Software.

## Changes to be made on worker node for deployment to work
All the machines that should be part of the cluster and do not have gitlab running on them are considered to be workers. These workers need to follow the following steps:  
- they should have Ubuntu and Docker installed  
- they should map the ip of manager to same artificial name in /etc/hosts like manager does  
- they should also set insecure registry in daemon.json to get access to image registry 

## Add monitoring solution
- Clone already existing solution swarmprom from: https://github.com/stefanprodan/swarmprom and follow steps  
- Remember to set experimental in daemon.json file to make prometheus work properly  
- Enter Prometheus over master:9090 and Grafana over master:3000  
- Log in Prometheus and Grafana with username admin and password admin  
- If one node is not visible, this could be because of strange docker version

## Test if everthing is working
- Check if container is deployed: docker ps and look for some container called ello  
- Go inside container: docker exec -it "name of container" bash  
- Print hello.txt (when inside container): cat hello.txt  
- Change hello.txt in gitlab and let pipeline build image, and update service, changes will be deployed on every machine  
- Check again the hello.txt file: docker exec -it "name of container" bash (to get name of container you can just type ello and then push tab then auto-completion)  
- Print hello.txt (when inside container): cat hello.txt and look if changed  
- Type in browser master:9090 to enter Prometheus and master:3000 to enter Grafana and check if monitoring is working 

## Add ssh authentication
To make your workflow faster you can add ssh key authentication for Gitlab. Afterwards you can just push and pull from the gitlab repository if git is configured for gitlab instance.  
- Create ssh key pair: https://docs.gitlab.com/ee/ssh/README.html  
- Add public key to Gitlab: User settings -> ssh keys  
For more information: https://docs.gitlab.com/ee/ssh/ 

## Approach with Kubernetes
This approach is currently not working properly. If someone wants to pick up our system and use Kubernetes, feel free to use this as a starting guide.  
- install Kubernetes with kubeadm, kubectl, kubelet  
https://kubernetes.io/docs/tasks/tools/install-kubeadm/  
- initialize cluster, join workers and make cluster recoverable  
http://stytex.de/blog/2018/01/16/how-to-recover-self-hosted-kubeadm-kubernetes-cluster-after-reboot/  
- if warning that docker version too high, don't care but if error because of swap, turn it off with: sudo swapoff -a -> kubeadm reset -> initialize with kubeadm again  
- connect cluster to gitlab  
https://docs.gitlab.com/ee/user/project/clusters/  
-> The system is now ready  
Issue: no kubectl in docker service, kubectl not able to find DNS-server
