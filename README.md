# openstack

## Stand alone installation on 1 VM with devstack
* https://www.medo64.com/2021/04/devstack-on-virtualbox-ubuntu-20-04/
```
Terminal 

echo "$USER ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/$USER

sudo apt update
sudo apt dist-upgrade --yes
shutdown -r now

sudo useradd -s /bin/bash -d /opt/stack -m stack
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
sudo su - stack

tee  configure_openstack.sh<<EOF
#!/bin/bash

IP=achanger
ADMIN_PASSWORD=achanger
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD


STACK_BRANCH=stable/wallaby
git clone https://github.com/openstack-dev/devstack.git -b $STACK_BRANCH devstack/
cd devstack
cp samples/local.conf .
sed -i "s/#HOST_IP=w.x.y.z/HOST_IP=${IP}/" local.conf

sed -i "s/ADMIN_PASSWORD=nomoresecret/ADMIN_PASSWORD=${ADMIN_PASSWORD}/" local.conf
sed -i "s/DATABASE_PASSWORD=.*/DATABASE_PASSWORD=${DATABASE_PASSWORD}/" local.conf
sed -i "s/RABBIT_PASSWORD=.*/RABBIT_PASSWORD=${RABBIT_PASSWORD}/" local.conf
sed -i "s/SERVICE_PASSWORD=.*/SERVICE_PASSWORD=${SERVICE_PASSWORD}/" local.conf

echo "#Enable heat services" >> local.conf
echo "enable_service h-eng h-api h-api-cfn h-api-cw" >> local.conf
echo "#Enable heat plugin" >> local.conf
echo "enable_plugin heat https://git.openstack.org/openstack/heat $STACK_BRANCH" >> local.conf
EOF

```

Get informations
* http://192.168.0.159/dashboard/admin/info/ 


* https://www.journaldev.com/30037/install-openstack-ubuntu-devstack

* Fix for ubuntu 20.04 LTS : https://opendev.org/openstack/devstack/commit/9b0364f20ee6e076f5af14d28dd189713301280e

## Multi node installations
* https://raymii.org/s/tutorials/Openstack_Glance_Image_Download.html
* https://docs.openstack.org/glance/latest/admin/manage-images.html



Download Image:

```bash
glance image-list
glance image-download --file ./example-test.img 0a[...]5dd
glance image-download --file $FILENAME $UUID
```

## Courses

* https://github.com/openstack
* https://github.com/openstack/networking-vpp

## Get a token
```
curl  -X POST -H "Content-Type: application/json"   -d '{ "auth": { "identity": { "methods": ["password"], "password": { "user": { "name": "USERID_CHANGE_IT", "domain": { "id": "default" }, "password": "MYPASSWORD_CHANGE_IT" }  } }, "scope": { "project": { "name": "admin", "domain": { "id": "default" } } } } }' -i "http://controller:5000/v3/auth/tokens"



```


## Get images
* https://cloud-images.ubuntu.com/
* http://cloud.centos.org/centos/

## Tools to build images
* https://www.packer.io/ (hashicorp)





## Openstack CLI
* https://docs.openstack.org/python-openstackclient/latest/

### Create a VM
```
cd devstack
source openrc admin admin

vagrant@controller:~/devstack$ openstack image list
+--------------------------------------+---------------------------------+--------+
| ID                                   | Name                            | Status |
+--------------------------------------+---------------------------------+--------+
| ac11a84e-373d-4297-a319-d83d86008c65 | Fedora-Cloud-Base-25-1.3.x86_64 | active |
| 2abd2f03-f088-4f64-b97f-fc71087b6d85 | cirros-0.3.5-x86_64-disk        | active |
+--------------------------------------+---------------------------------+--------+

vagrant@controller:~/devstack$ openstack network list
+--------------------------------------+---------+----------------------------------------------------------------------------+
| ID                                   | Name    | Subnets                                                                    |
+--------------------------------------+---------+----------------------------------------------------------------------------+
| 11741fda-b0a3-49c4-9171-89ce451b17c4 | public  | 0ba83d7b-7fdc-4c6d-a470-fba072bfc280, 41fceb65-4009-4920-a19a-983169b5dbe6 |
| e5ab4a07-4f40-41a2-be23-48f8b11f4630 | private | 46363502-58f6-4a43-8b05-b8aab9cbe8a5, e5b354ae-cddf-4b2b-bafb-bdbee30eb9da |
+--------------------------------------+---------+----------------------------------------------------------------------------+

vagrant@controller:~/devstack$ openstack flavor list
+----+-----------+-------+------+-----------+-------+-----------+
| ID | Name      |   RAM | Disk | Ephemeral | VCPUs | Is Public |
+----+-----------+-------+------+-----------+-------+-----------+
| 1  | m1.tiny   |   512 |    1 |         0 |     1 | True      |
| 2  | m1.small  |  2048 |   20 |         0 |     1 | True      |
| 3  | m1.medium |  4096 |   40 |         0 |     2 | True      |
| 4  | m1.large  |  8192 |   80 |         0 |     4 | True      |
| 5  | m1.xlarge | 16384 |  160 |         0 |     8 | True      |
| c1 | cirros256 |   256 |    0 |         0 |     1 | True      |
| d1 | ds512M    |   512 |    5 |         0 |     1 | True      |
| d2 | ds1G      |  1024 |   10 |         0 |     1 | True      |
| d3 | ds2G      |  2048 |   10 |         0 |     2 | True      |
| d4 | ds4G      |  4096 |   20 |         0 |     4 | True      |
+----+-----------+-------+------+-----------+-------+-----------+


openstack server create --image cirros-0.3.5-x86_64-disk --flavor 1 --nic net-id=11741fda-b0a3-49c4-9171-89ce451b17c4 my_first_vm
+-------------------------------------+-----------------------------------------------------------------+
| Field                               | Value                                                           |
+-------------------------------------+-----------------------------------------------------------------+
| OS-DCF:diskConfig                   | MANUAL                                                          |
| OS-EXT-AZ:availability_zone         |                                                                 |
| OS-EXT-SRV-ATTR:host                | None                                                            |
| OS-EXT-SRV-ATTR:hypervisor_hostname | None                                                            |
| OS-EXT-SRV-ATTR:instance_name       |                                                                 |
| OS-EXT-STS:power_state              | NOSTATE                                                         |
| OS-EXT-STS:task_state               | scheduling                                                      |
| OS-EXT-STS:vm_state                 | building                                                        |
| OS-SRV-USG:launched_at              | None                                                            |
| OS-SRV-USG:terminated_at            | None                                                            |
| accessIPv4                          |                                                                 |
| accessIPv6                          |                                                                 |
| addresses                           |                                                                 |
| adminPass                           | 3RjDRiBdo8m9                                                    |
| config_drive                        |                                                                 |
| created                             | 2022-04-04T08:44:50Z                                            |
| flavor                              | m1.tiny (1)                                                     |
| hostId                              |                                                                 |
| id                                  | 8bcb2aa5-00a0-4ce9-ac50-30c250fe9d9c                            |
| image                               | cirros-0.3.5-x86_64-disk (2abd2f03-f088-4f64-b97f-fc71087b6d85) |
| key_name                            | None                                                            |
| name                                | my_first_vm                                                     |
| progress                            | 0                                                               |
| project_id                          | 200d842136a249ff9778d3bb2d9e9202                                |
| properties                          |                                                                 |
| security_groups                     | name='default'                                                  |
| status                              | BUILD                                                           |
| updated                             | 2022-04-04T08:44:49Z                                            |
| user_id                             | bf7206456b88476997d567f3d57c808f                                |
| volumes_attached                    |                                                                 |
+-------------------------------------+-----------------------------------------------------------------+


vagrant@controller:~/devstack$ openstack server list
+--------------------------------------+-------------+--------+--------------------------------+--------------------------+---------+
| ID                                   | Name        | Status | Networks                       | Image                    | Flavor  |
+--------------------------------------+-------------+--------+--------------------------------+--------------------------+---------+
| 8bcb2aa5-00a0-4ce9-ac50-30c250fe9d9c | my_first_vm | ACTIVE | public=2001:db8::b, 172.24.4.8 | cirros-0.3.5-x86_64-disk | m1.tiny |
+--------------------------------------+-------------+--------+--------------------------------+--------------------------+---------+


vagrant@controller:~/devstack$ openstack server show my_first_vm
+-------------------------------------+-----------------------------------------------------------------+
| Field                               | Value                                                           |
+-------------------------------------+-----------------------------------------------------------------+
| OS-DCF:diskConfig                   | MANUAL                                                          |
| OS-EXT-AZ:availability_zone         | nova                                                            |
| OS-EXT-SRV-ATTR:host                | controller                                                      |
| OS-EXT-SRV-ATTR:hypervisor_hostname | controller                                                      |
| OS-EXT-SRV-ATTR:instance_name       | instance-00000005                                               |
| OS-EXT-STS:power_state              | Running                                                         |
| OS-EXT-STS:task_state               | None                                                            |
| OS-EXT-STS:vm_state                 | active                                                          |
| OS-SRV-USG:launched_at              | 2022-04-04T08:44:56.000000                                      |
| OS-SRV-USG:terminated_at            | None                                                            |
| accessIPv4                          |                                                                 |
| accessIPv6                          |                                                                 |
| addresses                           | public=2001:db8::b, 172.24.4.8                                  |
| config_drive                        |                                                                 |
| created                             | 2022-04-04T08:44:49Z                                            |
| flavor                              | m1.tiny (1)                                                     |
| hostId                              | 7043d88172f23d951e27a1d5896a418c390581ea426bf3636023a697        |
| id                                  | 8bcb2aa5-00a0-4ce9-ac50-30c250fe9d9c                            |
| image                               | cirros-0.3.5-x86_64-disk (2abd2f03-f088-4f64-b97f-fc71087b6d85) |
| key_name                            | None                                                            |
| name                                | my_first_vm                                                     |
| progress                            | 0                                                               |
| project_id                          | 200d842136a249ff9778d3bb2d9e9202                                |
| properties                          |                                                                 |
| security_groups                     | name='default'                                                  |
| status                              | ACTIVE                                                          |
| updated                             | 2022-04-04T08:44:56Z                                            |
| user_id                             | bf7206456b88476997d567f3d57c808f                                |
| volumes_attached                    |                                                                 |
+-------------------------------------+-----------------------------------------------------------------+
```

## Troubleshooting
```
systemctl list-units devstack@c-*
UNIT                   LOAD   ACTIVE SUB     DESCRIPTION
devstack@c-api.service loaded active running Devstack devstack@c-api.service
devstack@c-sch.service loaded active running Devstack devstack@c-sch.service
devstack@c-vol.service loaded active running Devstack devstack@c-vol.service


# Check cinder 
sudo systemctl restart devstack@c-*

# there are errors : Apr 04 09:08:30 controller cinder-volume[6775]: ERROR cinder.volume.manager Stderr: u'  Volume group "stack-volumes-lvmdriver-1" not found\n  Cannot process volume group stack-volumes-lvmdriver-1\n'

vagrant@controller:~/devstack$ sudo journalctl -f --unit devstack@c-vol
-- Logs begin at Mon 2022-04-04 07:36:23 UTC. --
Apr 04 09:08:30 controller cinder-volume[6775]: ERROR cinder.volume.manager Stdout: u''
Apr 04 09:08:30 controller cinder-volume[6775]: ERROR cinder.volume.manager Stderr: u'  Volume group "stack-volumes-lvmdriver-1" not found\n  Cannot process volume group stack-volumes-lvmdriver-1\n'
Apr 04 09:08:30 controller cinder-volume[6775]: ERROR cinder.volume.manager
Apr 04 09:08:30 controller cinder-volume[6775]: DEBUG cinder.service [None req-7d35fd55-a5a6-4f9a-a150-9b4efc6a365f None None] Creating RPC server for service cinder-volume {{(pid=6810) start /opt/stack/cinder/cinder/service.py:219}}
Apr 04 09:08:30 controller cinder-volume[6775]: DEBUG oslo_db.sqlalchemy.engines [None req-7d35fd55-a5a6-4f9a-a150-9b4efc6a365f None None] MySQL server mode set to STRICT_TRANS_TABLES,STRICT_ALL_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,TRADITIONAL,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION {{(pid=6810) _check_effective_sql_mode /usr/local/lib/python2.7/dist-packages/oslo_db/sqlalchemy/engines.py:285}}
Apr 04 09:08:30 controller cinder-volume[6775]: DEBUG cinder.service [None req-7d35fd55-a5a6-4f9a-a150-9b4efc6a365f None None] Pinning object versions for RPC server serializer to 1.27 {{(pid=6810) start /opt/stack/cinder/cinder/service.py:226}}
Apr 04 09:08:30 controller cinder-volume[6775]: INFO cinder.volume.manager [None req-7d35fd55-a5a6-4f9a-a150-9b4efc6a365f None None] Initializing RPC dependent components of volume driver LVMVolumeDriver (3.0.0)
Apr 04 09:08:30 controller cinder-volume[6775]: ERROR cinder.utils [None req-7d35fd55-a5a6-4f9a-a150-9b4efc6a365f None None] Volume driver LVMVolumeDriver not initialized
Apr 04 09:08:30 controller cinder-volume[6775]: ERROR cinder.volume.manager [None req-7d35fd55-a5a6-4f9a-a150-9b4efc6a365f None None] Cannot complete RPC initialization because driver isn't initialized properly.: DriverNotInitialized: Volume driver not ready.
Apr 04 09:08:34 controller cinder-volume[6775]: WARNING cinder.volume.manager [None req-96ee484f-e21d-43e2-bc1a-75f5e77adbe9 None None] Update driver status failed: (config name lvmdriver-1) is uninitialized.
Apr 04 09:08:40 controller cinder-volume[6775]: ERROR cinder.service [-] Manager for service cinder-volume controller@lvmdriver-1 is reporting problems, not sending heartbeat. Service will appear "down".


```


