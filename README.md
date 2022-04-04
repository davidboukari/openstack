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
```
