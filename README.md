# pi-k3s
Install and use K3s Kubernetes Rancher on Raspberrypi

Enable container features in Kernel
Edit /boot/cmdline.txt on both the Raspberry Pi nodes and add the following to the end of the line:

  cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory

Install

   curl -sfL https://get.k3s.io | sh -
   
sudo k3s kubectl get node -o wide

Get token
cat /var/lib/rancher/k3s/server/node-token

Add token as variable
 NODETOKEN=K108b8................
 
 
 on node
 curl -sfL https://get.k3s.io | sh -
 k3s agent --server https://192.168.1.5:6443 --token ${NODETOKEN}
 
https://github.com/rancher/k3s

https://rancher.com/docs/k3s/latest/en/quick-start/

https://collabnix.com/get-started-with-k3s-a-lightweight-kubernetes-distribution-for-raspberry-pi-cluster/
