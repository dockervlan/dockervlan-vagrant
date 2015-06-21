# dockervlan-vagrant
Quickly checkout dockervlan using Vagrant/Virtualbox.

## Getting started
1) Install dependencies

* Virtualbox (Tested with 4.3.10_Ubuntu r93012)
* Vagrant (Tested with 1.7.2)


2) Clone rancherio/os-vagrant https://github.com/rancherio/os-vagrant.git

```
git clone https://github.com/vlandocker/vlandocker-vagrant.git
cd vlandocker-vagrant
```

3) Download the latest Docker experimental build

```
wget -q https://github.com/dockervlan/dockervlan-vagrant/releases/download/latest/docker-1.8.0-dev -O docker-1.8.0-dev
```

4) Up and Running

```
vagrant up
vagrant ssh
```

5) Login again to the vagrant
```
vagrant ssh
```

6) Create the docker network for VLAN with id 200
Note: `apt-get install vlan` and `sudo modprobe 8021q` for Ubuntu 14.04 or equivalent in other Linux distros are required for VLAN functionality.
```
/opt/bin/docker-1.8.0-dev network create -d bridge --vlanid=100 --ifname=eth1 --bip=172.16.1.1/16 --fixed-cidr=172.16.1.0/24 vlan100
```

7) Confirm that the br200 and eth1.200 has been created
```
ifconfig -a | less
```

8) Create two docker containers
```
/opt/bin/docker-1.8.0-dev run -tid --net vlan100 busybox
/opt/bin/docker-1.8.0-dev run -tid --net vlan100 busybox
```

9) Confirm that the containers exist and view the underlying IPs
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
8b13eea86028        ubuntu              "/bin/bash"         13 seconds ago      Up 12 seconds                           dreamy_curie
492f0dd637f1        ubuntu              "/bin/bash"         20 minutes ago      Up 20 minutes                           tender_curie
```

10) Get the IP addresses of the two containers
```
[rancher@rancher ~]$ /opt/bin/docker-1.8.0-dev inspect 492f0dd637f1  | grep IP
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "IPAddress": "20.0.1.65",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "LinkLocalIPv6Address": "",
        "LinkLocalIPv6PrefixLen": 0,
        "SecondaryIPAddresses": null,
        "SecondaryIPv6Addresses": null
```
11) Attach one of the docker containers and ping to the other. This setup demonstrates the communication between two container within the same host.

12) Create new container with static address and ping.

```
/opt/bin/docker-1.8.0-dev run -tid --net vlan100:172.16.1.22 busybox
ping 172.16.1.22
```

13) Create new new network without a CIDR

```
/opt/bin/docker-1.8.0-dev network create -d bridge --vlanid=101 --ifname=eth1 vlan101
```

14) Create new container and ensure it can't talk with static IP from previous step.

```
/opt/bin/docker-1.8.0-dev run -ti --net vlan101 busybox
```


## Multiple rancheros instances.

1) It is possible to have multiple rancher-Os instanses and to demonstrate the interconnectivity between containers on different hosts. First step is to modify the Vagrantfile under the os-vagrant directory modifying number_of_nodes from 1 to the number of virtualboxes you want to be created.  In our example 2

```
..

$number_of_nodes = 2
..
```

2) Up and Running

```
vagrant up --parallel
vagrant ssh
```

3) Host 1 - Create Network

```
/opt/bin/docker-1.8.0-dev network create -d bridge --vlanid=101 --ifname=eth1 --bip=172.16.1.1/16 --fixed-cidr=172.16.1.0/24 vlan101
```

4) Host 1 - Create Container with Static IP

```
/opt/bin/docker-1.8.0-dev run -tid --net vlan101:172.16.1.21 busybox
```

5) Host 2 - Create Network

```
/opt/bin/docker-1.8.0-dev network create -d bridge --vlanid=101 --ifname=eth1 --bip=172.16.1.2/16 --fixed-cidr=172.16.1.0/24 vlan101
```

6) Host 2 - Create Container with Static IP

```
/opt/bin/docker-1.8.0-dev run -tid --net vlan101:172.16.1.22 busybox
```
