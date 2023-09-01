## Additional
In **terminal 1**:<br>
This will create shell in newly created Linux namespaces.
```
unshare --user --map-root-user --uts --pid --fork --net --mount-proc
```
Let's run a simple command so we can easily identify processes and namespaces.
```
sleep 8888 &
```
or alternatively
```
unshare --user --map-root-user --uts --pid --fork --net --mount-proc sleep 666666 &
```

In **terminal 2**: <br>
Find the PID for the `-bash` process in the newly created namespaces.<br>
This is similar as the `pause` container in kubernetes.
```
lsns | grep "sleep 666666"
...
4026533025 pid         1 139809 ubuntu sleep 666666
...
```
```
sudo nsenter -t 139809  -a
```
```
ps aux
```
```
root@newhostname:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   7236   580 pts/0    S    15:15   0:00 sleep 666666
root           2  0.0  0.0   9944  4928 pts/0    S    15:19   0:00 -bash
root           7  0.0  0.0  10620  3400 pts/0    R+   15:19   0:00 ps aux
```
```
sleep 777777 &
```
Exit or use a different terminal.
```
sudo ps -ax -n -o pid,netns,utsns,ipcns,mntns,pidns,cmd | grep "sleep"
...
 139809 4026533026 4026533024 4026531839 4026533023 4026533025 sleep 666666
 143367 4026533026 4026533024 4026531839 4026533023 4026533025 sleep 777777
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
```
$ sudo ip netns list
...
new_namespace_128634
...
```
Note: https://gist.github.com/cfra/39f4110366fa1ae9b1bddd1b47f586a3
