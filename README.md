# pi-k3s
# Install and use K3s Kubernetes Rancher on Raspberrypi


## Enable container features in Kernel (all nodes)
Edit /boot/cmdline.txt on both the Raspberry Pi nodes and add the following to the end of the line:

    sudo nano /boot/cmdline.txt
    cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory

## Modify hosts (all nodes)
    sudo nano /etc/hosts
  
  ### Make sure all hosts had a diferent name and include all nodes on cluster, ie
     pi@k3master:~ $ sudo cat /etc/hosts
      127.0.0.1       localhost
      ::1		localhost ip6-localhost ip6-loopback
      ff02::1		ip6-allnodes
      ff02::2		ip6-allrouters

      192.168.42.114	k3master
      192.168.42.115  k3node1
      192.168.42.116  k3node2

  ### Modify hostname (all nodes)
   This will be your node name
      
      pi@k3master:~ $ sudo nano /etc/hostname
      pi@k3master:~ $ sudo cat /etc/hostname
      k3master
  
## --  REBOOT YOUR PI   --
       sudo reboot
       Check hostname is ok: $hostname
       try to ping master:  $ping k3master, make sure you get ping.

## Install server node:

    curl -sfL https://get.k3s.io | sh -

   To check if master node is live
    
    sudo kubectl get nodes
  
    pi@k3master:~ $ sudo kubectl get nodes
    NAME          STATUS   ROLES    AGE     VERSION
    k3master      Ready    master   5h42m   v1.17.4+k3s1
   
    Make sure all system pods are running
    kubectl get pods --all-namespaces
   
    sudo k3s kubectl get node -o wide

## Install worker nodes
  On the master node - get token

    sudo cat /var/lib/rancher/k3s/server/node-token

  ### On worker nodes

    Add token as variable (not mandatory, you can add straight)
    NODETOKEN=K108b8................
 
    sudo curl -sfL https://get.k3s.io  | K3S_TOKEN=xxx K3S_URL=https://server-url:6443 sh -
    K3S_TOKEN , is the token you get in the previous step
    K3S_URL , is your master node ip ...ie. 192.168.42.114 
    
 
 ## Some reference pages
  https://github.com/rancher/k3s
  
  https://rancher.com/docs/k3s/latest/en/quick-start/
  
  https://rancher.com/docs/k3s/latest/en/installation/install-options/
  
  https://kubernetesbyexample.com
  
  

## Install Kube Dashboard
   https://rancher.com/docs/k3s/latest/en/installation/kube-dashboard/
   
## Deploy NGIX - great tutorial
   https://opensource.com/article/20/3/kubernetes-traefik

## Useful
In case you need to unnistall K3 for a new install
      
      cd /usr/local/bin
      
      -- On master --
      chmod +x k3-unistall.sh
      ./k3-unistall.sh
      
      -- On workers ---
      /usr/local/bin
      chmod +x k3s-agent-uninstall.sh
      ./k3s-agent-uninstall.sh

The main configuration yaml for K3 is
        
       sudo cat /etc/rancher/k3s/k3s.yaml

# Test your Kubernetes cluster

## Deploy nginx

    kubectl run mynginx --image=nginx --replicas=3 --port=80
    
  Check deployment works, all pods should look on Running , it may take some time
        
    pi@k3master:~ $ sudo kubectl get pods
    NAME                       READY   STATUS              RESTARTS   AGE
    mynginx-7f79686c94-4c26p   0/1     ContainerCreating   0          28s
    mynginx-7f79686c94-s7w4w   0/1     ContainerCreating   0          28s
    mynginx-7f79686c94-2dw9t   0/1     ContainerCreating   0          28s

    pi@k3master:~ $ sudo su
    root@k3master:/home/pi# k3s kubectl get po
    NAME                       READY   STATUS    RESTARTS   AGE
    mynginx-7f79686c94-s7w4w   1/1     Running   0          2m50s
    mynginx-7f79686c94-2dw9t   1/1     Running   0          2m50s
    mynginx-7f79686c94-4c26p   1/1     Running   0          2m50s

## Exposed deployment
    root@k3master:/home/pi# sudo kubectl expose deployment mynginx --port 80
    service/mynginx exposed
    
## Verify endpoints for PODs
    root@k3master:/home/pi# sudo kubectl get endpoints mynginx
    NAME      ENDPOINTS                                 AGE
    mynginx   10.42.0.22:80,10.42.1.4:80,10.42.2.4:80   53s
    
## Test NGNIX is running
  Pick one of the endpoints IPs and curl'it
        
    root@k3master:/home/pi# sudo curl 10.42.0.22:80

  You should see the NGNIX welcome HTML
        
        <!DOCTYPE html>
        <html>
        <head>
            <title>Welcome to nginx!</title>
            <style>
             body {
                    width: 35em;
                 margin: 0 auto;
                 font-family: Tahoma, Verdana, Arial, sans-serif;
                }
            </style>
         etc.....................

