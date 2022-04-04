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

