# Ansible AWS & Docker Demo (WordPress) & Prometheus & ELK

## What this script does
- Setup a AWS VPC and EC2 Instance, and 2 Security Groups (Web and LB)
- Create Web Application and Load Balancer
- Configure Docker and run Docker in the created AMI Instance 

## Requirements
Things required to run the script:
- Ansible 2.8
- Python
- Boto 3
- Pip
- AWS Access and Secret Key

## Install
### Ansible 2.8

We will have get Ansible from ansible personal repository
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
sudo apt-add-repository "deb http://ppa.launchpad.net/ansible/ansible/ubuntu bionic main"
sudo apt install ansible
```

### Pip

```
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
sudo python get-pip.py
```

### Boto3

```
sudo pip install boto3
```

# Running Script

## Setting up AWS VPC, and EC2

Login to AWS Console Management - IAM , and create one programmatic access account `ansible` 
If you want to run this script you will be needing to add your AWS Access and Secret Key to `setup-aws.yml`

Should look something like this
```
---
- hosts: localhost
  vars:
    access_key: # Put Access Key Here
    secret_key: # Put Secret Key Here
  roles:
    - ec2
```

#### Running files to setup the AWS
Since we aren't connecting to a remote server this ansible file will run locally so all you need to do is run
```
ansible-playbook setup-aws.yml -vvvv 

```

After running successfully, this should create three files for you. First it will create `hosts.yml` in root directory of this project which has the Public IP of the instance created. Second a file called `dns.txt` this has the DNS name for the the Application Load Balancer that is created with this project. Finally it will create a folder in root directory of you machine called `.pems`  and there is created a private key  `demo-key.pem`, you will need to change permission for this file to be 400.

#### Running files to setup Docker
You can simply run the following commands to configure and setup Docker and relevant components in the EC2 instance
```
ansible-playbook configure-setup.yml -i hosts.yml --private-key ~/.pems/demo-key.pem -vvvv
ansible-playbook configure-prom.yml -i hosts.yml --private-key ~/.pems/demo-key.pem -vvvv
ansible-playbook configure-elk.yml -i hosts.yml --private-key ~/.pems/demo-key.pem -vvvv

```

the `configure-setup.yml` will install the WordPress docker instance in the running machine
the `configure-prom.yml` will install the Prometheus and Grafana docker instance in the running machine
the `configure-elk.yml` will install the ELK docker instance in the running machine.

Using the DNS name in `dns.txt` you can access the WordPress instance that is running in the machine


Using the `demo-key.pem` (export privately on your control machine and you can ssh into running machine - help troubleshooting) 
```
ssh -i "demo-key.pem" ubuntu@ec2-54-255-221-74.ap-southeast-1.compute.amazonaws.com

```

## Default Variables
You can overwrite these default variable by defining in `setup-aws.yml`
```
aws_region: ap-southeast-1
vpc_name: "My VPC"
vpc_cidr_block: "10.0.0.0/16"
vpc_public_subnet_1_cidr: "10.0.0.0/24"
vpc_public_subnet_2_cidr: "10.0.1.0/24"
instance_type: 't3.micro'
# https://cloud-images.ubuntu.com/locator/ec2/
ami_id: 'ami-068f0e11411e30b36'
instance_count: 1
# forward to lb on target_group_name (port 80) and target_group_monitor_name(port 300 - Grafana)
target_group_name: tgdemo
target_group_monitor_name: gfdemo

```
