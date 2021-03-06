# KubernetesNetworkTroubleshooting
Troubleshooting Kubernetes Networking with Calico.
I created this repo to help prepare for the CKA exam:


## PS AUX Command:
The ps aux command is a tool to monitor processes running on your Linux system. 
```
ps -aux | grep kubelet
```

## SystemCTL Command:
How to view status of a service on Linux using systemctl
```
systemctl status kubelet.service -l
```

## JournalCTL Command:
journalctl command in Linux is used to view systemd, kernal and journal logs.
```
journalctl -u kubelet
```

## Deployments

Set image of a deployment nginx:
```kubectl set image deploy nginx nginx=nginx:1.18```
Scale deployment nginx to 4 replicas and record the action:
```kubectl scale deploy nginx --repliacs=4 --record```
Get events in the current namespace:
```kubectl get events```

## Scheduling
Get taints of node node01:
```kubectl describe node node01 | grep -i Taints:```
Label node node01 with label size=small:
```kubectl label nodes node01 size=small```
Default static pods path:
```/etc/kubernetes/manifests```
Check pod nginx logs:
```kubectl logs nginx```
Check pod logs with multiple containers:
```kubectl logs <pod_name> -c <container_name>```

## Calico Stuff in Progress:

Probably the most common starting point is with 'ifconfig':
```
ifconfig -a
```

<img width="527" alt="Screenshot 2021-05-31 at 10 50 34" src="https://user-images.githubusercontent.com/82048393/120180526-c4f4cc80-c203-11eb-86bb-c371eef93fa6.png">

You can do this for a specific pod via the below command:

```
kubectl exec calico-node-75pfg -n calico-system -- ip a
```

<img width="744" alt="Screenshot 2021-05-31 at 13 42 01" src="https://user-images.githubusercontent.com/82048393/120195066-4190a680-c216-11eb-8df7-7896cc598e96.png">

Another option would be simply print the route to the pod ip address on the node:

To find the cluster IP address of a Kubernetes pod, use the kubectl get pod command on your local machine, with the option -o wide. This option will list more information, including the node the pod resides on, and the pod???s cluster IP.

```
kubectl get pods -n storefront -o wide
```

<img width="938" alt="Screenshot 2021-05-31 at 10 16 24" src="https://user-images.githubusercontent.com/82048393/120179285-464b5f80-c202-11eb-95f4-4c2cd8709fac.png">

That route (route -n | grep podIP) should give the cali* interface of the pod.

```
route -n | grep 192.168.4.173
```

Once that is found, it is just one command to find the corresponding chain in iptables path. 
The key thing is to remember is how Calico programs the chain. 

- All traffic to the pod go through the chain (cali-tw-<cali**>). 
- All traffic from the pod go through the chain (cali-fw-<cali**>). 

Refer to the following terminal video to understand how to interpret the traffic going out from the pod:



Get DaemonSets:
```
kubectl get ds -A
```

<img width="900" alt="Screenshot 2021-05-31 at 10 02 17" src="https://user-images.githubusercontent.com/82048393/120178448-6c243480-c201-11eb-9c65-9fb4f8d3cdfc.png">

Discover pods in Calico System:
```
kubectl get pods -n calico-system
```

<img width="543" alt="Screenshot 2021-05-31 at 10 03 56" src="https://user-images.githubusercontent.com/82048393/120178516-7fcf9b00-c201-11eb-9333-3ffa91d29fe3.png">

Exec into a Calico Node and get the ARP table output:
```
kubectl exec -it -n calico-system calico-node-75pfg -- /bin/bash
```
```
arp -a
```

<img width="747" alt="Screenshot 2021-05-31 at 10 07 52" src="https://user-images.githubusercontent.com/82048393/120178791-c3c2a000-c201-11eb-91b5-12d46de9a640.png">

Confirm output of the ip routing table
```
ip route
```

<img width="469" alt="Screenshot 2021-05-31 at 10 08 13" src="https://user-images.githubusercontent.com/82048393/120178938-ec4a9a00-c201-11eb-980b-896d320ecfbc.png">

Confirm which IP addresses are assigned to each of your pods (and which node they are running on):
```
kubectl get pods -A -o wide
```

<img width="1378" alt="Screenshot 2021-05-31 at 10 14 58" src="https://user-images.githubusercontent.com/82048393/120179125-1b610b80-c202-11eb-93ca-7f16fd6d1226.png">

If a pod is in 'ImagePullBackOff' status, we can 'describe' what has gone wrong with the pod:
```
kubectl describe pod tigera-internal-app-cvn98 -n tigera-internal
```

<img width="784" alt="Screenshot 2021-05-31 at 14 09 43" src="https://user-images.githubusercontent.com/82048393/120198324-e2349580-c219-11eb-9e22-2bd7b60f415b.png">

You can run kubectl edit to change the image pulled for your pulled if incorrect/misconfigured:
```
kubectl edit pod tigera-internal-app-cvn98 -n tigera-internal
```
<img width="644" alt="Screenshot 2021-05-31 at 14 13 52" src="https://user-images.githubusercontent.com/82048393/120200139-e792df80-c21b-11eb-9469-4b25a93c1d01.png">



