## Demo and research script
Testing is done with calico cni

```
export PREFIX="pod4"
export NS=$PREFIX"-ns"
export IP="192.168.122.34"
export GW="192.168.122.33"

# Create an new network namespace
sudo ip netns add $NS

# Create a network interface virtual ethernet pair
sudo ip link add $PREFIX"-host" type veth peer name $PREFIX"-client"
# sudo ip link show

# Move one interface of the pair to the networkwork namespace
sudo ip link set $PREFIX"-client" netns $NS

# Configure the network interface in netns
sudo ip netns exec $NS ip addr add $IP/30 dev $PREFIX"-client"
sudo ip netns exec $NS ip link set lo up
sudo ip netns exec $NS ip link set $PREFIX"-client" up
sudo ip netns exec $NS ip a

# configure the network interface in host namespace
sudo ip addr add $GW/30 dev $PREFIX"-host"
sudo ip link set $PREFIX"-host" up

# routing
sudo ip netns exec $NS ip route add 0.0.0.0/0 via $GW
sudo ip netns exec $NS ip route
sudo ip route add $IP/32 scope link dev $PREFIX"-host"


# Testing
sudo ip netns exec $NS ping 8.8.8.8
```


## Testing
- kubectl apply -f https://raw.githubusercontent.com/xxradar/kubernetes_learning/master/nginx-deployment.yaml
- kubectl apply -f https://raw.githubusercontent.com/xxradar/kubernetes_learning/master/nginx-expose-clusterip.yaml
- sudo ip netns exec $NS ping 8.8.8.8 
- sudo ip netns exec $NS curl https://8.8.8.8 #google
- sudo ip netns exec $NS http://10.99.38.41  #k8s service
- pod to pod is  working


## Cleanup
```
sudo ip netns del $NS
sudo ip link  del $PREFIX"-host
```


