# CONFIGURATION MANAGEMENT WITH ANSIBLE

## INTRODUCTION TO ANSIBLE

Ansible is an open-source automation tool used for configuration management, application deployment, infrastructure provisioning, and orchestration.

It allows you to automate tasks on multiple servers (or repetitive tasks) at the same time from a single machine called the control node.

Tasks are defined in form of yaml files and can be reused.

Ansible is agentless.

Ansible operates using modules - they get pushed to remote server and then removed when the execution is done. Modules are very granular and specific.

File that has tasks defined that are run on specific hosts as specific user is called a playbook (play).

Alternatives?

- Puppet
- Chef

## INSTALL ANSIBLE

Ansible should be installed on control node - but you can also have it on your laptop.

## SETUP MANAGED SERVER TO CONFIGURE WITH ANSIBLE

Hosts managed by ansible need to have python installed - that's the requirement.

## ANSIBLE INVENTORY AND ANSIBLE AD-HOC COMMANDS

Ansible ad-hoc commands are run as follows:

```
$ ansible [pattern - host/group] -m  [module] -a "[module options]"
```

## CONFIGURE AWS EC2 SERVER WITH ANSIBLE

Useful configuration options for EC2 host groups:

```
[ec2:vars]
ansible_ssh_private_key_file=~/.ssh/aws-ansible
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python3.9
```

## MANAGING HOST KEY CHECKING AND SSH KEYS

It depends how long the machine is going to live - if it is long-lived then key checking and verification can be done from control node (once).

```
$ ssh-keyscan -H x.x.x.x >> ~/.ssh/known_hosts
```

To copy default public key to remote server:

```
$ ssh-copy-id root@x.x.x.x
```

For ephemeral infrastructure - less secure but still used is to define in ~/.ansible.cfg (or other cfg location):

```
[default]
host_key_ckecing = False
```

## INTRODUCTION TO PLAYBOOKS

Installing specific version of a package using ansible:

```
  - name: install nginx server
    apt:
      name: nginx=1.24.0-2ubuntu7.6
      state: present
```

Ansible is **idempotent**.

## MODULES AND COLLECTIONS IN ANSIBLE

An Ansible module is a small program that performs a specific task on a managed host.

A collection is a bundle of Ansible content (modules, plugins etc.) distributed together. It can be released and installed independent of other collections.

Ansible Galaxy is a marketplace/library where you can download ready-made Ansible roles and collections.

To get list of collections:

```
$  ansible-galaxy collection list
```

To upgrade specific ansible collection:

```
ansible-galaxy collection install amazon.aws --upgrade
```

In Ansible, a namespace is the top-level identifier used to group and uniquely identify collections and their content. It is advised to use namespaces when referencing modules.

## PROJECT: DEPLOY NODEJS APPLICATION PART 1-3:

To pack nodejs app package directory use: `$ npm pack`

To run the nodejs app on the server we need to run it asynchronously:
```
    async: 1000
    poll: 0
```

To return values of modules we use **register** keyword to register it into a variable.

```
    - name: ensure app is running
      shell: ps aux | grep node
      register: app_status

    - debug: msg={{ app_status }}
```


Modules such as **command** or **shell** don't have state management - it needs to be implemented using conditionals.

To execute playbook as different user:

```
  become: True
  become_user: nodeuser
```

## ANSIBLE VARIABLES - MAKE YOUR PLAYBOOK CUSTOMIZABLE

Tho set variable value we use **register** keyword:

```
      register: user_creation_result
    - debug: msg={{ user_creation_result }}
```

When using variables within {{ }} right after : we need to double quote the variable name for it not to be confused with YAML dictionary.

```
src: "{{ node_file_location }}"
```

To set variable value we use **vars** keyword (they can have multiple scopes):

```
  vars:
    archive_path: "~/Downloads/nodejs-app"
    version: 1.0.0
    destination_path: /home/nodeuser
```

Or as an argument when calling ansible-playbook:

```
$ ansible-playbook ... -e "version=1.0.0 archive_path=~/Downloads/nodejs-app"
```

Or include them from separate file such as:

```
  vars-files:
    project-vars.yaml
```

## PROJECT: DEPLOY NEXUS PART 1-2

Installing net-tools is not needed in that case - Ubuntu comes with "ss" which even takes the same command switches as netstat and can be used to diagnose potential problems just like netstat did.

Since I'm deploying Nexus in its later version - 3.88 setting run_as_user is not done in nexus.rc anymore. It's done in nexus file which is actually a script - that's why I used **lineinfile** module:

```
    - name: set run_as_user for nexus
      lineinfile:
        path: /opt/nexus/bin/nexus
        regexp: '^#?\s*run_as_user='
        line: 'run_as_user="nexus"'
```

## ANSIBLE CONFIGURATION - DEFAULT INVENTORY FILE

