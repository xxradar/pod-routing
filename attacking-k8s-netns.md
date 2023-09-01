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

## Additional
In terminal 1:
```
unshare --user --map-root-user --uts --pid --fork --net --mount-proc
```
```
sleep 8888 &
```
In terminal 2: <br>
Find the PID for the `-bash` process in the newly created namespaces
```
lsns | grep bash
```
```
sudo nsenter -t 109110  -a
```
```
ps aux
```
```
root@newhostname:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0  10048  5012 pts/2    S+   13:50   0:00 -bash
root          11  0.0  0.0   3260   828 pts/2    S    13:50   0:00 nc -l 8989
root          12  0.0  0.0   9944  5060 pts/4    S    13:53   0:00 -bash
root          17  0.0  0.0  10620  3324 pts/4    R+   13:53   0:00 ps aux
root@newhostname:/#
```
```
nc -l 9999
```
In terminal 1:
```
ps aux
```
```
root@newhostname:~# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0  10048  5012 pts/2    S    13:50   0:00 -bash
root          11  0.0  0.0   3260   828 pts/2    S    13:50   0:00 nc -l 8989
root          12  0.0  0.0   9944  5060 pts/4    S+   13:53   0:00 -bash
root          18  0.0  0.0   3260   764 pts/4    S    13:54   0:00 nc -l 9999
root          20  0.0  0.0  10620  3204 pts/2    R+   13:54   0:00 ps aux
root@newhostname:~#
```
In terminal 3:
```
sudo ps -ax -n -o pid,netns,utsns,ipcns,mntns,pidns,cmd | grep nc
...
  96176 4026531840 4026531838 4026531839 4026532787 4026532789 nc -l 8989
  97966 4026531840 4026531838 4026531839 4026532787 4026532789 nc -l 9999
...
```
Notes: https://www.redhat.com/sysadmin/pid-namespace <br>
## Setting a ns name ...
On the host
```
unshare --user --map-root-user --uts --pid --fork --net --mount-proc bash --norc -c '(sleep 555 &) && (ps a &) && sleep 999' &
```

```
sudo touch /run/netns/new_namespace_128634
sudo mount -o bind /proc/128634/ns/net /run/netns/new_namespace_128634
```
````
$ sudo ip netns list
...
new_namespace_128634
...
```
Note: https://gist.github.com/cfra/39f4110366fa1ae9b1bddd1b47f586a3
