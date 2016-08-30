StarCluster

yum install epel-release
yum install python-pip
yum install gcc
yum install python-devel
yum install libffi-devel
yum install openssl-devel

pip install StarCluster


starcluster help
# select #2

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


starcluster start <name>
starcluster terminate <name>

starcluster sshmaster mycluster

starcluster put mycluster /path/to/local/file/or/dir /remote/path/
starcluster get mycluster /path/to/remote/file/or/dir /local/path/



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
