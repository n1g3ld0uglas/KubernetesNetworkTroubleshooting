# KubernetesNetworkTroubleshooting
Troubleshooting our Kubernetes Networking with Calico

Probably the most common starting point is with 'ifconfig':
```
ifconfig -a
```

<img width="527" alt="Screenshot 2021-05-31 at 10 50 34" src="https://user-images.githubusercontent.com/82048393/120180526-c4f4cc80-c203-11eb-86bb-c371eef93fa6.png">




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

Similarly, you can filter pod search down to a specific namespace:
```
kubectl get pods -n storefront -o wide
```

<img width="938" alt="Screenshot 2021-05-31 at 10 16 24" src="https://user-images.githubusercontent.com/82048393/120179285-464b5f80-c202-11eb-95f4-4c2cd8709fac.png">

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



