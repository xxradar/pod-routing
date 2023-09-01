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
unshare --user --map-root-user --uts --pid --fork --net --mount-proc sleep 666666
```

In **terminal 2**: <br>
Find the PID for the `-bash` process in the newly created namespaces.<br>
This is similar as the `pause` container in kubernetes.
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
```
$ sudo ip netns list
...
new_namespace_128634
...
```
Note: https://gist.github.com/cfra/39f4110366fa1ae9b1bddd1b47f586a3
