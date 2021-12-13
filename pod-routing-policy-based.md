```
kubectl  edit  ippool default-ipv4-ippool
 .... change CIDR to  cidr: 192.168.2.0/24
```
Install calicoctl and create addition ippool

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
```
kubectl create namespace internal-ns
```
```
kubectl annotate namespace internal-ns "cni.projectcalico.org/ipv4pools"='["internal-pool"]'
```
```
sudo vi /etc/iproute2/rt_tables
 ... add a line... 
 200     pod-route

sudo ip rule add from 192.168.3.0/24 table pod-route     # Degug pod ip-address


$ sudo ip rule ls
0:	from all lookup local
32765:	from 192.168.3.0/24 lookup pod-route
32766:	from all lookup main
32767:	from all lookup default


$ sudo ip route add default via 192.168.2.5 dev cali09d96a0e9e0 table pod-route

$ sudo ip route flush cache



```