To set default inventory file in ansible.cfg:

```
inventory = hosts
```

## PROJECT: RUN DOCKER APPLICATIONS PART 1-2

After adding user to the group to reconnect and refresh user environment:

```
    - name: reconnect to server session
      meta: reset_connection
```
To prompt for variable value:

```
  vars_prompt:
    - name: docker_password
      prompt: enter password for docker registry
```

## PROJECT: TERRAFORM AND ANSIBLE

It is possible to hand over job straight from Terraform to Ansible.

To use ansible-playbook from TF file we go this way:

```
  provisioner "local-exec" {
    working_dir = "/Users/dkolowski/Documents/git/personal/ansible"
    command = "ansible-playbook --inventory ${self.public_ip}, --private-key ${var.ssh_key_private} --user ec2-user  deploy-docker-new-user.yaml"
    }
```
When using **--inventory** to pass IP addresses of hosts instead of the classic hosts file we have to put a comma after the IPs.


when referencing "dynamic" IP address in ansible playbook that is going to be executed by TF it's best to set hosts to "all" in playbook file.

Another way to execute something straight from Terraform is to use **null resource**.

```
resource "null_resource" "configure_Server" {
  trigger = {
    trigger = aws_instance.myapp-server.public_ip
  }

  provisioner "local-exec" {
      working_dir = "/Users/dkolowski/Documents/git/personal/ansible"
      command = "ansible-playbook --inventory ${aws_instance.myapp-server.public_ip}, --private-key ${var.ssh_key_private} --user ec2-user deploy-docker-new-user.yml"
    }
}
```

Public IP in this case is a list so it would work with multiple IPs as well.

## DYNAMIC INVENTORY FOR EC2 SERVERS

There are two ways to handle dynamic inventory in ansible:

1. Dynamic inventory plugins (YAML)
2. Dynamic inventory scripts (PYTHON)

Ansible recommends first approach. To check what plugins are available:

`$ ansible-doc -t inventory -l`

Some of them may have requirements to be met on local controller.

For AWS we use **aws_ec2** plugin. We need to enable it in ansible.cfg file and create **inverntory_aws_ec2.yaml** file to configure the plugin.

To check if the dynamic inventory is working:

`$ ansible-inventory -i inventory_aws_ec2.yaml --list`

Or to display just hosts:

`$ ansible-inventory -i inventory_aws_ec2.yaml --graph`

the catch is that by default we are given list of EC2s with private DNS addresses. This comes from our initial Terraform file that was missing an option:

```
enable_dns_hostnames = true
```

To filter hosts to the ones that you actually need:

```
filters:
  tag:Name: dev*
  instance-state-name: running
```

Another way to group servers:

```
keyed_groups:
  - key: tags
    prefix: tag
  - key: instance_type
    prefix: instance_type
```

## PROJECT: DEPLOYING APPLICATION IN K8S

1. Set the K8S cluster up using TF from previous modules.
2. Configure it using kubernetes.core.k8s module in ansible.

After creating EKS cluster - to update context with kubeconfig in local dir:

```
$ aws eks update-kubeconfig --region eu-central-1 --name myapp-eks-cluster --kubeconfig ./kubeconfig_myapp-eks-cluster
```

Kubeconfig context would be needed in tasks with k8s module!

Instead of specifying kubeconfig attribute for every task you can set K8S_AUTH_KUBECONFIG env variable and ansible will use that.

```
$ export K8S_AUTH_KUBECONFIG=/path/to/kubeconfig
```

## PROJECT: RUN ANSIBLE FROM JENKINS PIPELINE PART 1-3

The aim of this project is to create separate ansible control node that would run playbook triggered by Jenkins.

Ansible node needs:

1. boto3 and botocore packages (Ubuntu/Debian)
2. .aws/credentials directory with credentials inside - needed for dynamic inventory for EC2

Jenkins will copy all needed files (ansible.cfg, inventory, playbook) to the control node.

To convert pem file to output format supported by Jenkins:

`$ ssh-keygen -p -f .ssh/id_rsa -m pem -P "" -N ""`

To copy private SSH key from Jenkins credentials to remote file:

```
withCredentials([sshUserPrivateKey(credentialsId: 'ec2-server-key', keyFileVariable: 'keyfile', usernameVariable: 'user')])
  sh 'scp $keyfile root@192.168.0.78:/root/ssh-key.pem'
```

To execute command in an CI/CD pipeline in Jenkins we need to install a plugin - **SSH Pipeline Steps**.

It needs to configure SSH connection in first step - where, who and how.

## ANSIBLE ROLES - MAKE YOUR ANSIBLE CONTENT MORE REUSABLE AND MODULAR

In Ansible, roles are a way to organize and structure your automation code so it’s reusable, maintainable, and scalable.

Roles have standard file structure.

When defining values for vars you have to be aware of variable precedence.