## Delete deployment and service
    kubectl delete deployments mynginx
    kubectl delete services mynginx
  
## Switch namespaces
    kubectl config set-context $(kubectl config current-context) --namespace=XXXX
    XXX can be default, kube-sysem or any other you may have
    
    You can see namespaces with
        kubectl get pods --all-namespaces
    


## Use kubectl from remote nodes
---  I spent hours fixig this issue, so follow steps as described ---

use scp to copy the k3s.yaml, DONT COPY/PASTE as secret will get corrupte

On MASTER node:
     
     cd /etc/rancher/k3s
     
     Now copy k3s.yaml file using scp
     
     sudo scp k3s.yaml remote_user@remote.ip:$HOME/
     
 On WORKER node:
 
    cd $HOME
    nano k3s.yaml
    
    Modify the server: https://127.0.0.1:6443 line to https://your_master_ip:6443
    
    now we need to make a .kube folder on /root
    
        sudo mkdir /root/.kube
    
    Now copy k3s.yaml file to the /root/.kube folder
    
        sudo cp $HOME/.kube/k3s.yaml /root/.kube/k3s.yaml
 
 On your Linux PC
     
   Install kubectl (it may vary depending of your distro, google it!!)
     
     Copy k3s.yaml from master node to your Linux PC
     cd /etc/rancher/k3s
     Now copy k3s.yaml file using scp
     sudo scp k3s.yaml remote_user@remote.ip:$HOME/
    
   Check if you have a folder on your PC call: ~/.kube
   if not create with 
          
          mkidr ~/.kube
   
   Then copy the k3s.yaml as config to ~/.kube
      
          cp $HOME/k3s.yaml ~/.kube/config
   
   
   Now ope the ~/kube/config and modify the cluster IP from 127.0.0.1:6443 to YOUR-master-IP:6443
   
   Now try to run Kubectl
   
        sudo kubectl get nodes
        
   If you get an error, try to run this commands and reboot your pi
   
        sudo dphys-swapfile swapoff && sudo dphys-swapfile uninstall && sudo systemctl disable dphys-swapfile

# Useful kubecommands

     kubectl get services --all-namespaces
     kubectl cluster-info
     
## Deploying openFaas
Checkout OpenFaaS:

    git clone https://github.com/openfaas/faas-netes
    cd faas-netes

Create Namespaces:

    kubectl apply -f ./namespaces.yml

Create secret and store user and new random password (required for UI):
    
    PASSWORD=$(head -c 12 /dev/urandom | shasum| cut -d' ' -f1)
    kubectl -n openfaas create secret generic basic-auth \
        --from-literal=basic-auth-user=admin \
    --from-literal=basic-auth-password="$PASSWORD"
    echo $PASSWORD

Deploy OpenFaaS:
        
    kubectl apply -f ./yaml_armhf
    * note you need to use armhf for pi
 
Confirm if everything is deployed correctly:

    kubectl get deployments --namespace openfaas

Normally OpenFaaS is using Dockerhub, but we want to use our own private registry, so we have to deploy it and expose the port:

    kubectl run registry --image=registry:latest --port=5000 \
    --namespace openfaas

    kubectl expose deployment registry --namespace openfaas \
    --type=LoadBalancer --port=5000 --target-port=5000

Log into your OpenFaaS gateway
Check the gateway is ready

    kubectl rollout status -n openfaas deploy/gateway

If you're using your laptop, a VM, or any other kind of Kubernetes distribution run the following instead:

    kubectl port-forward svc/gateway -n openfaas 8080:8080

This command will open a tunnel from your Kubernetes cluster to your local computer so that you can access the OpenFaaS gateway. There are other ways to access OpenFaaS, but that is beyond the scope of this workshop.

    Your gateway URL is: http://127.0.0.1:8080

Let’s deploy a function:
CLI Login:

    faas-cli login --gateway http://localhost:31112 --password <pwd>

Create a new function from a template — choose a language (more available):

    faas-cli new --lang python hello-python
    faas-cli new --lang java8 hello-java8
    faas-cli new --lang node hello-node

Open yml file and change gateway and image as we want to use our own private registry:
 
    gateway: http://127.0.0.1:8080
    image: localhost:5000/hello-python:latest
    image: localhost:5000/hello-java8:latest
    image: localhost:5000/hello-node:latest


Prior this you will need to install docker if you dont have it.

   This is how for elementar OS, you can googl it for your distro
   
   https://blog.avojak.com/2019/01/18/installing-docker-on-elementary-os/
   
Now we have to build, push and deploy the function, this could be done with one command:

    faas-cli up -f ./hello-python.yml
    faas-cli up -f ./hello-java8.yml
    faas-cli up -f ./hello-node.yml

https://github.com/openfaas/faas-netes/blob/master/chart/openfaas/README.md
    