# Hypervisor
```
 virsh
Welcome to virsh, the virtualization interactive terminal.

Type:  'help' for help with commands
       'quit' to quit

virsh # list
 Id    Name                           State
----------------------------------------------------
 1     instance-00000004              running

virsh # shutdown 1
Domain 1 is being shutdown

virsh # list
 Id    Name                           State
----------------------------------------------------

virsh # destroy 1
error: failed to get domain '1'
error: Domain not found: no domain with matching name '1'

```

## Automate
```
"automate.sh" 238L, 8573C
#!/bin/bash
# set -o xtrace

#########################################################################################
# Author: Naveen Joy
# Udemy Course Name: Fundamentals of the OpenStack Cloud with Hands-on Labs
# Course URL: https://www.udemy.com/deep-dive-the-openstack-cloud/?couponCode=NJ_STUDENT
#########################################################################################
#
# This script creates the following cloud resources using the OpenStack CLI Client:
#   A KeyPair
#   Two project networks (VXLAN) and their corresponding subnets
#   A router with interfaces added to both networks
#   A security-group with rules to allow ingress ssh and ICMP ping traffic
#   A host aggregate and exposes it as an Availability Zone named az2
#   Adds the compute host named "node1" to az2
#   Launches a VM ($VM1) on the default AZ (nova)
#   Launches another VM ($VM2) on az2
#   Creates a Volume
#   Attaches the Volume to $VM1
#
#  When the variable CLEANUP is set, it cleans up existing resources.
#  Use the variable CREATE to control the creation of resources
#########################################################################################

#########################################################################################
# Source Admin credentials
source /home/vagrant/devstack/openrc admin admin

# Variables that control the script action
CLEANUP=1  # When set, do a cleanup run to delete all old resources
CREATE=1   # When set, create cloud resources

# Set global variables to control the names of the resources we create
KEYPAIR=mykey
VM1=myvm1
VM2=myvm2
IMAGE="cirros-0.3.5-x86_64-disk"
FLAVOR="cirros256"
NET1=mynetwork1
NET2=mynetwork2
NET1_CIDR="10.1.1.0/24"
NET2_CIDR="10.2.2.0/24"
SUBNET1=mysubnet1
SUBNET2=mysubnet2
ROUTER=myrouter
SG=mysecgroup
COMPUTE_NODENAME=node1
AZ=az2
VOL_NAME=extra_space
VOL_SIZE=1  #1GB

# Creates a keypair
create_keypair(){
  openstack keypair create $KEYPAIR > ~/.ssh/${KEYPAIR} || { echo "failed to create keypair $KEYPAIR" >&2; exit 1; }
  echo "Created Keypair $KEYPAIR"
  chmod 600 ~/.ssh/${KEYPAIR}
}

# Deletes the keypair
delete_keypair(){
  openstack keypair delete $KEYPAIR || { echo "failed to delete keypair $KEYPAIR" >&2; }
  echo "Deleted Keypair $KEYPAIR"
  rm  ~/.ssh/${KEYPAIR}
}

# Create a network
create_network(){
  local netname=$1
  openstack network create $netname || { echo "failed to create network $netname" >&2; exit 1; }
  echo "Created network $netname"
}

# Delete a network
delete_network(){
  local netname=$1
  openstack network delete $netname || { echo "failed to delete network $netname" >&2; }
  echo "Deleted network $netname"
}

# Create a subnet
create_subnet(){
  local netname=$1
  local subnetname=$2
  local cidr=$3
  openstack subnet create --subnet-range $cidr --dns-nameserver 8.8.8.8 --dns-nameserver 8.8.4.4 \
                          --network $netname $subnetname || { echo "failed to create subnet $subnetname" >&2; exit 1; }
  echo "Created subnet $subnetname"
}

# Delete a subnet
delete_subnet(){
  local subnetname=$1
  openstack subnet delete $subnetname || { echo "failed to delete subnet $subnetname" >&2; }
  echo "Deleted subnet $subnetname"
}

# Create a router
create_router(){
  local routername=$1
  openstack router create $routername || { echo "failed to create router $routername" >&2; exit 1; }
  echo "Created router $routername"
}

# Delete a router
delete_router(){
  local routername=$1
  openstack router delete $routername || { echo "failed to delete router $routername" >&2; }
  echo "Deleted router $routername"
}

# Add a router interface to a subnet
add_router_interface(){
  local routername=$1
  local subnetname=$2
  openstack router add subnet $routername $subnetname || { echo "failed to add router intf to subnet $subnetname" >&2; exit 1; }
  echo "Added router $routername interface to subnet $subnetname"
}

# Remove a router interface from a subnet
remove_router_interface(){
  local routername=$1
  local subnetname=$2
  openstack router remove subnet $routername $subnetname || { echo "failed to remove router intf from subnet $subnetname" >&2; }
  echo "Deleted router $routername interface to subnet $subnetname"
}

# Create a security group and add rules to permit ingress ICMP and SSH traffic
create_security_group(){
  local sgname=$1
  openstack security group create $sgname || { echo "failed to create security group $sgname" >&2; exit 1; }
  echo "Created security group $sgname"
  openstack security group rule create --ingress --dst-port 22 ${sgname} || { echo "error creating SSH rule for SG:$sgname" >&2; exit 1; }
  openstack security group rule create --ingress --protocol icmp ${sgname} || { echo "error creating ICMP rule for SG:$sgname" >&2; exit 1; }
  echo "Created SSH and ICMP rules for security group $sgname"
}

# Delete a security group
delete_security_group(){
  local sgname=$1
  openstack security group delete $sgname || { echo "failed to delete security group $sgname" >&2; }
  echo "Deleted security group $sgname"
}

# Create a Host Aggregate named az1 and expose a new Availability-Zone with name $AZ. Then add the compute node to this AZ
create_az(){
  openstack aggregate create --zone $AZ $AZ || { echo "failed to create Availability Zone $AZ" >&2; exit 1; }
  echo "Created the Availability Zone $AZ"
  openstack aggregate add host $AZ $COMPUTE_NODENAME || { echo "failed to add the host $COMPUTE_NODENAME to $AZ" >&2; exit 1; }
  echo "Added the compute node $COMPUTE_NODENAME to the Host Aggregate $AZ"
}

# Delete the Host Aggregate and AZ
delete_az(){
  openstack aggregate remove host $AZ $COMPUTE_NODENAME || { echo "failed to remove host $COMPUTE_NODENAME from $AZ" >&2; }
  echo "Removed the compute node $COMPUTE_NODENAME from the Host Aggregate $AZ"
  openstack aggregate delete $AZ || { echo "failed to delete the Availability Zone $AZ" >&2; }
  echo "Deleted the Availability Zone $AZ"
}

# Boot a VM, inject keypair, add a NIC to the network, set security-group and place it in an AZ
boot_vm(){
 local vm_name=$1
 local net_name=$2
 local az=$3
 openstack server create --image $IMAGE --flavor $FLAVOR --key-name $KEYPAIR --network $net_name --security-group $SG \
                         --availability-zone $az $vm_name || { echo "Failed to create the VM: $vm_name" >&2; }
 echo "Created VM $vm_name, attached a NIC on $net_name, set security-group $SG, injected ssh-key $KEYPAIR and placed it on AZ $az"
}

# Delete a VM
delete_vm(){
  local vm_name=$1
  openstack server delete $vm_name || { echo "failed to delete the instance $vm_name" >&2; }
  echo "Deleted the instance $vm_name"
}

# Create a Volume
create_volume(){
  openstack volume create --size $VOL_SIZE $VOL_NAME || { echo "failed to create volume $VOL_NAME,SIZE=$VOL_SIZE" >&2; exit 1; }
  echo "Created Volume $VOL_NAME size=$VOL_SIZE GB"
}

# Delete a Volume
delete_volume(){
  openstack volume delete $VOL_NAME || { echo "failed to delete volume $VOL_NAME" >&2; }
  echo "Deleted Volume $VOL_NAME"
}

# Adds the Volume to VM1
add_volume(){
  openstack server add volume $VM1 $VOL_NAME || { echo "failed to add volume $VOL_NAME to server $VM1" >&2; exit 1; }
  echo "Added Volume $VOL_NAME to server $VM1"
}

# Remove the Volume from VM1
remove_volume(){
  openstack server remove volume $VM1 $VOL_NAME || { echo "failed to remove volume $VOL_NAME from server $VM1" >&2; }
  echo "Remove Volume $VOL_NAME from server $VM1"
}

### Cleanup if we are asked to do so ####
if (( CLEANUP )); then
  echo "Cleaning up.."
  delete_keypair
  remove_volume
  delete_vm $VM1
  delete_vm $VM2
  delete_volume
  delete_security_group $SG
  remove_router_interface $ROUTER $SUBNET1
  remove_router_interface $ROUTER $SUBNET2
  delete_router $ROUTER
  delete_subnet $SUBNET1
  delete_subnet $SUBNET2
  delete_network $NET1
  delete_network $NET2
  delete_az
fi

### Create new resources, if we are asked to do so ###
if (( CREATE )); then
 echo "Creating virtual infrastructure..."
 create_keypair
 create_network $NET1
 create_network $NET2
 create_subnet $NET1 $SUBNET1 $NET1_CIDR
 create_subnet $NET2 $SUBNET2 $NET2_CIDR
 create_router $ROUTER
 add_router_interface $ROUTER $SUBNET1
 add_router_interface $ROUTER $SUBNET2
 create_security_group $SG
 create_az
 boot_vm $VM1 $NET1 nova  # Boot the first VM on NET1 and AZ named nova (default) (i.e. place VM1 on the controller)
 boot_vm $VM2 $NET2 $AZ  # Boot the second VM on NET2 and in AZ=az2 (i.e. place VM2 on the compute node)
 create_volume   # Allocate some storage space

```

## Networking

