# dockervlan-vagrant
Quickly checkout dockervlan using Vagrant/Virtualbox.

## Getting started
1.) Install dependencies

* Virtualbox (Tested with 4.3.22)
* Vagrant (Tested with 1.7.2)

2.) Clone rancherio/os-vagrant https://github.com/rancherio/os-vagrant.git

```
git clone https://github.com/rancherio/os-vagrant.git
cd os-vagrant
```

3.) Up and Running

```
vagrant up
vagrant ssh
```

4.) We will modify rancher os to use our dockervlan docker executable. Assuming you have succsefully created from https://github.com/dockervlan/docker the docker executable do the following inside the rancheros-01 ( assuming docker-1.8.0-dev is the created docker executable )

```
sudo mkdir /opt/bin
sudo cp docker-1.8.0-dev /opt/bin
sudo chmod +x /opt/bin/docker-1.8.0-dev
```

5) We will create a custom-docker.yml and put it inside the rancher.conf
```
sudo vi /var/lib/rancher.conf/custom-docker.yml

Addd the following in the file

docker:
  image: docker
  links:
  - network
  net: host
  pid: host
  ipc: host
  privileged: true
  restart: always
  volumes_from:
  - all-volumes
  labels:
    io.rancher.os.scope: system
  volumes:
  - "/var/lib/rancher/state/opt/bin/docker-1.8.0-dev:/usr/bin/docker"


```

6) We will enable the service as default and we will reboot the vagrant
```
sudo ros service enable /var/lib/rancher/conf/custom-docker.yml
sudo reboot
```

7) Login again to the vagrant
```
vagrant ssh
```

8) Create the docker network for VLAN with id 200
```
./docker-1.8.0-dev network create -d bridge --vlanid=200 --ifname=eth1 --bip=20.0.1.1/16 --fixed-cidr=20.0.1.64/26 br200
```

9) Confirm that the br200 and eth1.200 has been created
```
ifconfig -a | less
```

10) Create two docker containers
```
./docker-1.8.0-dev run -ti --net br200 ubuntu
./docker-1.8.0-dev run -ti --net br200 ubuntu
```

11) Confirm that the containers exist and view the underlying IPs
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
8b13eea86028        ubuntu              "/bin/bash"         13 seconds ago      Up 12 seconds                           dreamy_curie        
492f0dd637f1        ubuntu              "/bin/bash"         20 minutes ago      Up 20 minutes                           tender_curie        
```

12) Get the IP addresses of the two containers
```
[rancher@rancher ~]$ docker inspect 492f0dd637f1  | grep IP
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
13) Attach one of the docker containers and ping to the other
