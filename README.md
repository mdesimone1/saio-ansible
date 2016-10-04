Purpose of this Git Repo
=================================
This repo provide ansible code to build an SAIO system on
 * GCE ( Google Cloud Platform )
 * Local VirtualBox via Vagrant
 * AWS ( Amazon Web Service / EC2 )

1. Before start, Please make sure you do install ansible
`sudo pip install ansible`

1. all the ansible for trigger from local ( control node )
In your playbook steps we’ll typically be using the following pattern for provisioning steps:
```
- hosts: localhost
  connection: local
  gather_facts: False
  tasks:
    - ...
```

SAIO ansible playbook via GCE
=================================
## Google Cloud Preparation
### Google Cloud Platform Setup
Prior to using this plugin, you will first need to make sure you have a
Google Cloud Platform account, enable Google Compute Engine, and create a
Service Account for API Access.

1. Log in with your Google Account and go to
   [Google Cloud Platform](https://cloud.google.com) and click on the
   `Try it free` button.
1. Create a new project and remember to record the `Project ID`
1. Next, enable the
   [Google Compute Engine API](https://console.cloud.google.com/apis/api/compute_component/)
   for your project in the API console. If prompted, review and agree to the
   terms of service.
1. While still in the API Console, go to
   [Credentials subsection](https://console.cloud.google.com/apis/credentials),
   and click `Create credentials` -> `Service account key`. In the
   next dialog, create a new service account, select `JSON` key type and
   click `Create`.
1. Download the JSON private key and save this file in a secure
   and reliable location.  This key file will be used to authorize all API
   requests to Google Compute Engine.
1. Still on the same page, click on
   [Manage service accounts](https://console.cloud.google.com/permissions/serviceaccounts)
   link to go to IAM console. Copy the `Service account id` value of the service
   account you just selected. (it should end with `gserviceaccount.com`) You will
   need this email address and the location of the private key file to properly
   configure this Vagrant plugin.
1. Add the SSH key you're going to use to GCE Metadata in `Compute` ->
   `Compute Engine` -> `Metadata` section of the console, `SSH Keys` tab. (Read
   the [SSH Support](https://github.com/mitchellh/vagrant-google#ssh-support)
   readme section for more information.)

   PS: if you don't have public key check it out [Generate SSH Key](https://cloud.google.com/compute/docs/instances/connecting-to-instance#generatesshkeypair)

1. In sum you should have three important info as below and add your public key
   into gce already.
 * google_project_id
 * google_client_email
 * google_json_key_location
 * added your new project-wide SSH key.

### Google Cloud Connection via Configuring Modules with secrets.py
 * If you don't want to specify above info in your Ansible code, you can create
   secrets.py into your $PYTHONPATH
1. Create a file secrets.py looking like following, and put it in some folder
   which is in your $PYTHONPATH:
```
GCE_PARAMS = ('i...@project.googleusercontent.com', '/path/to/project.json')
GCE_KEYWORD_PARAMS = {'project': 'project_id'}
```
1. Find out your $PYTHONPATH `python -c "import sys; print sys.path"`
 * for example : "/usr/local/lib/python2.7/dist-packages"
1. move your secrets.py to $PYTHONPATH `mv ./secrets.py /usr/local/lib/python2.7/dist-packages`

### Install apache-libcloud for ansible gce driver
Ansible contains modules for managing Google Compute Engine resources, including creating instances, controlling network access, working with persistent disks, and managing load balancers. Additionally, there is an inventory plugin that can automatically suck down all of your GCE instances into Ansible dynamic inventory, and create groups by tag and other properties.

1. The GCE modules all require the apache-libcloud module which you can install from pip: `sudo pip install apache-libcloud`

## Build and Test
### Boot Cent7 VM
1. `cd gce`
1. `ansible-playbook gce_create.yml`
1. find out external ip in Google Cloud Platform -> Compute Engine -> VM Instances
```
TASK [Wait for SSH to come up] *************************************************
[DEPRECATION WARNING]: Using bare variables is deprecated. Update your playbooks so that the environment value uses the full variable syntax
('{{gce.instance_data}}').
This feature will be removed in a future release. Deprecation warnings can be disabled by setting
deprecation_warnings=False in ansible.cfg.
ok: [127.0.0.1] => (item={u'status': u'RUNNING', u'network': u'default', u'zone': u'us-central1-a', u'tags': [], u'image': u'centos-7-v20160921', u'disks': [u'foo'], u'name': u'foo', u'public_ip': u'130.211.203.240', u'private_ip': u'10.128.0.2', u'machine_type': u'n1-standard-1', u'metadata': {}})
```

### Build SAIO
1. `cd ..` Back to root directory
1. `ansible-playbook site.yml -i "<gce external ip>,"`

#### GCE Build Output Screenshot
 * https://gist.github.com/chianingwang/9324671187713f41a0b4eee5c209def8

### SSH to the Box
1. `ssh <gce external ip>`

### Test Swift
1. `swift stat -v`
1. `echo 'this is test \n this is test \n this is test \n this is test \n this is test \n ' > test.txt`
1. `swift upload test_container test.txt`
1. `swift list `
1. `swift list test_container`

#### Double Check
1. `swift stat -v`

### Back to Local
1. `exit`

## Clean Up
### Wipe Out GCE VM
1. `cd gce`
1. `ansible-playbook gce_delete.yml`

SAIO Ansible playbook via Vagrant
=================================
## Preparation
This project assumes you have Virtualbox and Vagrant.

 * https://www.virtualbox.org/wiki/Downloads
 * http://www.vagrantup.com/downloads.html

- PS: I use VirtualBox 5.1.6 r110634 and Vagrant 1.8.5

## Build and Test
### Boot Cent7 and Build SAIO
1. `vagrant up`

### SSH to the Box
1. `vagrant ssh`

### Test Swift
1. `swift stat -v`
1. `echo 'this is test \n this is test \n this is test \n this is test \n this is test \n ' > test.txt`
1. `swift upload test_container test.txt`
1. `swift list `
1. `swift list test_container`

#### Double Check
1. `swift stat -v`

### Back to Local
1. `exit`

## Clean up
### Vagrant Destroy
1. `vagrant destroy`


SAIO ansible playbook via AWS ( EC2 )
=================================
## AWS Preparation
You use access keys to sign programmatic requests that you make to AWS if you use the AWS SDKs, REST, or Query APIs. The AWS SDKs use your access keys to sign requests for you, so that you don't have to handle the signing process.

### Boto AWS Module for ansible
All of the modules require and are tested against recent versions of boto. You’ll need this Python module installed on your control machine. Boto can be installed from your OS distribution or python’s.
1. `sudo pip install boto`

### AWS Access keys (access key ID and secret access key) Setup
Original Reference from AWS Doc: [Managing Access Keys for your AWS Account](http://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html)
#### via Security Credentials - Creating, Disabling, and Deleting Access Keys for your AWS Account
1. Log in with your AWS Account and go to
   [aws](https://aws.amazon.com/) and click on the
   `My Account` --> `AWS Management Console`.
1. In the upper right of the console, choose the account name or number and then choose Security Credentials.
1. On the AWS Security Credentials page, expand the Access Keys (Access Key ID and Secret Access Key) section.
1. Choose Create New Access Key. You can have a maximum of two access keys (active or inactive) at a time.
1. Choose Download Key File to save the access key ID and secret access key to a .csv file on your computer. After you close the dialog box, you can't retrieve this secret access key again.
1. To disable an access key, choose Make Inactive. AWS denies requests signed with inactive access keys. To re-enable the key, choose Make Active.
1. To delete an access key, choose Delete. To confirm that the access key was deleted, look for Deleted in the Status column.

 * Caution: Before you delete an access key, make sure it is no longer in use. You can't recover a deleted access key.
 * Other than Security Credential, you can create Access/Secrety Key via [IAM](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey)
 * PS: Temporary Access Key: You can use temporary access keys in less secure environments or distribute them to grant users temporary access to resources in your AWS account

### AWS Key pairs (SSH) Setup
For Amazon EC2, you use key pairs to access Amazon EC2 instances, such as when you use SSH to log in to a Linux instance. [Amazon EC2 Key Pairs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
#### [Creating Your Key Pair Using Amazon EC2](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair)
1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
1. In the navigation pane ( Left-hand side EC2 Dashboard Menu ), under **NETWORK & SECURITY**, choose **Key Pairs**.

 * Tip: The navigation pane is on the left side of the Amazon EC2 console. If you do not see the pane, it might be minimized; choose the arrow to expand the pane.

1. Choose **Create Key Pair**.
1. Enter a name for the new key pair in the **Key pair name** field of the **Create Key Pair** dialog box, and then choose **Create**.
1. The private key file is automatically downloaded by your browser ( <key pair name>.pem ). The base file name is the name you specified as the name of your key pair, and the file name extension is .pem. Save the private key file in a safe place.

 * Important: This is the only chance for you to save the private key file. You'll need to provide the name of your key pair when you launch an instance and the corresponding private key each time you connect to the instance.

1. If you will use an SSH client on a Mac or Linux computer to connect to your Linux instance, use the following command to set the permissions of your private key file so that only you can read it.
`$ chmod 400 my-key-pair.pem`

### Or you can import your key pair
#### [Importing Your Own Key Pair to Amazon EC2](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#how-to-generate-your-own-key-and-import-it-to-aws)
1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
1. In the navigation pane ( Left-hand side EC2 Dashboard Menu ), under **NETWORK & SECURITY**, choose **Key Pairs**.
 * Tip: The navigation pane is on the left side of the Amazon EC2 console. If you do not see the pane, it might be minimized; choose the arrow to expand the pane.
1. Choose **Import Key Pair**.
1. In the **Import Key Pair** dialog box, choose **Browse**, and select the public key file that you saved previously. Enter a name for the key pair in the Key pair name field, and choose **Import**.

### Sum of AWS Preparation
At this point you should have
 * Access Key ID:
 * Secret Access Key:
 * create or import SSH key
   * Create SSH ( Public Key in AWS and Download Private Key xxx.pem )
   * Import your own Public Key and keep your own private key in ~/.ssh/

## Build and Test
### Boot RHEL7 VM (AWS doesn't provide official CentOS-7)
1. `cd aws`
1. `ansible-playbook aws_create.yml`
1. find out external ip in AWS Management Console -> EC2 -> Select VM --> Description Tab Or from ansible output
```
TASK [Add all instance public IPs to host group] *******************************
[DEPRECATION WARNING]: Using bare variables is deprecated. Update your playbooks so that the environment value uses the full variable syntax
('{{ec2.instances}}').
This feature will be removed in a future release. Deprecation warnings can be disabled by setting deprecation_warnings=False in
ansible.cfg.
changed: [localhost] => (item={u'kernel': None, u'root_device_type': u'ebs', u'private_dns_name': u'ip-xxxx.us-west-1.compute.internal', u'public_ip': u'xxxx', u'private_ip': u'xxxx', u'id': u'i-5b67c4cc', u'ebs_optimized': False, u'state': u'running', u'virtualization_type': u'hvm', u'architecture': u'x86_64', u'ramdisk': None, u'block_device_mapping': {u'/dev/sda1': {u'status': u'attached', u'delete_on_termination': True, u'volume_id': u'vol-ac4c7902'}}, u'key_name': u'johnnywa', u'image_id': u'ami-d1315fb1', u'tenancy': u'default', u'groups': {u'sg-160e2572': u'default'}, u'public_dns_name': u'ec2-xxxx.us-west-1.compute.amazonaws.com', u'state_code': 16, u'tags': {u'Name': u'foo'}, u'placement': u'us-west-1a', u'ami_launch_index': u'0', u'dns_name': u'ec2-xxxx.us-west-1.compute.amazonaws.com', u'region': u'us-west-1', u'launch_time': u'2016-10-04T06:18:41.000Z', u'instance_type': u't2.medium', u'root_device_name': u'/dev/sda1', u'hypervisor': u'xen'})

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0
```

### Build SAIO
1. `cd ..` Back to root directory
1. `ansible-playbook site.yml -i "<gce external ip>,"`

#### AWS Build Output Screenshot
* https://gist.github.com/chianingwang/9324671187713f41a0b4eee5c209def8

### SSH to the Box
1. `ssh <aws external ip>`

### Test Swift
1. `swift stat -v`
1. `echo 'this is test \n this is test \n this is test \n this is test \n this is test \n ' > test.txt`
1. `swift upload test_container test.txt`
1. `swift list `
1. `swift list test_container`

#### Double Check
1. `swift stat -v`

### Back to Local
1. `exit`

## Clean Up
### Wipe Out GCE VM
1. `cd aws`
1. `ansible-playbook aws_delete.yml`