If this fails, you can try deleting all pods within that specific namespace. Kubernetes will attempt to recreate those pods:

```
kubectl delete --all pods --namespace=tigera-internal | grep tigera-internal
```

Then you can check if those pods re-create successfully:

```
kubectl get pods -A | grep tigera-internal
```

<img width="901" alt="Screenshot 2021-05-31 at 14 19 25" src="https://user-images.githubusercontent.com/82048393/120199870-971b8200-c21b-11eb-88a1-0f00f27c3816.png">


  

Let's do the same for the calico-system namespace:
```
kubectl get pods -n calico-system -o wide
```

<img width="1012" alt="Screenshot 2021-05-31 at 10 17 44" src="https://user-images.githubusercontent.com/82048393/120179385-6aa73c00-c202-11eb-8c48-b22062d4e75b.png">

If we were to exec back into the same calico node as earlier, we can confirm the chain of actions for a specific cali interface:
```
iptables-save | grep cali3fa04d10d5a
```

<img width="1348" alt="Screenshot 2021-05-31 at 10 22 19" src="https://user-images.githubusercontent.com/82048393/120179566-ab9f5080-c202-11eb-8419-cd0dfe1c6bf0.png">

Get error-specific logs for the calico nodes running within calico-system:
```
kubectl logs calico-node-75pfg -n calico-system -c calico-node | grep ERROR
```

<img width="1720" alt="Screenshot 2021-05-31 at 13 31 04" src="https://user-images.githubusercontent.com/82048393/120193561-7d2a7100-c214-11eb-9e4d-eda96c611b89.png">

If you haven't done so already, we recommend installing calicoctl command-line utility:
https://docs.projectcalico.org/getting-started/clis/calicoctl/install

Use the following command to download the calicoctl binary.
```
curl -o calicoctl -O -L  "https://github.com/projectcalico/calicoctl/releases/download/v3.19.1/calicoctl" 
```

Set the file to be executable.
```
chmod +x calicoctl 
```

<img width="913" alt="Screenshot 2021-05-31 at 13 34 10" src="https://user-images.githubusercontent.com/82048393/120193953-ef02ba80-c214-11eb-8434-a1635bccd108.png">



You can check on the status of calico nodes by running the below command:
```
sudo ./calicoctl node status
```

<img width="493" alt="Screenshot 2021-05-31 at 11 26 33" src="https://user-images.githubusercontent.com/82048393/120179978-27999880-c203-11eb-991a-cd6afac60ec3.png">

Any external nodes to the cluster (non-peered) will show as 'global' under 'PEER TYPE':

![Screenshot 2021-05-31 at 10 39 49](https://user-images.githubusercontent.com/82048393/120180158-57e13700-c203-11eb-9420-5dd0d21b1696.png)


To get a full .YAML output for your IPPool Configuration, run the below command:
```
./calicoctl get ippools default-ipv4-ippool -o yaml
```

<img width="530" alt="Screenshot 2021-05-31 at 10 52 14" src="https://user-images.githubusercontent.com/82048393/120180731-09806800-c204-11eb-9523-a98e9db881fa.png">

To confirm the interface and network where your workloads are running from, run the below command
```
./calicoctl get workloadEndpoint -A
```

<img width="1046" alt="Screenshot 2021-05-31 at 11 07 33" src="https://user-images.githubusercontent.com/82048393/120181029-68de7800-c204-11eb-942b-615be9a90bcd.png">

Similarly, in kubectl you can get the IP address and Node IP for all pods:
```
kubectl get po -o wide -A
```

<img width="1452" alt="Screenshot 2021-05-31 at 11 09 05" src="https://user-images.githubusercontent.com/82048393/120181158-93303580-c204-11eb-997a-f9b85072a70a.png">


Get an output of debug changes for the Nodes
```
./calicoctl -l debug get nodes
```

<img width="1779" alt="Screenshot 2021-05-31 at 14 31 38" src="https://user-images.githubusercontent.com/82048393/120201083-f0d07c00-c21c-11eb-8c1d-cb40d5ef524b.png">

## Deployments revisited:

Set image of a deployment nginx:
```
kubectl set image deploy nginx nginx=nginx:1.18
```


Scale deployment nginx to 4 replicas and record the action:
```
kubectl scale deploy nginx --repliacs=4 --record
```


## Cluster Maintenance:

Drain node node01 of all workloads:
```
kubectl drain node01
```
Make the node schedulable again:
```
kubectl uncordon node01
```
Upgrade cluster to 1.18 with kubeadm:
```
kubeadm upgrade plan
apt-get upgrade -y kubeadm=1.18.0???00
kubeadm upgrade apply v1.18.0
apt-get upgrade -y kubelet=1.18.0???00
systemctl restart kubelet
```
Backup etcd:
```
export ETCDCTL_API=3
etcdctl \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /tmp/etcd-backup.db
```
Restore etcd:
```
ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db --data-dir /var/lib/etcd-backup
```
After edit /etc/kubernetes/manifests/etcd.yaml and change /var/lib/etcd to /var/lib/etcd-backup.
