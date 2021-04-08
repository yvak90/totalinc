
I have taken two open source applications to run on kubernetes cluster. Both are java based applications. Here are the github links of those opensource applications.
Game of life: https://github.com/wakaleo/game-of-life ; 
Spring pet clinic: https://github.com/spring-projects/spring-petclinic
I have forked above projects to my github. From then I have created docker images for both of the applications and stored them in my public docker hub registry. I have used these images in kubernetes spec (yaml) for creating containers. Due to access related issues in my laptop, i have created all the stuff on my AWS environment. I have documented everything and it is as follows.
I have created four VMs on AWS. One is especially for building docker images and to push them to dockerhub registry.The other three VMs form a Kubernetes cluster. One is k8s Master and the other two are nodes.


![image](https://user-images.githubusercontent.com/58933081/114049561-299d4600-98a9-11eb-8926-07aa6697d488.png)


 
Docker file for game of life application is https://github.com/yvak90/totalinc/blob/main/gol-docker/Dockerfile
Docker file for spring petclinic application is 
https://github.com/yvak90/totalinc/blob/main/spc-docker/Dockerfile
All the code related to Dockerfiles and yaml specs of kubernetes are stored in below github link. Instructions of execution are mentioned in this file in detail rather in github. In Documentation, for better understanding i have attached screenshots of my execution.
https://github.com/yvak90/totalinc




1. Steps for building docker images:
Building game of life Docker image. For Dockerfile of game of life, refer here https://github.com/yvak90/totalinc/blob/main/gol-docker/Dockerfile
In the docker file iam building the package using mvn and copying the war file into the webapps folder of the tomcat. 
 
![image](https://user-images.githubusercontent.com/58933081/114049784-5e110200-98a9-11eb-94c3-8de598a8d824.png)
![image](https://user-images.githubusercontent.com/58933081/114049820-65d0a680-98a9-11eb-8e5e-aa03b615109c.png)


Building Docker image of spring pet clinic:
For Dockerfile of spring pet clinic, refer here https://github.com/yvak90/totalinc/blob/main/spc-docker/Dockerfile
In the docker image, i am adding jar file to the image from my AWS S3 bucket and running the jar file using CMD instruction.
 
 ![image](https://user-images.githubusercontent.com/58933081/114049930-7b45d080-98a9-11eb-812e-2202e29d206c.png)

![image](https://user-images.githubusercontent.com/58933081/114049999-8993ec80-98a9-11eb-95aa-bf68b9583897.png)

 

Now we have build the images for both the applications and here are the steps to create tag for the images and push them to docker hub registry.
Inorder to push to dockerhub registry, we need to login using docker login:
 
![image](https://user-images.githubusercontent.com/58933081/114050055-93b5eb00-98a9-11eb-9459-27fe7594483b.png)

![image](https://user-images.githubusercontent.com/58933081/114050105-9d3f5300-98a9-11eb-9ab7-8e9f676fbc6b.png)

![image](https://user-images.githubusercontent.com/58933081/114050130-a3353400-98a9-11eb-8d7e-7045e291e8a6.png)

 
$ docker image tag spc:1.0 yvak90/gameoflife:3.0
$ docker push yvak90/gameoflife:3.0

![image](https://user-images.githubusercontent.com/58933081/114050184-adefc900-98a9-11eb-9b69-cd476a0c1a8d.png)
 
$ docker image tag spc:1.0 yvak90/petclinic:3.0
$ docker push yvak90/petclinic:3.0
Now we have successfully created docker images and pushed to docker registry and can pulled from using below commands.
https://hub.docker.com/repository/docker/yvak90/petclinic
https://hub.docker.com/repository/docker/yvak90/gameoflife
We can pull images using commands
docker pull yvak90/petclinic:1.0
docker pull yvak90/gameoflife:1.0

2. Setting Kubernetes Cluster:
I have created 3 VMs in AWS inorder to create a kubernetes cluster. In all the 3 nodes I have installed docker run time using following script and initialized docker with following commands

 $ curl -fsSL https://get.docker.com -o get-docker.sh
 $ sh get-docker.sh
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl status docker
 
![image](https://user-images.githubusercontent.com/58933081/114050266-bfd16c00-98a9-11eb-9fcc-c5895e2f3169.png)
![image](https://user-images.githubusercontent.com/58933081/114050304-c5c74d00-98a9-11eb-8348-51fa97e58542.png)
![image](https://user-images.githubusercontent.com/58933081/114050337-ce1f8800-98a9-11eb-9f04-d955fc46f6f1.png)
 

 

After installing docker runtime, installed kubelet kubeadm and kubectl using below commands.
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl


After installing Docker runtime, kubectl kubeadm kubelet in master and nodes as shown above. Login into master node and execute
$ sudo kubeadm init
 
![image](https://user-images.githubusercontent.com/58933081/114050415-e099c180-98a9-11eb-893c-f16fa3535044.png)

After executing init command in k8s Master, we get following set to statements printed on console that are to be executed in master in order to start a cluster. Execute below commands on k8s Master. 




To join node to k8s cluster. Execute below command on node.
 
![image](https://user-images.githubusercontent.com/58933081/114050508-f4ddbe80-98a9-11eb-9cc8-7c03c928efd3.png)


Executed above join command on node.




Weave net can be installed in k8s Master using following command
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
K8s cluster is ready.
 

Running Game of Life Application:
Now execute kubectl apply command for replicaset and service file as shown below.  I have written a replicaset for game of life application and service file for the application to create a named label selector. For these yaml files, refer here.
https://github.com/yvak90/totalinc/tree/main/gameoflife-rs

 ![image](https://user-images.githubusercontent.com/58933081/114050661-18a10480-98aa-11eb-909a-c82cba708587.png)

![image](https://user-images.githubusercontent.com/58933081/114050691-1dfe4f00-98aa-11eb-86c6-89c32ed1794b.png)
 
Now game of life application is running successfully on 32766 port as shown above. We can access the application through <Public ipv4address>:32766/gameoflife> of all k8s master and nodes.   

![image](https://user-images.githubusercontent.com/58933081/114051094-6f0e4300-98aa-11eb-8404-3bad171febe7.png)

![image](https://user-images.githubusercontent.com/58933081/114051140-77ff1480-98aa-11eb-82ab-a293a49f805c.png)


Running spring pet clinic application:
Now execute kubectl apply command for replicaset and service file as shown below.  I have written a replicaset for spring pet clinic application and service file for the application to create a named label selector. For these yaml files, refer here.
https://github.com/yvak90/totalinc/tree/main/petclinic-rs

![image](https://user-images.githubusercontent.com/58933081/114051373-aa107680-98aa-11eb-84b0-130fbd30694a.png)

 
Now pet clinic application is running successfully on 32767 port as shown above. We can access the application through <Public ipv4address>:32767> of all k8s master and nodes.

 



