### Change ippool cidr (testing only)
```
kubectl  edit  ippool default-ipv4-ippool
 .... change CIDR to  cidr: 192.168.2.0/24
```
### Install calicoctl
```
curl -o kubectl-calico -O -L  "https://github.com/projectcalico/calicoctl/releases/download/v3.21.2/calicoctl" 
chmod +x kubectl-calico
sudo mv ./calicoctl /usr/local/bin/calicoctl
```
```
sudo mkdir /etc/calico
sudo vi /etc/calico/calicoctl.cfg 

apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  datastoreType: "kubernetes"
  kubeconfig: "/home/ubuntu/.kube/config"
```
### Create additional ippool

```
calicoctl apply -f -<<EOF
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: internal-pool
spec:
  cidr: 192.168.3.0/24
  blockSize: 29
  ipipMode: Always
  natOutgoing: false
EOF
```
### Create a router pod in default namespace (in default pool)
```
kubectl run -it --rm --privileged=true --image xxradar/hackon router
    sysctl net.ipv4.ip_forward=1
    ip -4 addr show eth0 | grep -oP "(?<=inet\s)\d+(\.\d+){3}" >ip.txt
    apt-get install -y iptables 
    iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to $(cat ip.txt)
    iptables -t nat -A POSTROUTING -j MASQUERADE
    iptables -L -t nat  #shoud show the SNAT rule
    tcpdump -i eth0 
```
### In an other terminal 
```
kubectl create namespace internal-ns
```
```
kubectl annotate namespace internal-ns "cni.projectcalico.org/ipv4pools"='["internal-pool"]'
```
### Create sourced based routing rules
```
sudo vi /etc/iproute2/rt_tables
 ... add a line... 
 200     pod-route

sudo ip rule add from 192.168.3.0/24 table pod-route     # Degug pod ip-address


sudo ip rule ls
0:	from all lookup local
32765:	from 192.168.3.0/24 lookup pod-route
32766:	from all lookup main
32767:	from all lookup default


sudo ip route add default via 192.168.2.5 dev cali09d96a0e9e0 table pod-route

sudo ip route flush cache
```
### Create testing pod
```
kubectl run --rm -it debug --image xxradar/hackon  --namespace internal-ns
```


### Remark
This will probably only work on a 1 node k8s cluster. Creating VXLAN tunnels between nodes and router node can solve this.

