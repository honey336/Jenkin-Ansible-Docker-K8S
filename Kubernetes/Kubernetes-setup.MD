# Setup Kubernetes (K8s) Cluster on AWS


<<kops
Honeywell Technologies  
honeybussy336@gmailcom  
kops
Setup Kubernetes (K8s) Cluster on AWS Using KOPS

#!/bin/bash
#1) Create Ubuntu EC2 instance in AWS and create kops user
   sudo adduser kop
   sudo echo "kop  ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/kop
   sudo su - kop

#2) install AWSCLI

 sudo apt update -y
 sudo apt install unzip wget -y
 sudo curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
 sudo apt install unzip python -y
 sudo unzip awscli-bundle.zip
 sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
 
 
#3) Install kops software on ubuntu instance:

   #Install wget if not installed
   sudo apt install wget -y
   sudo wget https://github.com/kubernetes/kops/releases/download/v1.22.0/kops-linux-amd64
   sudo chmod +x kops-linux-amd64
   sudo mv kops-linux-amd64 /usr/local/bin/kops
 
#4) Install kubectl = command line utility for kubernetes

 sudo curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
 sudo chmod +x ./kubectl
 sudo mv ./kubectl /usr/local/bin/kubectl
 
 
 
 aws s3 mb s3://honeywell.local
 aws s3 ls

# 5) Create an IAM role from AWS Console or CLI with below Policies.

  AmazonEC2FullAccess  = ecs instances
  AmazonS3FullAccess   = key-value-store 
  IAMFullAccess        = 
  AmazonVPCFullAccess  = vpc 
    user cannot manage billing 


Then Attach IAM role to ubuntu server from Console Select KOPS Server --> Actions --> 
    Instance Settings/Security --> Attach/Replace IAM Role --> Select the role which
You Created. --> Save.
   


#6) create an S3 bucket Execute below commond in KOPS Server use unique bucket name if you get bucket name exists error.

  
  aws s3 mb s3://landmark.local
  aws s3 ls
  
    ex:
  S3 bucket name should be unique across AWS accounts 
  aws s3 mb s3://simonlegah.k8s.local    s3://nubong.k8s.local
     
  Expose environment variable:

    # Add env variables in bashrc
    vi .bashrc
  
  # Give Unique Name And S3 Bucket which you created.
  export NAME=honey.k8s.local
  export KOPS_STATE_STORE=s3://honey.k8s.local
 
    source .bashrc
  
#7) Create sshkeys before creating cluster

    ssh-keygen
 

# 8)Create kubernetes cluster definitions on S3 bucket

kops create cluster --zones us-east-2b --networking weave --master-size t2.medium --master-count 1 --node-size t2.medium --node-count=2 ${NAME}
  
# 8b. copy sshkeys into the cluster
kops create secret --name ${NAME} sshpublickey admin -i ~/.ssh/id_rsa.pub

# 9) Create kubernetes cluser

   kops update cluster --name honey.k8s.local --yes --admin

# 10) Validate your cluster(KOPS will take some time to create cluster ,Execute below commond after 10  mins)

     kops validate cluster honey.k8s.local
 
# 11) To list nodes

    kubectl get nodes 
  
 # connect to your master node
sh -i ~/.ssh/id_rsa ubuntu@ipAddress
ssh -i ~/.ssh/id_rsa ubuntu@3.90.203.23
#Get the master node ipAddress 

 kubectl get all 
 kubectl get pod
 
  # 12) To Delete Cluster

   kops delete cluster --name=${NAME} --state=${KOPS_STATE_STORE} --yes  
   
kops update cluster --name honey.k8s.local --yes --admin
   
#### Deploying Nginx pods on Kubernetes
1. Deploying Nginx Container
    ```sh
    kubectl run sample-nginx --image=nginx --replicas=2 --port=80
    # kubectl run simple-devops-project --image=yankils/simple-devops-image --replicas=2 --port=8080
    kubectl get pods
    kubectl get deployments
   ```

1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them.
   ```sh
   kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
   # kubectl expose deployment simple-devops-project --port=8080 --type=LoadBalancer
   kubectl get services -o wide
   ```
