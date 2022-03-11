# openstack

## Stand alone installation on 1 VM with devstack
* https://www.medo64.com/2021/04/devstack-on-virtualbox-ubuntu-20-04/
```
Terminal 

echo "$USER ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/$USER

sudo apt update
sudo apt dist-upgrade --yes

sudo useradd -s /bin/bash -d /opt/stack -m stack
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
sudo su - stack

STACK_BRANCH=stable/wallaby
git clone https://github.com/openstack-dev/devstack.git -b $STACK_BRANCH devstack/
cd devstack
cp samples/local.conf .
sed -i 's/#HOST_IP=w.x.y.z/HOST_IP=10.0.2.15/' local.conf
sed -i 's/ADMIN_PASSWORD=nomoresecret/ADMIN_PASSWORD=devstack/' local.conf
echo "#Enable heat services" >> local.conf
echo "enable_service h-eng h-api h-api-cfn h-api-cw" >> local.conf
echo "#Enable heat plugin" >> local.conf
echo "enable_plugin heat https://git.openstack.org/openstack/heat $STACK_BRANCH" >> local.conf

```

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
