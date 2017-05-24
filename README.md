# Run and destroy EC2 instance quicly

Script used to quicly create AWS VPC and launch there instance. Perfect when you need
start temporary Instance for tests or some work.

### Usage

Run `ec2_manage` script without arguments to see usage info.

### Configuring

Export AWS credentials

```
export AWS_ACCESS_KEY_ID="YOUAWSACCESSKEY"
export AWS_SECRET_ACCESS_KEY="YoUrSecRetAcCesKeyHeRe"
export AWS_DEFAULT_REGION="us-east-1"
```

Edit these fields in head of script before you create AWS resources:

- `aws_tag='temp-node'` - used to tagg all AWS resources created by script
- `userdata='user-data-file'` - location of user-data script running on first boot
- `ec2type='t2.small'` - EC2 type or running instance

### Examples

See usage info

```
$ ./ec2_manage

=== Dev AWS infra and Instance ===

Usage: ec2_manage <command>

Commands:

    help        display this help and exit

    noderun     launch  dev node
    nodedel     remove  dev node
    status      check node running
    ssh         ssh into running node

```

Create VPC and launch there EC2 instance

```
$ ./ec2_manage noderun

=== Dev AWS infra and Instance ===

Checking utils
Getting data about VPC
Getting data about instance
Adding AWS resources
Creating VPC
Tagging VPC
Creating Subnet
Modify Subnet for public IPs
Tagging Subnet
Creating  Internet Gateway
Tagging Gateway
Attaching Gateway to VPC
Obtaining Default Route table Id
Tagging Route Table
Adding default route
Attaching Subnet to Route Table
Creating Security Group
Tagging group
Adding open rule to group
Creating KeyPair
Launching t2.small Instance on AWS
Obtaining AMI if for Ubuntu 16.04
Checking userdata file present
Running instance
Tagging instance
Waiting while instance initalized
..........
Waitng for node
Checking ssh connection
.......
Connected
Waiting while orchestration finished
......................
Instance ready!
To login: ssh -i temp-node.pem ubuntu@54.194.16.12
```

Getting status of AWS resources

```
$ ./ec2_manage status

=== Dev AWS infra and Instance ===

Checking utils
Getting data about VPC
Getting data about instance
AWS VPC and correspoinding resoures created
Instance started
How to login: ssh -i temp-node.pem ubuntu@54.152.192.198
```

SSH into running instance

```
$ ./ec2_manage ssh

=== Dev AWS infra and Instance ===

Checking utils
Getting data about VPC
Getting data about instance
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-66-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

26 packages can be updated.
18 updates are security updates.


*** System restart required ***
Last login: Tue Mar 28 11:59:59 2017 from 46.4.69.7
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

ubuntu@ip-10-222-222-123:~$
```

Destroy Instance and AWS VPC

```
$ ./ec2_manage nodedel

=== Dev AWS infra and Instance ===

Checking utils
Getting data about VPC
Getting data about instance
Deleting Instance
Waiting while Instance gone
..........................................
Instance removed
Removing AWS resources
Deleting Security Group
Deleting Subnet
Detaching Internet Gateway
Deleting Internet Gateway
Deleting VPC
Deleting KeyPair
AWS resources removed
```

### User data scripts

Basic user-data script resides in root of repo: [user-data-file](user-data-file). This script install docker engine on Instance.
There are several examples of user-data script to use:

- [userdata.examples/user-data-file.cuda](userdata.examples/user-data-file.cuda) - install Nvidia driver, docker, nvidia-docker plugin (EC2 type must be g2. or p2.)
- [userdata.examples/user-data-file.mongo](userdata.examples/user-data-file.mongo) - install docker, run MongoDB server, configure authentication
- [userdata.examples/user-data-file.k8s_single](userdata.examples/user-data-file.k8s_single) - install single-node [Kubernetes](https://kubernetes.io/) cluster with components:
    - [flannel](https://kubernetes.io/docs/concepts/cluster-administration/addons/) (pod network addon)
    - standalone [heapster](https://github.com/kubernetes/heapster)
    - [nginx-ingresss-controller](https://github.com/kubernetes/ingress/tree/master/controllers/nginx) (with `.spec.HostNetwork: 'true'` setting) - server requests on port 80 of node
    - [kubernetes-dashboard](https://github.com/kubernetes/dashboard) (accesible via `http://<node_public_ip_address>` with default credentials `login: admin, password: single`)


### Notes

- Security Group created for Instance has rule to open all ports and all protocols to Internet
- On `noderun` script creates ssh pirivate key file placed next to script
- On `nodedel` ssh key file also deleted
- AWS resources script creates/removes:
  - AWS VPC with CIDR 10.222.0.0/16
  - Subnet with CIDR 10.222.222.0/24
  - Route Table
  - Association of Subnet with Route Table
  - Internet Gateway
  - Association Internet Gateway with VPC
  - Default route for Subnet to Internet Gateway
  - Security Group
  - Rule for Security group (simple rule full open to World)
  - KeyaPair and private SSH key (stored locally)
  - EC2 Instance with public IP address (see IP on `ec2_manage status`)
