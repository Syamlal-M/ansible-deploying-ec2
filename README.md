# ansible-deploying-ec2

   Ansible playbook for creating a security group and deploying instances.
   
   ## Prerequisite:
  - Install Ansible on an ec2 Instance and setup it as Ansible-master
  - Python boto library
  - Create an IAM Role with Policy AmazonEC2FullAccess and attach it to the Ansible master instance
  
  ### Below is a simple playbook to deploy ec2 instances. The problem of the below playbook is, it will deploy instances again if you run the playbook multiple times.
  
  ```
  ---
 - name: "ec2 instance creation"
   hosts: localhost
   tasks:
     - name: "Launching ec2 Instance"
       ec2:
        instance_type: t2.micro
        key_name: loadbalancer
        image: ami-0cb0e70f44e1a4bb5
        region: ap-south-1
        group: ansible-sg
        count: 1
        vpc_subnet_id: subnet-30820a7c
        wait: yes
        assign_public_ip: yes
```
      
      
 ### An improved version of the playbook which creates a security group and deploy instances using that security group is given below.

   We can control the number of instances using the exact_count parameter. Here it is set as 2 and no matter how many times you run the
   playbook there will only be two instances running at same time. That means if we turn off one of the instance and run the playbook again,
   it will deploy a new ec2-instance using the instance tags given. This will make sure that the exact_count number of instances are currently
   running(Basis of given instance tag).
    
```
---
 - name: "Deploying ec2 instances"
   hosts: localhost

   vars:
     instance_type: t2.micro
     security_group: ansible-sg
     image: ami-0cb0e70f44e1a4bb5
     region: ap-south-1
     keypair: loadbalancer
     exact_count: 2
     subnet: subnet-30820a7c

   tasks:
     - name: Create a security group
       local_action:
         module: ec2_group
         name: "{{ security_group }}"
         description: Ansible-webserver
         region: "{{ region }}"
         rules:
           - proto: tcp
             from_port: 22
             to_port: 22
             cidr_ip: 0.0.0.0/0
           - proto: tcp
             from_port: 80
             to_port: 80
             cidr_ip: 0.0.0.0/0
           - proto: tcp
             from_port: 443
             to_port: 443
             cidr_ip: 0.0.0.0/0
         rules_egress:
           - proto: all
             cidr_ip: 0.0.0.0/0

     - name: "Launching ec2 Instance"
       ec2:
         instance_type: "{{instance_type}}"
         key_name: "{{keypair}}"
         image: "{{image}}"
         region: "{{region}}"
         group: "{{security_group}}"
         vpc_subnet_id: "{{subnet}}"
         wait: yes
         count_tag:
           Name: Ansible-slave
         instance_tags:
           Name: Ansible-slave
         exact_count: "{{exact_count}}"
```

### Variables

```
 instance_type: t2.micro # Change the instance type according to your requirement
  security_group: ansible-sg  # Select Security group
  image: ami-0cb0e70f44e1a4bb5  # Change it to your need(Currently using Amazon Linux)
  region: ap-south-1 # Select Region
  keypair: loadbalancer # Select the key-pair which to be used
  exact_count:2  # Count of the instances, you can deploy more than 1 instances
  subnet: subnet-30820a7c  # VPC subnet
  ```
