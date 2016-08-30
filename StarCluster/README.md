# StarCluster

```bash
$ yum install epel-release
$ yum install python-pip
$ yum install gcc
$ yum install python-devel
$ yum install libffi-devel
$ yum install openssl-devel

$ pip install StarCluster


$ starcluster help
select #2

vi .starcluster/config
# Enter AWS keys in empty fields:
AWS_ACCESS_KEY_ID = # add value
AWS_SECRET_ACCESS_KEY = # add value

# Add omicia key pair
# Note: [key is no a free-form entry - it must match the filename in KEY_LOCATION
[key om-aws-keypair]
KEY_LOCATION=~/.ssh/om-aws-keypair.pem

# Use it in smallcluster
KEYNAME = om-aws-keypair
```

# Useful commands

```bash
starcluster start <name>
starcluster terminate <name>

starcluster sshmaster mycluster

starcluster put mycluster /path/to/local/file/or/dir /remote/path/
starcluster get mycluster /path/to/remote/file/or/dir /local/path/
```

# Starting cluster mycluster
mycluster is an arbitrary name for the cluster specified in the config file
```
starcluster start mycluster
StarCluster - (http://star.mit.edu/cluster) (v. 0.95.6)
Software Tools for Academics and Researchers (STAR)
Please submit bug reports to starcluster@mit.edu

>>> Using default cluster template: smallcluster
>>> Validating cluster template settings...
>>> Cluster template settings are valid
>>> Starting cluster...
>>> Launching a 2-node cluster...
>>> Creating security group @sc-mycluster...
Reservation:r-4241f7fc
>>> Waiting for instances to propagate...
2/2 |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||| 100%  
>>> Waiting for cluster to come up... (updating every 30s)
>>> Waiting for all nodes to be in a 'running' state...
2/2 |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||| 100%  
>>> Waiting for SSH to come up on all nodes...
2/2 |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||| 100%  
>>> Waiting for cluster to come up took 1.637 mins
>>> The master node is ec2-54-158-84-228.compute-1.amazonaws.com
>>> Configuring cluster...
>>> Running plugin starcluster.clustersetup.DefaultClusterSetup
>>> Configuring hostnames...
2/2 |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||| 100%  
>>> Creating cluster user: sgeadmin (uid: 1001, gid: 1001)
2/2 |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||| 100%  
>>> Configuring scratch space for user(s): sgeadmin
2/2 |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||| 100%  
>>> Configuring /etc/hosts on each node
2/2 |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||| 100%  
>>> Starting NFS server on master
>>> Configuring NFS exports path(s):
/home
>>> Mounting all NFS export path(s) on 1 worker node(s)
1/1 |||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||| 100%  
>>> Setting up NFS took 0.169 mins
>>> Configuring passwordless ssh for root
>>> Configuring passwordless ssh for sgeadmin
>>> Configuring cluster took 0.553 mins
>>> Starting cluster took 2.256 mins

The cluster is now ready to use. To login to the master node
as root, run:

    $ starcluster sshmaster mycluster

If you're having issues with the cluster you can reboot the
instances and completely reconfigure the cluster from
scratch using:

    $ starcluster restart mycluster

When you're finished using the cluster and wish to terminate
it and stop paying for service:

    $ starcluster terminate mycluster

Alternatively, if the cluster uses EBS instances, you can
use the 'stop' command to shutdown all nodes and put them
into a 'stopped' state preserving the EBS volumes backing
the nodes:

    $ starcluster stop mycluster

WARNING: Any data stored in ephemeral storage (usually /mnt)
will be lost!

You can activate a 'stopped' cluster by passing the -x
option to the 'start' command:

    $ starcluster start -x mycluster

This will start all 'stopped' nodes and reconfigure the
cluster.
```

# Create StarCluster compatible AMI


ssh awscluster@lando.omicia-private.com

Use current amazon image from https://aws.amazon.com/amazon-linux-ami/ 
Select ami-2a69aa47 (PV EBS-Backed 64-bit, US East N. Virginia)

``` bash
starcluster start -o -s 1 -i t1.micro -n ami-2a69aa47 imagehost
starcluster listclusters --show-ssh-status imagehost
starcluster sshmaster imagehost -u ec2-user
```

In imagehost, enable root logins

```bash
ssh -i .ssh/om-aws-keypair.pem ec2-user@ec2-54-166-106-202.compute-1.amazonaws.com
sudo sed -i.bak -e's/\#PermitRootLogin\ yes/PermitRootLogin\ without-password/g' /etc/ssh/sshd_config
sudo sed -i.bak -e's/\#UseDNS\ yes/UseDNS\ no/g' /etc/ssh/sshd_config
sudo cp -f /home/ec2-user/.ssh/authorized_keys /root/.ssh/authorized_keys
sudo service sshd reload
```

Create custom script for key assignment
```bash
cat /etc/rc.d/rc.local 
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.

touch /var/lock/subsys/local
# update ec2-ami-tools
wget http://s3.amazonaws.com/ec2-downloads/ec2-ami-tools.noarch.rpm && rpm -Uvh ec2-ami-tools.noarch.rpm
# reset root password
dd if=/dev/urandom count=50|md5sum|passwd --stdin root
dd if=/dev/urandom count=50|md5sum|passwd --stdin ec2-user
# update root ssh keys
sleep 40
if [ ! -d /root/.ssh ]; then
    mkdir -p /root/.ssh
    chmod 700 /root/.ssh
fi
wget http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key && cat openssh-key >>/root/.ssh/authorized_keys && chmod 600 /root/.ssh/authorized_keys
rm -f openssh-key
```

Test root login
```bash
ssh -i .ssh/om-aws-keypair.pem root@ec2-54-166-106-202.compute-1.amazonaws.com
```

Convert image to AMI
```bash
starcluster listclusters
# Shows i-946db4a5 as image id we just created
starcluster ebsimage i-946db4a5 starcluster-amazon_linux_pv-20160829
>>> Removing private data...
>>> Cleaning up SSH host keys
>>> Cleaning up /var/log
>>> Cleaning out /root
>>> Cleaning up /tmp
>>> Creating new EBS AMI...
>>> New EBS AMI created: ami-cb4c23dc
>>> Fetching block device mapping for ami-cb4c23dc 
>>> Waiting for snapshot to complete: snap-8b918c10
snap-8b918c10: |||||||||||||||||||||||||||||||||||||||||||||100% Time: 00:04:19
>>> Waiting for ami-cb4c23dc to become available... 
>>> create_image took 5.298 mins
>>> Your new AMI id is: ami-cb4c23dc
```

Cleanup
```bash
starcluster terminate imagehost
```

Test new image
```bash
starcluster start -o -s 1 -i t1.micro -n ami-cb4c23dc imagehost
starcluster sshmaster imagehost
# And terminate
starcluster terminate imagehost
```
