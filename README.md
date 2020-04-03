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
  
  https://mhausenblas.info/kube-rpi/

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
  

    



