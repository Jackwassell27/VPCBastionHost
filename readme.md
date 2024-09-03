# Virtual Private Cloud (VPC) With Bastion Host

This repository contains the instruction steps for setting up a Virtual Private Cloud (VPC) with Bastion Host with Secure SSH Access, below will be a table of contents with each section to help you with setting up your VPC with Bastion host.

## Architecture 

- Virtual Private Cloud
- Security Groups 
- Route Tables, Internet Gateways & NAT Gateways
- Elastic IP Allocation
- SSH Accesss Setup
- Instance Setup and Configuration

## Table of Contents
- [Architecture](#architecture)
- [Requirements](#requirements)
- [Setup Instructions](#setup-instructions)
    - [Unbuntu](#ubuntu)
    - [1. Create a VPC](#1-create-a-vpc)
    - [2. Subnets](#2-subnets)
    - [3. Internet Gateway](#3-internet-gateway-igw)
    - [4. Route Tables](#4-route-tables)
    - [5. NAT Gateway](#5-nat-gateway)


## Features
- Network Isolation
- Security Control
- Bastion Host
- Access Management
- Connectivity

## Requirements
- AWS Account: An Active AWS account with necessary permissions to create and manage VPC, EC2 Instances and related resources.
- Basic Knowledge: Familiarity with AWS services such as EC2, VPC and IAM
- Tools:
    - AWS Management Console: For creating and configuring resources via the web interface.
    - SSH Client: For connecting to the Bastion Host (eg. Ubuntu or PuTTY)

## Setup Instructions

### Ubuntu
_If you do not already have Ubuntu installed there will be instructions below before moving onto the AWS Section_

1. First open your windows start menu
2. Open PowerShell and run it as Administrator.
3. Once open, type `wsl --install`
4. Allow it to run it's Installation.
5. Once finished, close Powershell.
6. Next, head to the Microsoft Store.
7. Type in `Ubuntu` in the search bar.
8. Look for `Ubuntu 18.04.6 LTS` and click install.
9. After finishing the download open Ubuntu and follow it's initial setup.
10. To ensure it's updated put `sudo apt update` then `sudo apt full-grade -y`

### 1. Create a VPC

1. On the AWS Management Console, type in the top VPC and click on the service.
2. Click `Create VPC`
3. For the VPC Settings, click `VPC Only`
    - Give it a name. (eg. BastionHost-VPC)
    - Leave IPv4 CIDR manual input selected.
    - IPv4 CIDR, put `10.0.0.0/16`
    - Leave IPv6 CIDR block as default.
    - Leave Tenancy as default.
    - Click `Create VPC`

### 2. Subnets

For this section we will need to have two subnets, a public and a private

1. After creating the VPC, navigate to the left panel and click `Subnets`.
2. Click `Create Subnet`
_First we will do the public subnet._
    - Select the VPC you created.
    - Name your subnet (eg. PublicSubnet-BastionHost)
    - Select your Availability Zone
    - For IPv4 subnet CIDR block put `10.0.1.0/24`
3. After following the settings provided, click `Create Subnet`
4. After creating the subnet, select it via the tick box in the subnets list.
5. Select Actions at the top of the page and select `Edit subnet settings`
6. Tick the box for `Enable auto-assign public IPv4 address` and Save.

_Now let's make the private subnet_
1. Click `Create Subnet`
    - Select your VPC you created.
    - Name your subnet (eg. PrivateSubnet-BastionHost)
    - Select your Availability Zone
    - For IPv4 Subnet CIDR block put `10.0.2.0/24`
    _This CIDR Block must fall within the VPC range but must not overlap with any other subnet's CIDR._
3. After following the setup, click `Create Subnet` 
4. This time, do not enable auto-assign public IPv4 address.

### 3. Internet Gateway (IGW)

1. From the previous steps of setting up subnets, navigate to the left panel and click `Internet Gateways`
2. Click `Create Internet Gateway`
    - Give it a name (eg. Bastion-IGW)
3. Click `Create Internet Gateway`
4. Click on the Internet Gateway ID
    _It will look like this `igw-XXXXXXXXXXXXXXXXX` the X's will be a mix of numbers and letters._
5. Click `Actions` at the top right
6. Attatch to VPC and select your VPC you made.

### 4. Route Tables

1. Navigate to the left panel and click `Route Tables`
2. Create a route table.
    - Give it a name (eg. PublicRouteTable)
    - Select your VPC
3. Create the route table after following the settings.
4. Select the tick box on the Route Table you just made.
5. Navigate to the bottom table that opened and click `Routes`
6. Click `Edit Routes`
    - At the bottom left click `Add route` 
    - For it's Destination, put `0.0.0.0/0` 
    - For the target, put `Internet Gateway`
    - Then below select your Internet Gateway. _It will show you the name of it_
7. Save Changes.
8. While still selecting your public route table, click on `Subnet associations` tab.
9. Click `Edit subnet associations`
10. Select your PublicSubnet and click `Save associations`
11. Create another route table, but this time call it `PrivateRouteTable`
12. After creating a `PrivateRouteTable` this time do not give it a route to an Internet Gateway 
13. Select the tick box for your `PrivateRouteTable` and click `Subnet associations` 
14. Select your PrivateSubnet and click `Save Associations`

### 5. NAT Gateway
1. First Navigate to `Elastic IPs` under the Virtual Private Cloud panel.
    - Click `Allocate Elastic IP address` 
    - Click `Allocate` 
2. Navigate to `NAT Gateway` on the left panel.
    - Click `Create NAT Gateway`
    - Choose the `PublicSubnet` for the Subnet Field.
    - Select the Elastic IP that you just created. 
    - Click `Create NAT Gateway`
3. Update the Private Route Table to Use the NAT Gateway:
    - Go back to `Route Tables`
    - Select `PrivateRouteTable`
    - Click on the Routes tab, then click `Edit routes`
    - Add a route: 
        - Destination: `0.0.0.0/0` 
        - Target: Select the NAT Gateway you just created.
    - Click `Save Changes`

### 6. EC2 Instance with SSH Secure Access

To ensure you can securely access your EC2 instance via SSH, you should launch the instance in the public subnet of your VPC. The public subnet has a route to the internet via the Internet Gateway (IGW), which is necessary for SSH access from outside the VPC.

1. On AWS Management Console, Head to EC2.
2. Click on `Launch instance` 
    - Give it a name (eg. BastionHost)
    - For it's Application and OS Images select `Amazon Linux 2023 AMI` and ensure it is the free tier eligible.
    - For instance type select `t2.micro` for a small workload
    - Key pair, click `Create new key pair` 
        - Give it a name (eg. KeyPairSSH)
        - Key Pair type: `RSA`
        - Private key file format: `.pem`
    - Click `Create Key Pair`
        - This will download a .pem file to your computer, keep this safe.
3. Below on the Network Settings click `Edit`
    - First, click on VPC and select your VPC.
    - Next, ensure `Auto-assign public IP` is set to Enabled.
    - Move onto `Security Groups` below.
    - Choose `Create security group`
        - Give it a name (eg. PublicInstanceSG)
    - Below, make sure you allow SSH Inbound rule.
        - Set Type to `SSH`
        - Set Protocol to `TCP`
        - Set Port Range to `22`
        - Set Source type `My IP`
    - Double check all settings are correct.
    - Click `Launch` to start your instance.
4. On the left panel, click on `Instances`
_You will now notice you have BastionHost there with it's Instance state pending, allow this to finish_
    - In the meantime, the .pem file you just downloaded.
    - Navigate to a file explorer.
    - On the left panel you will see `Linux`
    - Head into that folder and open the `home` folder
    - Inside will be your user you created, open the folder.
    - Make a new folder either with the terminal or through windows directly, call it whatever. 
    - Place your .pem file inside of the new folder and close it.
5. Once you are on the `Instances` page and it is now running, select the tick box for BastionHost.
    - Once selected click `Connect` at the top of your screen.
    - Click on `SSH Client`
6. Open Ubuntu
    - Once open ensure your folder is accessible, you can do this by typing `ls`
    - If your folder is there do `cd <foldername>`
    - Once inside, we neelsd to ensure the file has permissions.
    - You can do this by typing `chmod 400 KeyPairSSH.pem`
    - Once finished, you can use the SSH command to connect to the instance.
    - You do this by typing `ssh -i KeyPairSSH.pem ec2-user@<Public-IP-Address>
    _The IP address will be found within your EC2 Instance when you click on the Instance ID_
    _Make sure to not include < > while putting the IP in place.
### Congratulations! You have successfully created a VPC with a Bastion Host.