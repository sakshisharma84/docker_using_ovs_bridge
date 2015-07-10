docker_using_ovs_bridge
========================

This patch allows docker to use OVS bridge instead of its default linux bridge "docker0".
Now, whenever docker creates a container, it binds its veth* interface to this OVS bridge (instead of docker0).  

Pre-requisites
===============

1. Stopping Docker and removing docker0
-----------------------------------------
$ sudo service docker stop  
$ sudo ip link set dev docker0 down  
$ sudo brctl delbr docker0  
$ sudo iptables -t nat -F POSTROUTING

2. Make a new OVS bridge
--------------------------
$ sudo ovs-vsctl add-br ovs_bridge  
$ sudo ovs-vsctl set bridge ovs_bridge protocols=OpenFlow10  
$ sudo ip addr add 192.168.2.0/16 dev ovs_bridge  
$ sudo ip link set dev ovs_bridge up  
$ sudo ovs-ofctl show ovs_bridge

3. Link docker to the new bridge and restart docker
------------------------------------------------------
$ echo 'DOCKER_OPTS="-b=ovs_bridge"' >> /etc/default/docker  
$ sudo service docker start  

Confirming new outgoing NAT masquerade is set up  
$ sudo iptables -t nat -L -n  

4. Check whether docker is using your ovs bridge  
Run $ps -eaf | grep docker   
(docker -b ovs_bridge)  

5. Create a container and check if its interfaces are bound to the OVS bridge  
Use this command : $ sudo ovs-vsctl show







