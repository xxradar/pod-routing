# pod-routing


### Create a gateway pod 
```
kubectl run -it --rm  --privileged=true --image xxradar/hackon gateway
```
Inside the pod
```
sysctl net.ipv4.ip_forward=1

ip link add vxlan1 type vxlan id 1 remote <client1_pod_ip> dstport 4789 dev eth0
ip link set vxlan1 up
ip addr add 10.0.0.110/24 dev vxlan1

ip link add vxlan2 type vxlan id 2 remote <client2_pod_ip>  dstport 4789 dev eth0
ip link set vxlan2 up
ip addr add 10.0.0.111/24 dev vxlan2

apt-get install -y    iptables
iptables -t nat -A POSTROUTING -j MASQUERADE
tcpdump -i eth0 -n 
```

### Create a client1 pod
```
kubectl run -it --rm  --privileged=true --image xxradar/hackon client1

ip link add vxlan1 type vxlan id 1 remote <gateway_pod_ip> dstport 4789 dev eth0
ip link set vxlan1 up
ip addr add 10.0.0.120/24 dev vxlan1
route delete default
route add default gw 10.0.0.110 dev vxlan1
```

### Create a client1 pod
```
kubectl run -it --rm  --privileged=true --image xxradar/hackon client2
ip link add vxlan1 type vxlan id 2 remote <gateway_pod_ip>  dstport 4789 dev eth0
ip link set vxlan1 up
ip addr add 10.0.0.120/24 dev vxlan1
route delete default
route add default gw 10.0.0.111 dev vxlan1
```