```
openstack console log show myvm1

source openrc admin admin
WARNING: setting legacy OS_TENANT_NAME to support cli tools.
vagrant@controller:~/devstack$ openstack server list
+--------------------------------------+-------------+---------+--------------------------------+--------------------------+-----------+
| ID                                   | Name        | Status  | Networks                       | Image                    | Flavor    |
+--------------------------------------+-------------+---------+--------------------------------+--------------------------+-----------+
| 674a59d4-5665-4a01-b2d5-8ca545a992c2 | myvm2       | ACTIVE  | mynetwork2=10.2.2.10           | cirros-0.3.5-x86_64-disk | cirros256 |
| b6acdc09-0b89-492a-b2fe-17d98b4100dc | myvm1       | ACTIVE  | mynetwork1=10.1.1.3            | cirros-0.3.5-x86_64-disk | cirros256 |
| 95bdba24-2f9a-4e59-92aa-db94be3f49e2 | my_first_vm | SHUTOFF | public=2001:db8::d, 172.24.4.9 | cirros-0.3.5-x86_64-disk | m1.tiny   |
+--------------------------------------+-------------+---------+--------------------------------+--------------------------+-----------+


myvm2 login: vagrant@controller:~/devstack$ openstack console log show myvm2

vagrant@controller:~/devstack$ openstack router list
+--------------------------------------+----------+--------+-------+-------------+-------+----------------------------------+
| ID                                   | Name     | Status | State | Distributed | HA    | Project                          |
+--------------------------------------+----------+--------+-------+-------------+-------+----------------------------------+
| df8f2668-1f74-41c4-a984-1c02ec679d18 | myrouter | ACTIVE | UP    | False       | False | 200d842136a249ff9778d3bb2d9e9202 |
| f4c4910a-4b39-4dd9-8928-77fefa6a3ec0 | router1  | ACTIVE | UP    | False       | False | 09a20d123b024904a0bea9a4d7b1cae1 |
+--------------------------------------+----------+--------+-------+-------------+-------+----------------------------------+

# Namespace list
vagrant@controller:~/devstack$ sudo ip netns ls
qrouter-df8f2668-1f74-41c4-a984-1c02ec679d18
qdhcp-5064de9b-536c-4d44-9187-a2ca125367a5
qdhcp-f011b7d1-0540-418d-ad0d-be35ff400c99
qrouter-f4c4910a-4b39-4dd9-8928-77fefa6a3ec0
qdhcp-e5ab4a07-4f40-41a2-be23-48f8b11f4630

# IP in a namespace
vagrant@controller:~/devstack$ sudo ip netns exec qrouter-df8f2668-1f74-41c4-a984-1c02ec679d18 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
36: qr-f7dea8e4-a6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default qlen 1
    link/ether fa:16:3e:f1:2b:5e brd ff:ff:ff:ff:ff:ff
    inet 10.1.1.1/24 brd 10.1.1.255 scope global qr-f7dea8e4-a6
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fef1:2b5e/64 scope link
       valid_lft forever preferred_lft forever
37: qr-bc731fbb-da: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default qlen 1
    link/ether fa:16:3e:c5:67:a3 brd ff:ff:ff:ff:ff:ff
    inet 10.2.2.1/24 brd 10.2.2.255 scope global qr-bc731fbb-da
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fec5:67a3/64 scope link
       valid_lft forever preferred_lft forever
       
# Server List      
vagrant@controller:~/devstack$ openstack server list
+--------------------------------------+-------------+---------+--------------------------------+--------------------------+-----------+
| ID                                   | Name        | Status  | Networks                       | Image                    | Flavor    |
+--------------------------------------+-------------+---------+--------------------------------+--------------------------+-----------+
| 674a59d4-5665-4a01-b2d5-8ca545a992c2 | myvm2       | ACTIVE  | mynetwork2=10.2.2.10           | cirros-0.3.5-x86_64-disk | cirros256 |
| b6acdc09-0b89-492a-b2fe-17d98b4100dc | myvm1       | ACTIVE  | mynetwork1=10.1.1.3            | cirros-0.3.5-x86_64-disk | cirros256 |
| 95bdba24-2f9a-4e59-92aa-db94be3f49e2 | my_first_vm | SHUTOFF | public=2001:db8::d, 172.24.4.9 | cirros-0.3.5-x86_64-disk | m1.tiny   |
+--------------------------------------+-------------+---------+--------------------------------+--------------------------+-----------+

# ---------------------------------------
# ssh connexion in the namespace
# ---------------------------------------
vagrant@controller:~/devstack$ #sudo ip netns exec qrouter-df8f2668-1f74-41c4-a984-1c02ec679d18  ssh -i ~/.ssh/mykey cirros@10.1.1.3
vagrant@controller:~/devstack$ sudo ip netns ls
qrouter-df8f2668-1f74-41c4-a984-1c02ec679d18
qdhcp-5064de9b-536c-4d44-9187-a2ca125367a5
qdhcp-f011b7d1-0540-418d-ad0d-be35ff400c99
qrouter-f4c4910a-4b39-4dd9-8928-77fefa6a3ec0
qdhcp-e5ab4a07-4f40-41a2-be23-48f8b11f4630

# ---------------------------------------
# ssh connexion in the namespace
# ---------------------------------------
vagrant@controller:~/devstack$ sudo ip netns exec qrouter-df8f2668-1f74-41c4-a984-1c02ec679d18  ssh -i ~/.ssh/mykey cirros@10.1.1.3
The authenticity of host '10.1.1.3 (10.1.1.3)' can't be established.
RSA key fingerprint is SHA256:a0G7TIr5FyrDPeXxHZ/LVg6UBSZv4Y0BOaCogs3PTDM.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.1.1.3' (RSA) to the list of known hosts.
$ ls
$ hostname
myvm1
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast qlen 1000
    link/ether fa:16:3e:6c:d1:df brd ff:ff:ff:ff:ff:ff
    inet 10.1.1.3/24 brd 10.1.1.255 scope global eth0
    inet6 fe80::f816:3eff:fe6c:d1df/64 scope link
       valid_lft forever preferred_lft forever
$ ping 10.2.2.10
PING 10.2.2.10 (10.2.2.10): 56 data bytes
64 bytes from 10.2.2.10: seq=0 ttl=63 time=6.836 ms
64 bytes from 10.2.2.10: seq=1 ttl=63 time=1.593 ms
^C
--- 10.2.2.10 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 1.593/4.214/6.836 ms
$ exit
Connection to 10.1.1.3 closed.


# ---------------------------------------
# ovs: Open vswitch
# ---------------------------------------
* https://docs.openvswitch.org/en/latest/

vagrant@controller:~/devstack$ sudo ovs-vsctl list-br
br-ex
br-int
br-tun


vagrant@controller:~/devstack$ sudo ovs-vsctl list-ports br-tun
patch-int
vxlan-c0a8210b


vagrant@controller:~/devstack$ sudo ovs-vsctl list-ports br-int
int-br-ex
patch-tun
qg-8565d8fb-aa
qr-5b9b859b-84
qr-bc731fbb-da
qr-e7a10b24-54
qr-f7dea8e4-a6
qvo90708252-0e
qvo9b46662b-13
tap21371c49-7f
tap83584174-f7
tapd44acacc-1a


vagrant@controller:~/devstack$ sudo ovs-vsctl list port qg-8565d8fb-aa
_uuid               : 33db17dc-4c0f-461e-9165-4a55096bc471
bond_active_slave   : []
bond_downdelay      : 0
bond_fake_iface     : false
bond_mode           : []
bond_updelay        : 0
external_ids        : {}
fake_bridge         : false
interfaces          : [31358b29-a3eb-4b22-a413-b35654494cb4]
lacp                : []
mac                 : []
name                : "qg-8565d8fb-aa"
other_config        : {net_uuid="11741fda-b0a3-49c4-9171-89ce451b17c4", network_type=flat, physical_network=public, tag="2"}
qos                 : []
rstp_statistics     : {}
rstp_status         : {}
statistics          : {}
status              : {}
tag                 : 2
trunks              : []
vlan_mode           : []


# ---------------------------------------
# Check the router from a vm
# ---------------------------------------
vagrant@controller:~/devstack$ sudo ip netns exec qrouter-df8f2668-1f74-41c4-a984-1c02ec679d18  ssh -i ~/.ssh/mykey cirros@10.1.1.3
$ netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.1.1.1        0.0.0.0         UG        0 0          0 eth0
10.1.1.0        0.0.0.0         255.255.255.0   U         0 0          0 eth0
169.254.169.254 10.1.1.1        255.255.255.255 UGH       0 0          0 eth0
$ ping 10.1.1.1
PING 10.1.1.1 (10.1.1.1): 56 data bytes
64 bytes from 10.1.1.1: seq=0 ttl=64 time=0.720 ms


64 bytes from 10.1.1.1: seq=1 ttl=64 time=0.511 ms
64 bytes from 10.1.1.1: seq=2 ttl=64 time=0.740 ms
^C
--- 10.1.1.1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.511/0.657/0.740 ms
```


## Create a cloud by command line

