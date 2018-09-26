ec2-vpc
=========

 Create a new VPC in an AWS region. Optionally, create a subnet and NAT gateway
in each AZ in that region, and create routing tables for them so internal
subnets can use them

Requirements
------------

 As this uses the AWS ansible modules, boto is required for this role.

Role Variables
--------------
  - vpc_action:    Set to one of create or remove.
  - create_nat_gw: Define (to true) if you want the role to create NAT gatways
                   for each AZ in the region. You must also pass in a dict of
                   availablity zones with the region as key. Check example
                   playbook below. This will also create the initial
                   presentation subnets, that will have direct access to the
                   internet via the IGW. You must also be creating an IGW.
                   Subnets will be /28, and 1 in each AZ.
  - dns_domain: set to a DNS domain name. This will update the DHCP options to
                the default DNS search domain for instances created in this VPC
  - vpc_name:  The name for the new VPC
  - new_vpc:   A dict containing information about the new VPC. This must
               contain the following keys:

  - new_vpc:
      cidr: ip/mask CIDR for new VPC
      tags: Dict of tags to apply to new VPC
      igw: true / false to create an Internet Gateway with the VPC
      region: aws region id

Dependencies
------------

 This does not depend on any other role.

Example Playbook
----------------

    - hosts: localhost
      vars:
        vpc_region: eu-west-2
        availability_zones:
          eu-west-2:
            - eu-west-2a
            - eu-west-2b
        vpc_cidr: 172.18.0.0/16
        vpc_action: create
        vpc_name: Management
        create_nat_gw: true
        new_vpc:
          cidr: "{{ vpc_cidr }}"
          tags:
            Name: "{{ vpc_name }}"
            CIDR: "{{ vpc_cidr }}"
            Region: "{{ vpc_region }}"
          igw: true
          region: "{{ vpc_region }}"
      roles:
    - role: ec2-vpc

License
-------

GPL

Author Information
------------------

Iain M. Conochie <iain@shihad.org>

Bugs
------------------

 When removing the VPC, I have had to add an ignore_errors line when removing
the routing tables, there does not seem to be a way to determine which routing
table is the main routing table associated with this VPC. Attempting to delete
this main routing table is what throws the error. This will be deleted when
the VPC is finally deleted.

 Deleting the VPC will most likely throw an error if there are instances still
running in any of the subnets of the VPC.