![image](https://user-images.githubusercontent.com/32338685/161752268-45c31ee9-2958-4da7-82d5-884a020946e8.png)

```
### Cleanup if we are asked to do so ####
if (( CLEANUP )); then
  echo "Cleaning up.."
  delete_keypair
  remove_volume
  delete_vm $VM1
  delete_vm $VM2
  delete_volume
  delete_security_group $SG
  remove_router_interface $ROUTER $SUBNET1
  remove_router_interface $ROUTER $SUBNET2
  delete_router $ROUTER
  delete_subnet $SUBNET1
  delete_subnet $SUBNET2
  delete_network $NET1
  delete_network $NET2
  delete_az
fi

### Create new resources, if we are asked to do so ###
if (( CREATE )); then
 echo "Creating virtual infrastructure..."
 create_keypair
 create_network $NET1
 create_network $NET2
 create_subnet $NET1 $SUBNET1 $NET1_CIDR
 create_subnet $NET2 $SUBNET2 $NET2_CIDR
 create_router $ROUTER
 add_router_interface $ROUTER $SUBNET1
 add_router_interface $ROUTER $SUBNET2
 create_security_group $SG
 create_az 
 boot_vm $VM1 $NET1 nova  # Boot the first VM on NET1 and AZ named nova (default) (i.e. place VM1 on the controller)
 boot_vm $VM2 $NET2 $AZ  # Boot the second VM on NET2 and in AZ=az2 (i.e. place VM2 on the compute node)
 create_volume   # Allocate some storage space
 add_volume      # Attach the storage volume to $VM1
fi
```

```
# -------------------------------
# Create the keypair
# -------------------------------
vagrant@controller:~$ openstack keypair create mykey > ~/.ssh/mykey
vagrant@controller:~$ ls -lahtr ~/.ssh/
total 16K
-rw------- 1 vagrant vagrant  389 Dec  1  2017 authorized_keys
drwxr-xr-x 7 vagrant vagrant 4.0K Apr  5 08:40 ..
drwx------ 2 vagrant root    4.0K Apr  5 08:43 .
-rw-rw-r-- 1 vagrant vagrant 1.7K Apr  5 08:43 mykey


# -------------------------------
# Create the nertwork
# -------------------------------
vagrant@controller:~$ openstack network create mynetwork1
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2022-04-05T08:45:47Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | e83755ae-b262-4cb7-bf1a-d36815dbc148 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1450                                 |
| name                      | mynetwork1                           |
| port_security_enabled     | True                                 |
| project_id                | 200d842136a249ff9778d3bb2d9e9202     |
| provider:network_type     | vxlan                                |
| provider:physical_network | None                                 |
| provider:segmentation_id  | 94                                   |
| qos_policy_id             | None                                 |
| revision_number           | 2                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| updated_at                | 2022-04-05T08:45:47Z                 |
+---------------------------+--------------------------------------+


vagrant@controller:~$ openstack network create mynetwork2
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2022-04-05T08:46:02Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | f883cafc-3607-40eb-a6f4-1cb4fd010038 |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | False                                |
| is_vlan_transparent       | None                                 |
| mtu                       | 1450                                 |
| name                      | mynetwork2                           |
| port_security_enabled     | True                                 |
| project_id                | 200d842136a249ff9778d3bb2d9e9202     |
| provider:network_type     | vxlan                                |
| provider:physical_network | None                                 |
| provider:segmentation_id  | 15                                   |
| qos_policy_id             | None                                 |
| revision_number           | 2                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      |                                      |
| updated_at                | 2022-04-05T08:46:02Z                 |
+---------------------------+--------------------------------------+

vagrant@controller:~$ openstack network list
+--------------------------------------+------------+----------------------------------------------------------------------------+
| ID                                   | Name       | Subnets                                                                    |
+--------------------------------------+------------+----------------------------------------------------------------------------+
| 11741fda-b0a3-49c4-9171-89ce451b17c4 | public     | 0ba83d7b-7fdc-4c6d-a470-fba072bfc280, 41fceb65-4009-4920-a19a-983169b5dbe6 |
| e5ab4a07-4f40-41a2-be23-48f8b11f4630 | private    | 46363502-58f6-4a43-8b05-b8aab9cbe8a5, e5b354ae-cddf-4b2b-bafb-bdbee30eb9da |
| e83755ae-b262-4cb7-bf1a-d36815dbc148 | mynetwork1 |                                                                            |
| f883cafc-3607-40eb-a6f4-1cb4fd010038 | mynetwork2 |                                                                            |
+--------------------------------------+------------+----------------------------------------------------------------------------+

# -------------------------------
# Create the subnet
# -------------------------------
vagrant@controller:~$ openstack subnet list
+--------------------------------------+---------------------+--------------------------------------+---------------------+
| ID                                   | Name                | Network                              | Subnet              |
+--------------------------------------+---------------------+--------------------------------------+---------------------+
| 0ba83d7b-7fdc-4c6d-a470-fba072bfc280 | ipv6-public-subnet  | 11741fda-b0a3-49c4-9171-89ce451b17c4 | 2001:db8::/64       |
| 41fceb65-4009-4920-a19a-983169b5dbe6 | public-subnet       | 11741fda-b0a3-49c4-9171-89ce451b17c4 | 172.24.4.0/24       |
| 46363502-58f6-4a43-8b05-b8aab9cbe8a5 | private-subnet      | e5ab4a07-4f40-41a2-be23-48f8b11f4630 | 10.0.0.0/26         |
| e5b354ae-cddf-4b2b-bafb-bdbee30eb9da | ipv6-private-subnet | e5ab4a07-4f40-41a2-be23-48f8b11f4630 | fd4d:7949:115a::/64 |
+--------------------------------------+---------------------+--------------------------------------+---------------------+

vagrant@controller:~$ openstack subnet create --help
usage: openstack subnet create [-h] [-f {json,shell,table,value,yaml}]
                               [-c COLUMN] [--max-width <integer>]
                               [--fit-width] [--print-empty] [--noindent]
                               [--prefix PREFIX] [--project <project>]
                               [--project-domain <project-domain>]
                               [--subnet-pool <subnet-pool> | --use-default-subnet-pool]
                               [--prefix-length <prefix-length>]
                               [--subnet-range <subnet-range>]
                               [--dhcp | --no-dhcp] [--gateway <gateway>]
                               [--ip-version {4,6}]
                               [--ipv6-ra-mode {dhcpv6-stateful,dhcpv6-stateless,slaac}]
                               [--ipv6-address-mode {dhcpv6-stateful,dhcpv6-stateless,slaac}]
                               [--network-segment <network-segment>] --network
                               <network> [--description <description>]
                               [--allocation-pool start=<ip-address>,end=<ip-address>]
                               [--dns-nameserver <dns-nameserver>]
                               [--host-route destination=<subnet>,gateway=<ip-address>]
                               [--service-type <service-type>]
                               [--tag <tag> | --no-tag]
                               name
... 


vagrant@controller:~$ openstack network list
+--------------------------------------+------------+----------------------------------------------------------------------------+
| ID                                   | Name       | Subnets                                                                    |
+--------------------------------------+------------+----------------------------------------------------------------------------+
| 11741fda-b0a3-49c4-9171-89ce451b17c4 | public     | 0ba83d7b-7fdc-4c6d-a470-fba072bfc280, 41fceb65-4009-4920-a19a-983169b5dbe6 |
| e5ab4a07-4f40-41a2-be23-48f8b11f4630 | private    | 46363502-58f6-4a43-8b05-b8aab9cbe8a5, e5b354ae-cddf-4b2b-bafb-bdbee30eb9da |
| e83755ae-b262-4cb7-bf1a-d36815dbc148 | mynetwork1 |                                                                            |
| f883cafc-3607-40eb-a6f4-1cb4fd010038 | mynetwork2 |                                                                            |
+--------------------------------------+------------+----------------------------------------------------------------------------+


vagrant@controller:~$ openstack subnet create --subnet-range 10.1.1.0/24 --dns-nameserver 8.8.8.8 --network mynetwork1 mysubnet1
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| allocation_pools        | 10.1.1.2-10.1.1.254                  |
| cidr                    | 10.1.1.0/24                          |
| created_at              | 2022-04-05T08:56:04Z                 |
| description             |                                      |
| dns_nameservers         | 8.8.8.8                              |
| enable_dhcp             | True                                 |
| gateway_ip              | 10.1.1.1                             |
| host_routes             |                                      |
| id                      | 49b594ff-a997-4ce8-9af0-598e15128df8 |
| ip_version              | 4                                    |
| ipv6_address_mode       | None                                 |
| ipv6_ra_mode            | None                                 |
| name                    | mysubnet1                            |
| network_id              | e83755ae-b262-4cb7-bf1a-d36815dbc148 |
| project_id              | 200d842136a249ff9778d3bb2d9e9202     |
| revision_number         | 0                                    |
| segment_id              | None                                 |
| service_types           |                                      |
| subnetpool_id           | None                                 |
| tags                    |                                      |
| updated_at              | 2022-04-05T08:56:04Z                 |
| use_default_subnet_pool | None                                 |
+-------------------------+--------------------------------------+


vagrant@controller:~$ openstack subnet create --subnet-range 10.2.2.0/24 --dns-nameserver 8.8.8.8 --network mynetwork2 mysubnet2
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| allocation_pools        | 10.2.2.2-10.2.2.254                  |
| cidr                    | 10.2.2.0/24                          |
| created_at              | 2022-04-05T08:56:25Z                 |
| description             |                                      |
| dns_nameservers         | 8.8.8.8                              |
| enable_dhcp             | True                                 |
| gateway_ip              | 10.2.2.1                             |
| host_routes             |                                      |
| id                      | 8a403578-8fbc-48d6-a248-ac06865954da |
| ip_version              | 4                                    |
| ipv6_address_mode       | None                                 |
| ipv6_ra_mode            | None                                 |
| name                    | mysubnet2                            |
| network_id              | f883cafc-3607-40eb-a6f4-1cb4fd010038 |
| project_id              | 200d842136a249ff9778d3bb2d9e9202     |
| revision_number         | 0                                    |
| segment_id              | None                                 |
| service_types           |                                      |
| subnetpool_id           | None                                 |
| tags                    |                                      |
| updated_at              | 2022-04-05T08:56:25Z                 |
| use_default_subnet_pool | None                                 |
+-------------------------+--------------------------------------+


vagrant@controller:~$ openstack subnet list
+--------------------------------------+---------------------+--------------------------------------+---------------------+
| ID                                   | Name                | Network                              | Subnet              |
+--------------------------------------+---------------------+--------------------------------------+---------------------+
| 0ba83d7b-7fdc-4c6d-a470-fba072bfc280 | ipv6-public-subnet  | 11741fda-b0a3-49c4-9171-89ce451b17c4 | 2001:db8::/64       |
| 41fceb65-4009-4920-a19a-983169b5dbe6 | public-subnet       | 11741fda-b0a3-49c4-9171-89ce451b17c4 | 172.24.4.0/24       |
| 46363502-58f6-4a43-8b05-b8aab9cbe8a5 | private-subnet      | e5ab4a07-4f40-41a2-be23-48f8b11f4630 | 10.0.0.0/26         |
| 49b594ff-a997-4ce8-9af0-598e15128df8 | mysubnet1           | e83755ae-b262-4cb7-bf1a-d36815dbc148 | 10.1.1.0/24         |
| 8a403578-8fbc-48d6-a248-ac06865954da | mysubnet2           | f883cafc-3607-40eb-a6f4-1cb4fd010038 | 10.2.2.0/24         |
| e5b354ae-cddf-4b2b-bafb-bdbee30eb9da | ipv6-private-subnet | e5ab4a07-4f40-41a2-be23-48f8b11f4630 | fd4d:7949:115a::/64 |
+--------------------------------------+---------------------+--------------------------------------+---------------------+

# -------------------------------
# Create the router
# -------------------------------
vagrant@controller:~$ openstack router list
+--------------------------------------+---------+--------+-------+-------------+-------+----------------------------------+
| ID                                   | Name    | Status | State | Distributed | HA    | Project                          |
+--------------------------------------+---------+--------+-------+-------------+-------+----------------------------------+
| f4c4910a-4b39-4dd9-8928-77fefa6a3ec0 | router1 | ACTIVE | UP    | False       | False | 09a20d123b024904a0bea9a4d7b1cae1 |
+--------------------------------------+---------+--------+-------+-------------+-------+----------------------------------+


vagrant@controller:~$ openstack router create myrouter
+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| admin_state_up          | UP                                   |
| availability_zone_hints |                                      |
| availability_zones      |                                      |
| created_at              | 2022-04-05T09:01:41Z                 |
| description             |                                      |
| distributed             | False                                |
| external_gateway_info   | None                                 |
| flavor_id               | None                                 |
| ha                      | False                                |
| id                      | bd7ceb3a-49f4-418d-a9c5-88ccc1212df0 |
| name                    | myrouter                             |
| project_id              | 200d842136a249ff9778d3bb2d9e9202     |
| revision_number         | None                                 |
| routes                  |                                      |
| status                  | ACTIVE                               |
| tags                    |                                      |
| updated_at              | 2022-04-05T09:01:41Z                 |
+-------------------------+--------------------------------------+


vagrant@controller:~$ openstack router list
+--------------------------------------+----------+--------+-------+-------------+-------+----------------------------------+
| ID                                   | Name     | Status | State | Distributed | HA    | Project                          |
+--------------------------------------+----------+--------+-------+-------------+-------+----------------------------------+
| bd7ceb3a-49f4-418d-a9c5-88ccc1212df0 | myrouter | ACTIVE | UP    | False       | False | 200d842136a249ff9778d3bb2d9e9202 |
| f4c4910a-4b39-4dd9-8928-77fefa6a3ec0 | router1  | ACTIVE | UP    | False       | False | 09a20d123b024904a0bea9a4d7b1cae1 |
+--------------------------------------+----------+--------+-------+-------------+-------+----------------------------------+

vagrant@controller:~$ openstack router add subnet myrouter mysubnet1
vagrant@controller:~$ openstack router add subnet myrouter mysubnet2

# -------------------------------
# Create the security group
# -------------------------------
vagrant@controller:~$ openstack security group list
+--------------------------------------+---------+------------------------+----------------------------------+
| ID                                   | Name    | Description            | Project                          |
+--------------------------------------+---------+------------------------+----------------------------------+
| 1a0a86c3-6290-4624-a029-f553f963f752 | default | Default security group | 200d842136a249ff9778d3bb2d9e9202 |
| 6db70520-38e5-4bda-986e-b51d6acf3b58 | default | Default security group | 09a20d123b024904a0bea9a4d7b1cae1 |
| 97043a79-18b4-46b3-b857-d9e3ff78f4e0 | default | Default security group |                                  |
| dbab7bfd-51c6-4235-980d-81170fffc73c | default | Default security group | d817aae2059a4de9818c059995df570a |
+--------------------------------------+---------+------------------------+----------------------------------+
vagrant@controller:~$ openstack security group --help
Command "security" matches:
  security group create
  security group delete
  security group list
  security group rule create
  security group rule delete
  security group rule list
  security group rule show
  security group set
  security group show
  



vagrant@controller:~$ openstack security group show 1a0a86c3-6290-4624-a029-f553f963f752
+-----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field           | Value                                                                                                                                                                                                          |
+-----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| created_at      | 2017-12-04T21:00:03Z                                                                                                                                                                                           |
| description     | Default security group                                                                                                                                                                                         |
| id              | 1a0a86c3-6290-4624-a029-f553f963f752                                                                                                                                                                           |
| name            | default                                                                                                                                                                                                        |
| project_id      | 200d842136a249ff9778d3bb2d9e9202                                                                                                                                                                               |
| revision_number | 4                                                                                                                                                                                                              |
| rules           | created_at='2017-12-04T21:00:03Z', direction='egress', ethertype='IPv6', id='36304f7e-a652-4c50-8987-2d125fdaf366', updated_at='2017-12-04T21:00:03Z'                                                          |
|                 | created_at='2017-12-04T21:00:03Z', direction='ingress', ethertype='IPv6', id='57e7ecfa-0da4-40ef-ba15-af515bc67988', remote_group_id='1a0a86c3-6290-4624-a029-f553f963f752', updated_at='2017-12-04T21:00:03Z' |
|                 | created_at='2017-12-04T21:00:03Z', direction='ingress', ethertype='IPv4', id='9f0f0f4f-bbf7-421a-92b7-abaf9be98f0a', remote_group_id='1a0a86c3-6290-4624-a029-f553f963f752', updated_at='2017-12-04T21:00:03Z' |
|                 | created_at='2017-12-04T21:00:03Z', direction='egress', ethertype='IPv4', id='c9812167-7b2b-484e-90c6-58fce28e18bd', updated_at='2017-12-04T21:00:03Z'                                                          |
| updated_at      | 2017-12-04T21:00:03Z                                                                                                                                                                                           |
+-----------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


vagrant@controller:~$ openstack security group rules show 57e7ecfa-0da4-40ef-ba15-af515bc67988
openstack: 'security group rules show 57e7ecfa-0da4-40ef-ba15-af515bc67988' is not an openstack command. See 'openstack --help'.
Did you mean one of these?
  security group create
  security group delete
  security group list
  security group rule create
  security group rule delete
  security group rule list
  security group rule show
  security group set
  security group show
  secret container create
  secret container delete
  secret container get
  secret container list
  secret delete
  secret get
  secret list
  secret order create
  secret order delete
  secret order get
  secret order list
  secret store
  secret update


vagrant@controller:~$ openstack security group rule show 57e7ecfa-0da4-40ef-ba15-af515bc67988

+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2017-12-04T21:00:03Z                 |
| description       | None                                 |
| direction         | ingress                              |
| ether_type        | IPv6                                 |
| id                | 57e7ecfa-0da4-40ef-ba15-af515bc67988 |
| name              | None                                 |
| port_range_max    | None                                 |
| port_range_min    | None                                 |
| project_id        | 200d842136a249ff9778d3bb2d9e9202     |
| protocol          | None                                 |
| remote_group_id   | 1a0a86c3-6290-4624-a029-f553f963f752 |
| remote_ip_prefix  | None                                 |
| revision_number   | 0                                    |
| security_group_id | 1a0a86c3-6290-4624-a029-f553f963f752 |
| updated_at        | 2017-12-04T21:00:03Z                 |
+-------------------+--------------------------------------+
vagrant@controller:~$
vagrant@controller:~$ openstack security group create mysecgroup
+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field           | Value                                                                                                                                                 |
+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------+
| created_at      | 2022-04-05T09:10:32Z                                                                                                                                  |
| description     | mysecgroup                                                                                                                                            |
| id              | 8ad65813-4333-407a-8428-9a2657c76ed2                                                                                                                  |
| name            | mysecgroup                                                                                                                                            |
| project_id      | 200d842136a249ff9778d3bb2d9e9202                                                                                                                      |
| revision_number | 2                                                                                                                                                     |
| rules           | created_at='2022-04-05T09:10:32Z', direction='egress', ethertype='IPv4', id='226d3287-39aa-48ae-98f7-279d740573c9', updated_at='2022-04-05T09:10:32Z' |
|                 | created_at='2022-04-05T09:10:32Z', direction='egress', ethertype='IPv6', id='fbefa687-7be8-43ee-92fe-0491f6c8bc7b', updated_at='2022-04-05T09:10:32Z' |
| updated_at      | 2022-04-05T09:10:32Z                                                                                                                                  |
+-----------------+-------------------------------------------------------------------------------------------------------------------------------------------------------+


vagrant@controller:~$ openstack security group list
+--------------------------------------+------------+------------------------+----------------------------------+
| ID                                   | Name       | Description            | Project                          |
+--------------------------------------+------------+------------------------+----------------------------------+
| 1a0a86c3-6290-4624-a029-f553f963f752 | default    | Default security group | 200d842136a249ff9778d3bb2d9e9202 |
| 6db70520-38e5-4bda-986e-b51d6acf3b58 | default    | Default security group | 09a20d123b024904a0bea9a4d7b1cae1 |
| 8ad65813-4333-407a-8428-9a2657c76ed2 | mysecgroup | mysecgroup             | 200d842136a249ff9778d3bb2d9e9202 |
| 97043a79-18b4-46b3-b857-d9e3ff78f4e0 | default    | Default security group |                                  |
| dbab7bfd-51c6-4235-980d-81170fffc73c | default    | Default security group | d817aae2059a4de9818c059995df570a |
+--------------------------------------+------------+------------------------+----------------------------------+


vagrant@controller:~$ openstack security group rule create --ingress --dst-port 22 mysecgroup
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2022-04-05T09:11:24Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | c937f438-466a-474e-b5b5-686dc0e6549e |
| name              | None                                 |
| port_range_max    | 22                                   |
| port_range_min    | 22                                   |
| project_id        | 200d842136a249ff9778d3bb2d9e9202     |
| protocol          | tcp                                  |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 0                                    |
| security_group_id | 8ad65813-4333-407a-8428-9a2657c76ed2 |
| updated_at        | 2022-04-05T09:11:24Z                 |
+-------------------+--------------------------------------+


vagrant@controller:~$ openstack security group rule create --ingress --protocol icmp mysecgroup
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2022-04-05T09:12:28Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | 9e44f96d-0db6-4985-9fef-98d0806989af |
| name              | None                                 |
| port_range_max    | None                                 |
| port_range_min    | None                                 |
| project_id        | 200d842136a249ff9778d3bb2d9e9202     |
| protocol          | icmp                                 |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 0                                    |
| security_group_id | 8ad65813-4333-407a-8428-9a2657c76ed2 |
| updated_at        | 2022-04-05T09:12:28Z                 |
+-------------------+--------------------------------------+


vagrant@controller:~$ openstack security group show mysecgroup
+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field           | Value                                                                                                                                                                                                                                          |
+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| created_at      | 2022-04-05T09:10:32Z                                                                                                                                                                                                                           |
| description     | mysecgroup                                                                                                                                                                                                                                     |
| id              | 8ad65813-4333-407a-8428-9a2657c76ed2                                                                                                                                                                                                           |
| name            | mysecgroup                                                                                                                                                                                                                                     |
| project_id      | 200d842136a249ff9778d3bb2d9e9202                                                                                                                                                                                                               |
| revision_number | 4                                                                                                                                                                                                                                              |
| rules           | created_at='2022-04-05T09:10:32Z', direction='egress', ethertype='IPv4', id='226d3287-39aa-48ae-98f7-279d740573c9', updated_at='2022-04-05T09:10:32Z'                                                                                          |
|                 | created_at='2022-04-05T09:12:28Z', direction='ingress', ethertype='IPv4', id='9e44f96d-0db6-4985-9fef-98d0806989af', protocol='icmp', remote_ip_prefix='0.0.0.0/0', updated_at='2022-04-05T09:12:28Z'                                          |
|                 | created_at='2022-04-05T09:11:24Z', direction='ingress', ethertype='IPv4', id='c937f438-466a-474e-b5b5-686dc0e6549e', port_range_max='22', port_range_min='22', protocol='tcp', remote_ip_prefix='0.0.0.0/0', updated_at='2022-04-05T09:11:24Z' |
|                 | created_at='2022-04-05T09:10:32Z', direction='egress', ethertype='IPv6', id='fbefa687-7be8-43ee-92fe-0491f6c8bc7b', updated_at='2022-04-05T09:10:32Z'                                                                                          |
| updated_at      | 2022-04-05T09:12:28Z                                                                                                                                                                                                                           |
+-----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+



# -------------------------------
# Create the AZ
# -------------------------------
vagrant@controller:~$ openstack aggregate create --zone az2 az2
+-------------------+----------------------------+
| Field             | Value                      |
+-------------------+----------------------------+
| availability_zone | az2                        |
| created_at        | 2022-04-05T09:21:02.209826 |
| deleted           | False                      |
| deleted_at        | None                       |
| id                | 3                          |
| name              | az2                        |
| updated_at        | None                       |
+-------------------+----------------------------+
vagrant@controller:~$ openstack aggregate list
+----+------+-------------------+
| ID | Name | Availability Zone |
+----+------+-------------------+
|  3 | az2  | az2               |
+----+------+-------------------+

vagrant@controller:~$ openstack aggregate add host az2 node1
+-------------------+--------------------------------+
| Field             | Value                          |
+-------------------+--------------------------------+
| availability_zone | az2                            |
| created_at        | 2022-04-05T09:21:02.000000     |
| deleted           | False                          |
| deleted_at        | None                           |
| hosts             | [u'node1']                     |
| id                | 3                              |
| metadata          | {u'availability_zone': u'az2'} |
| name              | az2                            |
| updated_at        | None                           |
+-------------------+--------------------------------+
vagrant@controller:~$ openstack aggregate show az2
+-------------------+----------------------------+
| Field             | Value                      |
+-------------------+----------------------------+
| availability_zone | az2                        |
| created_at        | 2022-04-05T09:21:02.000000 |
| deleted           | False                      |
| deleted_at        | None                       |
| hosts             | [u'node1']                 |
| id                | 3                          |
| name              | az2                        |
| properties        |                            |
| updated_at        | None                       |
+-------------------+----------------------------+
vagrant@controller:~$ openstack aggregate list
+----+------+-------------------+
| ID | Name | Availability Zone |
+----+------+-------------------+
|  3 | az2  | az2               |
+----+------+-------------------+



# -------------------------------
# Create the VMs
# -------------------------------


vagrant@controller:~$ openstack image list
+--------------------------------------+---------------------------------+--------+
| ID                                   | Name                            | Status |
+--------------------------------------+---------------------------------+--------+
| ac11a84e-373d-4297-a319-d83d86008c65 | Fedora-Cloud-Base-25-1.3.x86_64 | active |
| 2abd2f03-f088-4f64-b97f-fc71087b6d85 | cirros-0.3.5-x86_64-disk        | active |
+--------------------------------------+---------------------------------+--------+
vagrant@controller:~$ openstack flavor list
+----+-----------+-------+------+-----------+-------+-----------+
| ID | Name      |   RAM | Disk | Ephemeral | VCPUs | Is Public |
+----+-----------+-------+------+-----------+-------+-----------+
| 1  | m1.tiny   |   512 |    1 |         0 |     1 | True      |
| 2  | m1.small  |  2048 |   20 |         0 |     1 | True      |
| 3  | m1.medium |  4096 |   40 |         0 |     2 | True      |
| 4  | m1.large  |  8192 |   80 |         0 |     4 | True      |
| 5  | m1.xlarge | 16384 |  160 |         0 |     8 | True      |
| c1 | cirros256 |   256 |    0 |         0 |     1 | True      |
| d1 | ds512M    |   512 |    5 |         0 |     1 | True      |
| d2 | ds1G      |  1024 |   10 |         0 |     1 | True      |
| d3 | ds2G      |  2048 |   10 |         0 |     2 | True      |
| d4 | ds4G      |  4096 |   20 |         0 |     4 | True      |
+----+-----------+-------+------+-----------+-------+-----------+


vagrant@controller:~$ openstack keypair list
+-------+-------------------------------------------------+
| Name  | Fingerprint                                     |
+-------+-------------------------------------------------+
| mykey | 95:c6:0b:42:c6:49:a9:ba:23:a2:26:86:2f:16:49:66 |
+-------+-------------------------------------------------+


vagrant@controller:~$ openstack network list
+--------------------------------------+------------+----------------------------------------------------------------------------+
| ID                                   | Name       | Subnets                                                                    |
+--------------------------------------+------------+----------------------------------------------------------------------------+
| 11741fda-b0a3-49c4-9171-89ce451b17c4 | public     | 0ba83d7b-7fdc-4c6d-a470-fba072bfc280, 41fceb65-4009-4920-a19a-983169b5dbe6 |
| e5ab4a07-4f40-41a2-be23-48f8b11f4630 | private    | 46363502-58f6-4a43-8b05-b8aab9cbe8a5, e5b354ae-cddf-4b2b-bafb-bdbee30eb9da |
| e83755ae-b262-4cb7-bf1a-d36815dbc148 | mynetwork1 | 49b594ff-a997-4ce8-9af0-598e15128df8                                       |
| f883cafc-3607-40eb-a6f4-1cb4fd010038 | mynetwork2 | 8a403578-8fbc-48d6-a248-ac06865954da                                       |
+--------------------------------------+------------+----------------------------------------------------------------------------+


vagrant@controller:~$ openstack security group list
+--------------------------------------+------------+------------------------+----------------------------------+
| ID                                   | Name       | Description            | Project                          |
+--------------------------------------+------------+------------------------+----------------------------------+
| 1a0a86c3-6290-4624-a029-f553f963f752 | default    | Default security group | 200d842136a249ff9778d3bb2d9e9202 |
| 6db70520-38e5-4bda-986e-b51d6acf3b58 | default    | Default security group | 09a20d123b024904a0bea9a4d7b1cae1 |
| 8ad65813-4333-407a-8428-9a2657c76ed2 | mysecgroup | mysecgroup             | 200d842136a249ff9778d3bb2d9e9202 |
| 97043a79-18b4-46b3-b857-d9e3ff78f4e0 | default    | Default security group |                                  |
| dbab7bfd-51c6-4235-980d-81170fffc73c | default    | Default security group | d817aae2059a4de9818c059995df570a |
+--------------------------------------+------------+------------------------+----------------------------------+


vagrant@controller:~$ openstack aggregate create --zone az2 az2
+-------------------+----------------------------+
| Field             | Value                      |
+-------------------+----------------------------+
| availability_zone | az2                        |
| created_at        | 2022-04-05T09:21:02.209826 |
| deleted           | False                      |
| deleted_at        | None                       |
| id                | 3                          |
| name              | az2                        |
| updated_at        | None                       |
+-------------------+----------------------------+


vagrant@controller:~$ openstack aggregate list
+----+------+-------------------+
| ID | Name | Availability Zone |
+----+------+-------------------+
|  3 | az2  | az2               |
+----+------+-------------------+


vagrant@controller:~$ openstack network list
+--------------------------------------+------------+----------------------------------------------------------------------------+
| ID                                   | Name       | Subnets                                                                    |
+--------------------------------------+------------+----------------------------------------------------------------------------+
| 11741fda-b0a3-49c4-9171-89ce451b17c4 | public     | 0ba83d7b-7fdc-4c6d-a470-fba072bfc280, 41fceb65-4009-4920-a19a-983169b5dbe6 |
| e5ab4a07-4f40-41a2-be23-48f8b11f4630 | private    | 46363502-58f6-4a43-8b05-b8aab9cbe8a5, e5b354ae-cddf-4b2b-bafb-bdbee30eb9da |
| e83755ae-b262-4cb7-bf1a-d36815dbc148 | mynetwork1 | 49b594ff-a997-4ce8-9af0-598e15128df8                                       |
| f883cafc-3607-40eb-a6f4-1cb4fd010038 | mynetwork2 | 8a403578-8fbc-48d6-a248-ac06865954da                                       |
+--------------------------------------+------------+----------------------------------------------------------------------------+


# -------------------------------
# Create the VMs
# -------------------------------
vagrant@controller:~$ openstack server create --image cirros-0.3.5-x86_64-disk --flavor 1 --key-name mykey --network  mynetwork1 --security-group mysecgroup --availability-zone nova myvm1
+-------------------------------------+-----------------------------------------------------------------+
| Field                               | Value                                                           |
+-------------------------------------+-----------------------------------------------------------------+
| OS-DCF:diskConfig                   | MANUAL                                                          |
| OS-EXT-AZ:availability_zone         | nova                                                            |
| OS-EXT-SRV-ATTR:host                | None                                                            |
| OS-EXT-SRV-ATTR:hypervisor_hostname | None                                                            |
| OS-EXT-SRV-ATTR:instance_name       |                                                                 |
| OS-EXT-STS:power_state              | NOSTATE                                                         |
| OS-EXT-STS:task_state               | scheduling                                                      |
| OS-EXT-STS:vm_state                 | building                                                        |
| OS-SRV-USG:launched_at              | None                                                            |
| OS-SRV-USG:terminated_at            | None                                                            |
| accessIPv4                          |                                                                 |
| accessIPv6                          |                                                                 |
| addresses                           |                                                                 |
| adminPass                           | RiKKmHVmf6ja                                                    |
| config_drive                        |                                                                 |
| created                             | 2022-04-05T09:28:01Z                                            |
| flavor                              | m1.tiny (1)                                                     |
| hostId                              |                                                                 |
| id                                  | 2f72a0c5-0d50-4944-b83b-75e3a92d10af                            |
| image                               | cirros-0.3.5-x86_64-disk (2abd2f03-f088-4f64-b97f-fc71087b6d85) |
| key_name                            | mykey                                                           |
| name                                | myvm1                                                           |
| progress                            | 0                                                               |
| project_id                          | 200d842136a249ff9778d3bb2d9e9202                                |
| properties                          |                                                                 |
| security_groups                     | name='8ad65813-4333-407a-8428-9a2657c76ed2'                     |
| status                              | BUILD                                                           |
| updated                             | 2022-04-05T09:28:01Z                                            |
| user_id                             | bf7206456b88476997d567f3d57c808f                                |
| volumes_attached                    |                                                                 |
+-------------------------------------+-----------------------------------------------------------------+


vagrant@controller:~$ openstack server list
+--------------------------------------+-------------+---------+--------------------------------+--------------------------+---------+
| ID                                   | Name        | Status  | Networks                       | Image                    | Flavor  |
+--------------------------------------+-------------+---------+--------------------------------+--------------------------+---------+
| 2f72a0c5-0d50-4944-b83b-75e3a92d10af | myvm1       | ACTIVE  | mynetwork1=10.1.1.4            | cirros-0.3.5-x86_64-disk | m1.tiny |
| 95bdba24-2f9a-4e59-92aa-db94be3f49e2 | my_first_vm | SHUTOFF | public=2001:db8::d, 172.24.4.9 | cirros-0.3.5-x86_64-disk | m1.tiny |
+--------------------------------------+-------------+---------+--------------------------------+--------------------------+---------+



vagrant@controller:~$ openstack console log show myvm1
[    0.000000] Initializing cgroup subsys cpuset
[    0.000000] Initializing cgroup subsys cpu
[    0.000000] Linux version 3.2.0-80-virtual (buildd@batsu) (gcc version 4.6.3 (Ubuntu/Linaro 4.6.3-1ubuntu5) ) #116-Ubuntu SMP Mon Mar 23 17:28:52 UTC 2015 (Ubuntu 3.2.0-80.116-virtual 3.2.68)
[    0.000000] Command line: LABEL=cirros-rootfs ro console=tty1 console=ttyS0
[    0.000000] KERNEL supported cpus:
[    0.000000]   Intel GenuineIntel
[    0.000000]   AMD AuthenticAMD
[    0.000000]   Centaur CentaurHauls
[    0.000000] BIOS-provided physical RAM map:
[    0.000000]  BIOS-e820: 0000000000000000 - 000000000009fc00 (usable)
[    0.000000]  BIOS-e820: 000000000009fc00 - 00000000000a0000 (reserved)
[    0.000000]  BIOS-e820: 00000000000f0000 - 0000000000100000 (reserved)
[    0.000000]  BIOS-e820: 0000000000100000 - 000000001ffdc000 (usable)
[    0.000000]  BIOS-e820: 000000001ffdc000 - 0000000020000000 (reserved)
[    0.000000]  BIOS-e820: 00000000fffc0000 - 0000000100000000 (reserved)
[    0.000000] NX (Execute Disable) protection: active
[    0.000000] SMBIOS 2.8 present.
[    0.000000] No AGP bridge found
[    0.000000] last_pfn = 0x1ffdc max_arch_pfn = 0x400000000
[    0.000000] x86 PAT enabled: cpu 0, old 0x7040600070406, new 0x7010600070106
[    0.000000] found SMP MP-table at [ffff8800000f6a40] f6a40
[    0.000000] init_memory_mapping: 0000000000000000-000000001ffdc000
[    0.000000] RAMDISK: 1fc6d000 - 1ffcc000
[    0.000000] ACPI: RSDP 00000000000f6830 00014 (v00 BOCHS )
[    0.000000] ACPI: RSDT 000000001ffe18a9 00030 (v01 BOCHS  BXPCRSDT 00000001 BXPC 00000001)
[    0.000000] ACPI: FACP 000000001ffe1785 00074 (v01 BOCHS  BXPCFACP 00000001 BXPC 00000001)
[    0.000000] ACPI: DSDT 000000001ffe0040 01745 (v01 BOCHS  BXPCDSDT 00000001 BXPC 00000001)
[    0.000000] ACPI: FACS 000000001ffe0000 00040
[    0.000000] ACPI: APIC 000000001ffe17f9 00078 (v01 BOCHS  BXPCAPIC 00000001 BXPC 00000001)
[    0.000000] ACPI: HPET 000000001ffe1871 00038 (v01 BOCHS  BXPCHPET 00000001 BXPC 00000001)
[    0.000000] No NUMA configuration found
[    0.000000] Faking a node at 0000000000000000-000000001ffdc000
[    0.000000] Initmem setup node 0 0000000000000000-000000001ffdc000
[    0.000000]   NODE_DATA [000000001ffd4000 - 000000001ffd8fff]
[    0.000000] Zone PFN ranges:
[    0.000000]   DMA      0x00000010 -> 0x00001000
[    0.000000]   DMA32    0x00001000 -> 0x00100000
[    0.000000]   Normal   empty
[    0.000000] Movable zone start PFN for each node
[    0.000000] early_node_map[2] active PFN ranges
[    0.000000]     0: 0x00000010 -> 0x0000009f
[    0.000000]     0: 0x00000100 -> 0x0001ffdc
[    0.000000] ACPI: PM-Timer IO Port: 0x608
[    0.000000] ACPI: LAPIC (acpi_id[0x00] lapic_id[0x00] enabled)
[    0.000000] ACPI: LAPIC_NMI (acpi_id[0xff] dfl dfl lint[0x1])
[    0.000000] ACPI: IOAPIC (id[0x00] address[0xfec00000] gsi_base[0])
[    0.000000] IOAPIC[0]: apic_id 0, version 32, address 0xfec00000, GSI 0-23
[    0.000000] ACPI: INT_SRC_OVR (bus 0 bus_irq 0 global_irq 2 dfl dfl)
[    0.000000] ACPI: INT_SRC_OVR (bus 0 bus_irq 5 global_irq 5 high level)
[    0.000000] ACPI: INT_SRC_OVR (bus 0 bus_irq 9 global_irq 9 high level)
[    0.000000] ACPI: INT_SRC_OVR (bus 0 bus_irq 10 global_irq 10 high level)
[    0.000000] ACPI: INT_SRC_OVR (bus 0 bus_irq 11 global_irq 11 high level)
[    0.000000] Using ACPI (MADT) for SMP configuration information
[    0.000000] ACPI: HPET id: 0x8086a201 base: 0xfed00000
[    0.000000] SMP: Allowing 1 CPUs, 0 hotplug CPUs
[    0.000000] PM: Registered nosave memory: 000000000009f000 - 00000000000a0000
[    0.000000] PM: Registered nosave memory: 00000000000a0000 - 00000000000f0000
[    0.000000] PM: Registered nosave memory: 00000000000f0000 - 0000000000100000
[    0.000000] Allocating PCI resources starting at 20000000 (gap: 20000000:dffc0000)
[    0.000000] Booting paravirtualized kernel on bare hardware
[    0.000000] setup_percpu: NR_CPUS:64 nr_cpumask_bits:64 nr_cpu_ids:1 nr_node_ids:1
[    0.000000] PERCPU: Embedded 27 pages/cpu @ffff88001fa00000 s78848 r8192 d23552 u2097152
[    0.000000] Built 1 zonelists in Node order, mobility grouping on.  Total pages: 128870
[    0.000000] Policy zone: DMA32
[    0.000000] Kernel command line: LABEL=cirros-rootfs ro console=tty1 console=ttyS0
[    0.000000] PID hash table entries: 2048 (order: 2, 16384 bytes)
[    0.000000] Checking aperture...
[    0.000000] No AGP bridge found
[    0.000000] Memory: 496156k/524144k available (6576k kernel code, 452k absent, 27536k reserved, 6620k data, 928k init)
[    0.000000] SLUB: Genslabs=15, HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] Hierarchical RCU implementation.
[    0.000000] 	RCU dyntick-idle grace-period acceleration is enabled.
[    0.000000] NR_IRQS:4352 nr_irqs:256 16
[    0.000000] Console: colour VGA+ 80x25
[    0.000000] console [tty1] enabled
[    0.000000] console [ttyS0] enabled
[    0.000000] allocated 4194304 bytes of page_cgroup
[    0.000000] please try 'cgroup_disable=memory' option if you don't want memory cgroups
[    0.000000] Fast TSC calibration failed
[    0.000000] TSC: Unable to calibrate against PIT
[    0.000000] TSC: using HPET reference calibration
[    0.000000] Detected 2592.002 MHz processor.
[    0.029227] Calibrating delay loop (skipped), value calculated using timer frequency.. 5184.00 BogoMIPS (lpj=10368008)
[    0.029866] pid_max: default: 32768 minimum: 301
[    0.034697] Security Framework initialized
[    0.036119] AppArmor: AppArmor initialized
[    0.036490] Yama: becoming mindful.
[    0.039697] Dentry cache hash table entries: 65536 (order: 7, 524288 bytes)
[    0.040729] Inode-cache hash table entries: 32768 (order: 6, 262144 bytes)
[    0.041433] Mount-cache hash table entries: 256
[    0.049583] Initializing cgroup subsys cpuacct
[    0.050085] Initializing cgroup subsys memory
[    0.050771] Initializing cgroup subsys devices
[    0.051031] Initializing cgroup subsys freezer
[    0.051213] Initializing cgroup subsys blkio
[    0.051410] Initializing cgroup subsys perf_event
[    0.053003] mce: CPU supports 10 MCE banks
[    0.054495] SMP alternatives: switching to UP code
[    0.192756] Freeing SMP alternatives: 24k freed
[    0.193792] ACPI: Core revision 20110623
[    0.211768] ftrace: allocating 26610 entries in 105 pages
[    0.231598] ..TIMER: vector=0x30 apic1=0 pin1=2 apic2=-1 pin2=-1
[    0.268099] CPU0: AMD QEMU Virtual CPU version 2.5+ stepping 03
[    0.272016] Performance Events: Broken PMU hardware detected, using software events only.
[    0.273301] NMI watchdog disabled (cpu0): hardware events not enabled
[    0.274422] Brought up 1 CPUs
[    0.274868] Total of 1 processors activated (5184.00 BogoMIPS).
[    0.285418] devtmpfs: initialized
[    0.304660] EVM: security.selinux
[    0.305216] EVM: security.SMACK64
[    0.305664] EVM: security.capability
[    0.314031] print_constraints: dummy:
[    0.316500] RTC time:  9:28:08, date: 04/05/22
[    0.317981] NET: Registered protocol family 16
[    0.320019] ACPI: bus type pci registered
[    0.320019] PCI: Using configuration type 1 for base access
[    0.328019] bio: create slab <bio-0> at 0
[    0.333089] ACPI: Added _OSI(Module Device)
[    0.333676] ACPI: Added _OSI(Processor Device)
[    0.334233] ACPI: Added _OSI(3.0 _SCP Extensions)
[    0.334820] ACPI: Added _OSI(Processor Aggregator Device)
[    0.351538] ACPI: Interpreter enabled
[    0.352019] ACPI: (supports S0 S3 S4 S5)
[    0.352021] ACPI: Using IOAPIC for interrupt routing
[    0.373771] ACPI: No dock devices found.
[    0.374490] HEST: Table not found.
[    0.374985] PCI: Using host bridge windows from ACPI; if necessary, use "pci=nocrs" and report a bug
[    0.376022] ACPI: PCI Root Bridge [PCI0] (domain 0000 [bus 00-ff])
[    0.376022] pci_root PNP0A03:00: host bridge window [io  0x0000-0x0cf7]
[    0.376022] pci_root PNP0A03:00: host bridge window [io  0x0d00-0xffff]
[    0.376067] pci_root PNP0A03:00: host bridge window [mem 0x000a0000-0x000bffff]
[    0.377002] pci_root PNP0A03:00: host bridge window [mem 0x20000000-0xfebfffff]
[    0.389911] pci 0000:00:01.3: quirk: [io  0x0600-0x063f] claimed by PIIX4 ACPI
[    0.390849] pci 0000:00:01.3: quirk: [io  0x0700-0x070f] claimed by PIIX4 SMB
[    0.631079]  pci0000:00: Unable to request _OSC control (_OSC support mask: 0x1e)
[    0.652129] ACPI: PCI Interrupt Link [LNKA] (IRQs 5 *10 11)
[    0.653459] ACPI: PCI Interrupt Link [LNKB] (IRQs 5 *10 11)
[    0.654519] ACPI: PCI Interrupt Link [LNKC] (IRQs 5 10 *11)
[    0.656628] ACPI: PCI Interrupt Link [LNKD] (IRQs 5 10 *11)
[    0.657541] ACPI: PCI Interrupt Link [LNKS] (IRQs *9)
[    0.660526] vgaarb: device added: PCI:0000:00:02.0,decodes=io+mem,owns=io+mem,locks=none
[    0.661455] vgaarb: loaded
[    0.661824] vgaarb: bridge control possible 0000:00:02.0
[    0.663259] i2c-core: driver [aat2870] using legacy suspend method
[    0.663976] i2c-core: driver [aat2870] using legacy resume method
[    0.664150] SCSI subsystem initialized
[    0.665591] usbcore: registered new interface driver usbfs
[    0.666409] usbcore: registered new interface driver hub
[    0.667219] usbcore: registered new device driver usb
[    0.669290] PCI: Using ACPI for IRQ routing
[    0.673776] NetLabel: Initializing
[    0.674222] NetLabel:  domain hash size = 128
[    0.674692] NetLabel:  protocols = UNLABELED CIPSOv4
[    0.675772] NetLabel:  unlabeled traffic allowed by default
[    0.676041] HPET: 3 timers in total, 0 timers will be used for per-cpu timer
[    0.676209] hpet0: at MMIO 0xfed00000, IRQs 2, 8, 0
[    0.676875] hpet0: 3 comparators, 64-bit 100.000000 MHz counter
[    0.689333] Switching to clocksource hpet
[    0.751736] AppArmor: AppArmor Filesystem Enabled
[    0.752297] pnp: PnP ACPI init
[    0.752297] ACPI: bus type pnp registered
[    0.762867] pnp: PnP ACPI: found 10 devices
[    0.763510] ACPI: ACPI bus type pnp unregistered
[    0.784907] NET: Registered protocol family 2
[    0.796154] IP route cache hash table entries: 4096 (order: 3, 32768 bytes)
[    0.800391] TCP established hash table entries: 16384 (order: 6, 262144 bytes)
[    0.802428] TCP bind hash table entries: 16384 (order: 6, 262144 bytes)
[    0.803417] TCP: Hash tables configured (established 16384 bind 16384)
[    0.804266] TCP reno registered
[    0.804826] UDP hash table entries: 256 (order: 1, 8192 bytes)
[    0.805583] UDP-Lite hash table entries: 256 (order: 1, 8192 bytes)
[    0.807076] NET: Registered protocol family 1
[    0.807759] pci 0000:00:00.0: Limiting direct PCI/PCI transfers
[    0.808475] pci 0000:00:01.0: PIIX3: Enabling Passive Release
[    0.809400] pci 0000:00:01.0: Activating ISA DMA hang workarounds
[    0.811658] ACPI: PCI Interrupt Link [LNKD] enabled at IRQ 11
[    0.812681] pci 0000:00:01.2: PCI INT D -> Link[LNKD] -> GSI 11 (level, high) -> IRQ 11
[    0.814108] pci 0000:00:01.2: PCI INT D disabled
[    0.831154] Trying to unpack rootfs image as initramfs...
[    0.854612] audit: initializing netlink socket (disabled)
[    0.855692] type=2000 audit(1649150887.848:1): initialized
[    0.944851] HugeTLB registered 2 MB page size, pre-allocated 0 pages
[    0.962630] VFS: Disk quotas dquot_6.5.2
[    0.963508] Dquot-cache hash table entries: 512 (order 0, 4096 bytes)
[    0.989594] fuse init (API version 7.17)
[    0.990998] msgmni has been set to 969
[    1.010203] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 253)
[    1.015249] io scheduler noop registered
[    1.015810] io scheduler deadline registered (default)
[    1.016681] io scheduler cfq registered
[    1.036385] pci_hotplug: PCI Hot Plug PCI Core version: 0.5
[    1.037647] pciehp: PCI Express Hot Plug Controller Driver version: 0.4
[    1.040240] input: Power Button as /devices/LNXSYSTM:00/LNXPWRBN:00/input/input0
[    1.041649] ACPI: Power Button [PWRF]
[    1.083046] ERST: Table is not found!
[    1.083590] GHES: HEST is not enabled!
[    1.085034] ACPI: PCI Interrupt Link [LNKC] enabled at IRQ 10
[    1.085727] virtio-pci 0000:00:03.0: PCI INT A -> Link[LNKC] -> GSI 10 (level, high) -> IRQ 10
[    1.088055] virtio-pci 0000:00:04.0: PCI INT A -> Link[LNKD] -> GSI 11 (level, high) -> IRQ 11
[    1.093863] ACPI: PCI Interrupt Link [LNKA] enabled at IRQ 10
[    1.094522] virtio-pci 0000:00:05.0: PCI INT A -> Link[LNKA] -> GSI 10 (level, high) -> IRQ 10
[    1.096713] Serial: 8250/16550 driver, 32 ports, IRQ sharing enabled
[    1.119203] serial8250: ttyS0 at I/O 0x3f8 (irq = 4) is a 16550A
[    1.193136] 00:05: ttyS0 at I/O 0x3f8 (irq = 4) is a 16550A
[    1.227637] Linux agpgart interface v0.103
[    1.298467] brd: module loaded
[    1.316239] loop: module loaded
[    1.346346]  vda: vda1
[    1.376875] scsi0 : ata_piix
[    1.378729] scsi1 : ata_piix
[    1.380750] ata1: PATA max MWDMA2 cmd 0x1f0 ctl 0x3f6 bmdma 0xc0a0 irq 14
[    1.381821] ata2: PATA max MWDMA2 cmd 0x170 ctl 0x376 bmdma 0xc0a8 irq 15
[    1.388112] Fixed MDIO Bus: probed
[    1.388112] tun: Universal TUN/TAP device driver, 1.6
[    1.388112] tun: (C) 1999-2004 Max Krasnyansky <maxk@qualcomm.com>
[    1.400868] PPP generic driver version 2.4.2
[    1.409392] ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
[    1.410561] ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
[    1.411554] uhci_hcd: USB Universal Host Controller Interface driver
[    1.414020] uhci_hcd 0000:00:01.2: PCI INT D -> Link[LNKD] -> GSI 11 (level, high) -> IRQ 11
[    1.415002] uhci_hcd 0000:00:01.2: UHCI Host Controller
[    1.422517] uhci_hcd 0000:00:01.2: new USB bus registered, assigned bus number 1
[    1.423236] uhci_hcd 0000:00:01.2: irq 11, io base 0x0000c040
[    1.434723] Freeing initrd memory: 3452k freed
[    1.440018] hub 1-0:1.0: USB hub found
[    1.440705] hub 1-0:1.0: 2 ports detected
[    1.447104] usbcore: registered new interface driver libusual
[    1.448155] i8042: PNP: PS/2 Controller [PNP0303:KBD,PNP0f13:MOU] at 0x60,0x64 irq 1,12
[    1.452346] serio: i8042 KBD port at 0x60,0x64 irq 1
[    1.452734] serio: i8042 AUX port at 0x60,0x64 irq 12
[    1.454605] mousedev: PS/2 mouse device common for all mice
[    1.458461] input: AT Translated Set 2 keyboard as /devices/platform/i8042/serio0/input/input1
[    1.460577] rtc_cmos 00:01: RTC can wake from S4
[    1.463455] rtc_cmos 00:01: rtc core: registered rtc_cmos as rtc0
[    1.465241] rtc0: alarms up to one day, y3k, 114 bytes nvram, hpet irqs
[    1.467279] device-mapper: uevent: version 1.0.3
[    1.470387] device-mapper: ioctl: 4.22.0-ioctl (2011-10-19) initialised: dm-devel@redhat.com
[    1.472483] cpuidle: using governor ladder
[    1.473189] cpuidle: using governor menu
[    1.473664] EFI Variables Facility v0.08 2004-May-17
[    1.477165] TCP cubic registered
[    1.479144] NET: Registered protocol family 10
[    1.502742] hpet1: lost 1 rtc interrupts
[    1.515865] NET: Registered protocol family 17
[    1.516360] Registering the dns_resolver key type
[    1.519060] registered taskstats version 1
[    1.633513]   Magic number: 2:242:474
[    1.634230] rtc_cmos 00:01: setting system clock to 2022-04-05 09:28:09 UTC (1649150889)
[    1.634527] powernow-k8: Processor cpuid 663 not supported
[    1.635493] BIOS EDD facility v0.16 2004-Jun-25, 0 devices found
[    1.635707] EDD information not available.
[    1.651521] Freeing unused kernel memory: 928k freed
[    1.668251] Write protecting the kernel read-only data: 12288k
[    1.702137] Freeing unused kernel memory: 1596k freed
[    1.724559] Freeing unused kernel memory: 1184k freed

info: initramfs: up at 1.79
GROWROOT: CHANGED: partition=1 start=16065 old: size=64260 end=80325 new: size=2072385,end=2088450
info: initramfs loading root from /dev/vda1
info: /etc/init.d/rc.sysinit: up at 2.84
info: container: none
Starting logging: OK
modprobe: module virtio_blk not found in modules.dep
modprobe: module virtio_net not found in modules.dep
WARN: /etc/rc3.d/S10-load-modules failed
Initializing random number generator... done.
Starting acpid: OK
cirros-ds 'local' up at 3.92
no results found for mode=local. up 4.16. searched: nocloud configdrive ec2
Starting network...
udhcpc (v1.20.1) started
Sending discover...
Sending select for 10.1.1.4...
Lease of 10.1.1.4 obtained, lease time 86400
route: SIOCADDRT: File exists
WARN: failed: route add -net "0.0.0.0/0" gw "10.1.1.1"
cirros-ds 'net' up at 4.78
checking http://169.254.169.254/2009-04-04/instance-id
successful after 1/20 tries: up 4.92. iid=i-0000000c
failed to get http://169.254.169.254/2009-04-04/user-data
warning: no ec2 metadata for user-data
found datasource (ec2, net)
Top of dropbear init script
Starting dropbear sshd: WARN: generating key of type ecdsa failed!
OK
/run/cirros/datasource/data/user-data was not '#!' or executable
=== system information ===
Platform: OpenStack Foundation OpenStack Nova
Container: none
Arch: x86_64
CPU(s): 1 @ 2592.002 MHz
Cores/Sockets/Threads: 1/1/1
Virt-type: AMD-V
RAM Size: 491MB
Disks:
NAME MAJ:MIN       SIZE LABEL         MOUNTPOINT
vda  253:0   1073741824
vda1 253:1   1061061120 cirros-rootfs /
=== sshd host keys ===
-----BEGIN SSH HOST KEY KEYS-----
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgwDXQSHLnWS1uAVoY13U4VDqAc8OPwlrihOQtlLJCsWK3gwX4cRFowk/MoBBts1On9XHqowr9i9jqLJYzfBWxy/9C7hPPx7cNQ9BRaJOCxca2ie1bSujsRPMDFtuP3aOkf25GGxuu6pt67waMJGXxYRRil1mHooxfRqKiy4Gx5eykYXz root@myvm1
ssh-dss AAAAB3NzaC1kc3MAAACBAJcgLMvdY/EMcVSRe+Svg6/B41zEH+20MPGCzSaeSIFEkmFU04OOy+RplNfMBcz5xrRakX9DyXxVz6/+LWXF6l4dC1XXkrpUM1ndfgLSKsT2om1pxRLuL5Obxaz6cWmzNJUTuABgEH9+k03L/LqOl4acG5eAw9rC8yvDtlBaUi2jAAAAFQDPk3RlhetcM1T3e2O/ATKvgoohpQAAAIASsVTS9w8z0IbK6ldWQa3Voc0pKgJbN7j53ZF/BzYU74vna+iTCew32f7/kY/X56f8PYGiWecFxf8tJUCYN9X3kGbeRgEY4OnVyI6mU3CiRbRQn5uZBDyXqgjsNjixEzm5r935PLm6XkaTZMbakxae+CKzdyasWnABhJwOw3jBkwAAAIBUnMFkjykpYsc7xMTk0rk6wj76laZTpV3GXeA/E2sT0SF4bDdnGKt9xKlbllUHDRyVC/FobfSdsKAdzske3ifBDrNjrQbr/myGEOsJnaDMYZ4/0CP+v/FMcSQyzzzo14aiX027hDZvhUJKMkfrljOgHyAjQ2TOSKQPurVXAgKNBQ== root@myvm1
-----END SSH HOST KEY KEYS-----
=== network info ===
if-info: lo,up,127.0.0.1,8,::1
if-info: eth0,up,10.1.1.4,24,fe80::f816:3eff:fe6d:cd13
ip-route:default via 10.1.1.1 dev eth0
ip-route:10.1.1.0/24 dev eth0  src 10.1.1.4
ip-route:169.254.169.254 via 10.1.1.1 dev eth0
=== datasource: ec2 net ===
instance-id: i-0000000c
name: N/A
availability-zone: nova
local-hostname: myvm1.novalocal
launch-index: 0
=== cirros: current=0.3.5 uptime=9.91 ===
  ____               ____  ____
 / __/ __ ____ ____ / __ \/ __/
/ /__ / // __// __// /_/ /\ \
\___//_//_/  /_/   \____/___/
   http://cirros-cloud.net


login as 'cirros' user. default password: 'cubswin:)'. use 'sudo' for root.



vagrant@controller:~$ openstack aggregate list
+----+------+-------------------+
| ID | Name | Availability Zone |
+----+------+-------------------+
|  3 | az2  | az2               |
+----+------+-------------------+


vagrant@controller:~$ openstack aggregate show az2
+-------------------+----------------------------+
| Field             | Value                      |
+-------------------+----------------------------+
| availability_zone | az2                        |
| created_at        | 2022-04-05T09:21:02.000000 |
| deleted           | False                      |
| deleted_at        | None                       |
| hosts             | []                         |
| id                | 3                          |
| name              | az2                        |
| properties        |                            |
| updated_at        | None                       |
+-------------------+----------------------------+


vagrant@controller:~$ openstack server create --image cirros-0.3.5-x86_64-disk --flavor 1 --key-name mykey --network  mynetwork2 --security-group mysecgroup --availability-zone az2 myvm2
+-------------------------------------+-----------------------------------------------------------------+
| Field                               | Value                                                           |
+-------------------------------------+-----------------------------------------------------------------+
| OS-DCF:diskConfig                   | MANUAL                                                          |
| OS-EXT-AZ:availability_zone         | az2                                                             |
| OS-EXT-SRV-ATTR:host                | None                                                            |
| OS-EXT-SRV-ATTR:hypervisor_hostname | None                                                            |
| OS-EXT-SRV-ATTR:instance_name       |                                                                 |
| OS-EXT-STS:power_state              | NOSTATE                                                         |
| OS-EXT-STS:task_state               | scheduling                                                      |
| OS-EXT-STS:vm_state                 | building                                                        |
| OS-SRV-USG:launched_at              | None                                                            |
| OS-SRV-USG:terminated_at            | None                                                            |
| accessIPv4                          |                                                                 |
| accessIPv6                          |                                                                 |
| addresses                           |                                                                 |
| adminPass                           | RMK7con7T4Qq                                                    |
| config_drive                        |                                                                 |
| created                             | 2022-04-05T09:37:49Z                                            |
| flavor                              | m1.tiny (1)                                                     |
| hostId                              |                                                                 |
| id                                  | 37234661-96b6-4e2f-9001-715d4807873f                            |
| image                               | cirros-0.3.5-x86_64-disk (2abd2f03-f088-4f64-b97f-fc71087b6d85) |
| key_name                            | mykey                                                           |
| name                                | myvm2                                                           |
| progress                            | 0                                                               |
| project_id                          | 200d842136a249ff9778d3bb2d9e9202                                |
| properties                          |                                                                 |
| security_groups                     | name='8ad65813-4333-407a-8428-9a2657c76ed2'                     |
| status                              | BUILD                                                           |
| updated                             | 2022-04-05T09:37:48Z                                            |
| user_id                             | bf7206456b88476997d567f3d57c808f                                |
| volumes_attached                    |                                                                 |
+-------------------------------------+-----------------------------------------------------------------+
vagrant@controller:~$ openstack volume list

vagrant@controller:~$ openstack volume create --size=1 extra_space
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| attachments         | []                                   |
| availability_zone   | nova                                 |
| bootable            | false                                |
| consistencygroup_id | None                                 |
| created_at          | 2022-04-05T09:40:53.858997           |
| description         | None                                 |
| encrypted           | False                                |
| id                  | 94359400-0030-432d-9661-e7add324dfc9 |
| migration_status    | None                                 |
| multiattach         | False                                |
| name                | extra_space                          |
| properties          |                                      |
| replication_status  | None                                 |
| size                | 1                                    |
| snapshot_id         | None                                 |
| source_volid        | None                                 |
| status              | creating                             |
| type                | lvmdriver-1                          |
| updated_at          | None                                 |
| user_id             | bf7206456b88476997d567f3d57c808f     |
+---------------------+--------------------------------------+
vagrant@controller:~$ openstack volume list
+--------------------------------------+-------------+--------+------+-------------+
| ID                                   | Name        | Status | Size | Attached to |
+--------------------------------------+-------------+--------+------+-------------+
| 94359400-0030-432d-9661-e7add324dfc9 | extra_space | error  |    1 |             |
+--------------------------------------+-------------+--------+------+-------------+
vagrant@controller:~$ openstack volume --help
Command "volume" matches:
  volume backup create
  volume backup delete
  volume backup list
  volume backup restore
  volume backup set
  volume backup show
  volume create
  volume delete
  volume host failover
  volume host set
  volume list
  volume migrate
  volume qos associate
  volume qos create
  volume qos delete
  volume qos disassociate
  volume qos list
  volume qos set
  volume qos show
  volume qos unset
  volume service list
  volume service set
  volume set
  volume show
  volume snapshot create
  volume snapshot delete
  volume snapshot list
  volume snapshot set
  volume snapshot show
  volume snapshot unset
  volume transfer request accept
  volume transfer request create
  volume transfer request delete
  volume transfer request list
  volume transfer request show
  volume type create
  volume type delete
  volume type list
  volume type set
  volume type show
  volume type unset
  volume unset
vagrant@controller:~$ openstack volume show extra_space
+--------------------------------+--------------------------------------+
| Field                          | Value                                |
+--------------------------------+--------------------------------------+
| attachments                    | []                                   |
| availability_zone              | nova                                 |
| bootable                       | false                                |
| consistencygroup_id            | None                                 |
| created_at                     | 2022-04-05T09:40:54.000000           |
| description                    | None                                 |
| encrypted                      | False                                |
| id                             | 94359400-0030-432d-9661-e7add324dfc9 |
| migration_status               | None                                 |
| multiattach                    | False                                |
| name                           | extra_space                          |
| os-vol-host-attr:host          | None                                 |
| os-vol-mig-status-attr:migstat | None                                 |
| os-vol-mig-status-attr:name_id | None                                 |
| os-vol-tenant-attr:tenant_id   | 200d842136a249ff9778d3bb2d9e9202     |
| properties                     |                                      |
| replication_status             | None                                 |
| size                           | 1                                    |
| snapshot_id                    | None                                 |
| source_volid                   | None                                 |
| status                         | error                                |
| type                           | lvmdriver-1                          |
| updated_at                     | 2022-04-05T09:40:54.000000           |
| user_id                        | bf7206456b88476997d567f3d57c808f     |
+--------------------------------+--------------------------------------+
vagrant@controller:~$ sudo ip netns ls
qrouter-bd7ceb3a-49f4-418d-a9c5-88ccc1212df0
qdhcp-f883cafc-3607-40eb-a6f4-1cb4fd010038
qdhcp-e83755ae-b262-4cb7-bf1a-d36815dbc148
qrouter-f4c4910a-4b39-4dd9-8928-77fefa6a3ec0
qdhcp-e5ab4a07-4f40-41a2-be23-48f8b11f4630
vagrant@controller:~$ sudo ip netns exec  qrouter-bd7ceb3a-49f4-418d-a9c5-88ccc1212df0 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
44: qr-ee5eb998-ee: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default qlen 1
    link/ether fa:16:3e:28:2c:b4 brd ff:ff:ff:ff:ff:ff
    inet 10.1.1.1/24 brd 10.1.1.255 scope global qr-ee5eb998-ee
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe28:2cb4/64 scope link
       valid_lft forever preferred_lft forever
45: qr-1891b25a-50: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN group default qlen 1
    link/ether fa:16:3e:8d:03:7b brd ff:ff:ff:ff:ff:ff
    inet 10.2.2.1/24 brd 10.2.2.255 scope global qr-1891b25a-50
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe8d:37b/64 scope link
       valid_lft forever preferred_lft forever
vagrant@controller:~$ #sudo ip netns exec  qrouter-bd7ceb3a-49f4-418d-a9c5-88ccc1212df0 ssh -i ~/.ssh/mykey cirros@10.1.1
vagrant@controller:~$ openstack server list
+--------------------------------------+-------------+---------+--------------------------------+--------------------------+---------+
| ID                                   | Name        | Status  | Networks                       | Image                    | Flavor  |
+--------------------------------------+-------------+---------+--------------------------------+--------------------------+---------+
| 37234661-96b6-4e2f-9001-715d4807873f | myvm2       | ACTIVE  | mynetwork2=10.2.2.11           | cirros-0.3.5-x86_64-disk | m1.tiny |
| 2f72a0c5-0d50-4944-b83b-75e3a92d10af | myvm1       | ACTIVE  | mynetwork1=10.1.1.4            | cirros-0.3.5-x86_64-disk | m1.tiny |
| 95bdba24-2f9a-4e59-92aa-db94be3f49e2 | my_first_vm | SHUTOFF | public=2001:db8::d, 172.24.4.9 | cirros-0.3.5-x86_64-disk | m1.tiny |
+--------------------------------------+-------------+---------+--------------------------------+--------------------------+---------+
vagrant@controller:~$ sudo ip netns exec  qrouter-bd7ceb3a-49f4-418d-a9c5-88ccc1212df0 ssh -i ~/.ssh/mykey cirros@10.1.1.4
The authenticity of host '10.1.1.4 (10.1.1.4)' can't be established.
RSA key fingerprint is SHA256:cvT8uuPeCgRS/hjbVsYKzM99GFBDFC7cnFzJxzah0no.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.1.1.4' (RSA) to the list of known hosts.
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc pfifo_fast qlen 1000
    link/ether fa:16:3e:6d:cd:13 brd ff:ff:ff:ff:ff:ff
    inet 10.1.1.4/24 brd 10.1.1.255 scope global eth0
    inet6 fe80::f816:3eff:fe6d:cd13/64 scope link
       valid_lft forever preferred_lft forever
$ ping 10.2.2.11
PING 10.2.2.11 (10.2.2.11): 56 data bytes
64 bytes from 10.2.2.11: seq=0 ttl=63 time=5.786 ms
64 bytes from 10.2.2.11: seq=1 ttl=63 time=1.445 ms
^C
--- 10.2.2.11 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 1.445/3.615/5.786 ms
$


```


## Keystone
```
root@controller:/home/ubuntu# openstack --os-auth-url http://controller:5000/v3 \
>   --os-project-domain-name Default --os-user-domain-name Default \
>   --os-project-name admin --os-username admin token issue
Password: 
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2022-04-06T09:10:16+0000                                                                                                                                                                |
| id         | gAAAAABiTUroeDxnoM_AlMiHcKPbTdcVVUFuLX8dhrK3Pt9W4FXloXMfq6_zcx2Mxhg7MfLDFioYzsmADutnB43YLT2QLpBdr5AVt3AUc3eh-_eoqw945uHGmHSDvDUVqNsJsmaTQkpKLJmxhHAxdw7aZJmsrU_5gZTdnotmmewLz6l84omcRG4 |
| project_id | 1fdffce838fa4ac88f123f575aee60c3                                                                                                                                                        |
| user_id    | 6d1ef754be174f45afcc0bab8b9a5ea5                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```
