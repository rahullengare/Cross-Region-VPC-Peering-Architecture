# Cross-Region-VPC-Peering-Architecture
Multi-region VPC Peering with EC2, NAT, IGW, scalable &amp; more secure cloud architecture.
## About Project:

This project demonstrates how to securely connect two AWS Virtual Private Clouds (VPCs) across different regions using **VPC Peering**.

- **VPC-A (N. Virginia)** hosts a proxy server in a public subnet and an app server in a private subnet.
- **VPC-B (Mumbai)** hosts a database server in a private subnet.
The architecture ensures secure, scalable communication between application and database layers using NAT, IGW, and custom route tables.

## Technologies Used:

- **AWS VPC** – Isolated cloud network to host resources securely
- **EC2** – Virtual servers to run applications and databases
- **Subnets** – Logical divisions of VPC for public/private access
- **Internet Gateway** – Enables internet access for public subnet
- **Route Tables** – Controls traffic flow within VPC
- **Security Groups** – Firewall rules for EC2 instances
- **SSH** – Secure protocol to access servers remotely
- **IAM (optional)** – Manages user permissions and roles in AWS

## Prerequisites:

- AWS Free Tier account
- Basic knowledge of networking IP, CIDR, VPCs, EC2, NAT, IGW, and peering.
- Familiarity with EC2 and AWS Console
- SSH key pair for EC2 login
- Optional: AWS CLI or Terraform for automation

---

## What is VPC?

A **Virtual Private Cloud** is a logically isolated section of AWS where you can launch resources in a defined network. You control IP ranges, subnets, route tables, and gateways.

## What is VPC Peering?

**VPC Peering** is a networking connection between two VPCs that enables you to route traffic using private IP addresses.
It allows resources in different VPCs to communicate as if they were in the same network — without using public internet.

## What is a Subnet?

A **Subnet** is a segment of a VPC’s IP range.

- **Public Subnet**: Connected to the internet via an Internet Gateway
- **Private Subnet**: Isolated, no direct internet access

## What is an Internet Gateway?

An **IGW** allows communication between your VPC and the internet. It’s attached to the VPC and used in route tables for public access.

## What is a Route Table?

A **Route Table** defines how traffic flows within your VPC.

- Public route table: routes 0.0.0.0/0 to IGW
- Private route table: no internet route (or via NAT if needed)

## What is a NAT Gateway?

A **NAT Gateway** allows instances in a private subnet to access the internet **without being exposed** to incoming traffic.
(Not used in this project, but useful for updates or outbound access)

## What is a Security Group?

A **Security Group** acts as a virtual firewall for EC2 instances.

- App SG: allows SSH, HTTP
- DB SG: allows SSH/MySQL only from App SG

---

## Step 1: Create a VPC

1. Go to AWS console → Open VPC Service 
2. Click on Create 1st VPC on N. Virginia region (us-east-1)
    - Select → VPC only
    - Name → **vpc-a**
    - **IPv4 CIDR → 10.0.0.0/16**
    - Click on Create VPC

![Project Screenshot](/images/vpc-a1.png)

3. Click on Create 2nd VPC on Mumbai region (ap-south-1) 
    - Select → VPC only
    - Name → **vpc-b**
    - **IPv4 CIDR → 172.17.0.0/16**
    - Click on Create VPC

![Project Screenshot](/images/vpc-b1.png)

## Step 2: Creating Subnet’s

1. Go to Subnet Option on N. Virginia 
2. Click on Create Subnet 
    - Select VPC → **vpc-a**

![Project Screenshot](/images/us-subnet-1.png)

- Create Subnet 1
    - Name → Public Subnet
    - Availability Zone → N. Virginia region (us-east-1)
    - IPv4 subnet CIDR block → 10.0.0.0/20

![Project Screenshot](/images/us-subnet-2.png)

- **Add new subnet**
- Create Subnet 2
    - Name → Private Subnet
    - Availability Zone → N. Virginia region (us-east-1)
    - IPv4 subnet CIDR block → 10.0.16.0/20

![Project Screenshot](/images/us-subnet-3.png)

3. Public Subnet to set public IP 
    - Select Public Subnet
    - Click Action → Edit subnet settings
    - Check-in → Auto-assign IP settings
    - Click Save Changes

![Project Screenshot](/images/us-edit-pub-subnet-1.png)
![Project Screenshot](/images/us-edit-pub-subnet-2.png)

4. Go to Subnet Option on Mumbai Region (ap-south-1)
5. Click on Create Subnet
    - Select VPC → **vpc-b**

![Project Screenshot](/images/ap-subnet-1.png)

- Create Subnet
    - Name → Private Subnet
    - Availability Zone → Asia Pacific (Mumbai) / aps1-az3 (ap-south-1a)
    - IPv4 subnet CIDR block → 172.17.0.0/20

![Project Screenshot](/images/ap-subnet-2.png)

## Step 3: Launch EC2 Instances

1. Go to AWS Console **on N. Virginia→ EC2** 
2. Click a Create Instance on us-east-1 
    - Name → **proxy_server**
    - Choose AMI → Amazon Linux.
    - Instance type → t2.micro
    - Key pair → pem-server-key
    - Network settings Edit
    - Select VPC → **vpc-a**
    - Subnet → Public-Subnet
    - Auto-assign public IP → **Enable**
    - security group → VPC-SG-1 (port: ssh-22, http-80, https-443, All ICMP - IPv4 )

![Project Screenshot](/images/us-app1.png)
![Project Screenshot](/images/us-app2.png)

3. Click on Create Instance on us-east-1
    - Name → **app_server**
    - Choose AMI → Amazon Linux.
    - Instance type → t2.micro
    - Key pair → pem-server-key
    - Network settings Edit
    - Select VPC → **vpc-a**
    - Subnet → Private-Subnet
    - Auto-assign public IP → **Disable**
    - security group → VPC-SG-2 (port: ssh-22, All ICMP - IPv4)
    - Click on Create

![Project Screenshot](/images/us-proxy1.png)
![Project Screenshot](/images/us-proxy2.png)
![Project Screenshot](/images/us-instance.png)

4. Go to AWS Console **on Mumbai→ EC2** 
5. Click on Create Instance on ap-south-1
    - Name → **db_server**
    - Choose AMI → Amazon Linux.
    - Instance type → t2.micro
    - Key pair → vpc-mumbai-pem-key
    - Network settings Edit
    - Select VPC → **vpc-b**
    - Subnet → Private-Subnet
    - Auto-assign public IP → **Disable**
    - security group → VPC-SG-2 (port: ssh-22,mysql-3306, All ICMP - IPv4)
    - Click on Create

![Project Screenshot](/images/ap-db1.png)
![Project Screenshot](/images/ap-db2.png)
![Project Screenshot](/images/ap-instance.png)

## Step 4: Create Internet Gateway

1. Go to VPC service of N. Virginia → Click on Internet Gateway Tab
2. Click on **Create internet gateway**
    - Name → IGW-PEER
    - Click on Create internet gateway
3. Select internet gateway

![Project Screenshot](/images/IGW-create1.png)
![Project Screenshot](/images/IGW-create2.png)

4. Click on Action → Attach to VPC 
    - Select VPC → **vpc-a**
    - click on Attach internet gateway

![Project Screenshot](/images/IGW-create3.png)

5. IGW - Entry on Route Table
    - Click on Route Table Tab
    - Select your Route Table (Public-RT)
    - Click To Routes → Edit Routes
    - Add Route → Select internet gateway → Select IGW-current your
    - Click on Save Changes
6. Your Internet Gateway is Created 

![Project Screenshot](/images/IGW-Public-RT1.png)
![Project Screenshot](/images/IGW-Public-RT2.png)

## Step 5: Create NAT Gateway

1. Click on the NAT Gateway Tab on N. Virginia  
2. Click Create NAT Gateway
    - NAT gateway settings
    - Name → NAT-PEER
    - Subnet → Public-Subnet
    - Elastic IP → Click on **Allocate Elastic IP**
    - Click on Create NAT Gateway

![Project Screenshot](/images/NAT-create1.png)

3. Create a Private Route Table on vpc-a 
    - Click on Route Table Tab
    - Click on Create Route Table
    - Name → Private-RT
    - Click on Save Changes

![Project Screenshot](/images/us-Private-RT.png)

4. NAT - Entry on Route Table
    - Click on Route Table Tab
    - Select your Route Table (Private-RT)
    - Click To Routes → Edit Routes
    - Add Route →Select NAT-current your
    - Click on Save Changes

![Project Screenshot](/images/us-NAT-entry.png)

5. Edit Subnet Associations
    - Click on Route Table Tab
    - Select your Route Table (Private-RT)
    - Click To Routes → Edit Subnet Associations   ****
    - Select Private-Subnet
    - Click on Save Changes

![Project Screenshot](/images/NAT-subnet-ass.png)

## Step 6: Connect via SSH

1. connect o proxy server

```bash
ssh -i "pem-server-key.pem" ec2-user@13.221.145.87
```

2. hostname command to set name

```bash
sudo hostnamectl hostname proxy-server
```

3. Re Connect 
4. ping commad

```bash
ping <ip-of-app-server>
#checkin only connecttion to proxy to app 
```

5. Copy key via scp command used to local machine to proxy-server 

```bash
 scp -i /c/ssh-keys/pem-server-key.pem /c/ssh-keys/pem-server-key.pem ec2-user@13.221.145.87:/home/ec2-user/
```

6. Connect **app-server via ssh**

```bash
ssh -i "pem-server-key.pem" ec2-user@10.0.29.219
```

7. hostname command to set name

```bash
sudo hostnamectl hostname app-server
```

8. re connect then check internet access or not 

```bash
#checkin only connection app to proxy
ping <ip-of-proxy-server>
```

![Project Screenshot](/images/ssh-ping-app.png)

9. Copy key via scp command used to local machine to proxy-server 

```bash
 scp -i /c/ssh-keys/vpc-mumbai-pem-key.pem /c/ssh-keys/vpc-mumbai-pem-key.pem ec2-user@13.221.145.87:/home/ec2-user/
```

10. Copy key via scp command used to proxy-server to app server  

```bash
 scp -i /c/ssh-keys/vpc-mumbai-pem-key.pem /c/ssh-keys/vpc-mumbai-pem-key.pem ec2-user@172.17.13.36:/home/ec2-user/
```

11. Connect **db-server via ssh**

```bash
ssh -i "vpc-mumbai-pem-key.pem" ec2-user@172.17.13.36
```

12. hostname command to set name

```bash
sudo hostnamectl hostname db-server
```

13. re connect then check internet access or not 

```bash
ping <ip-of-app-server>
#this not reuning beacuse they are in both different newtwoek 
#that problem solve by **VPC PEER**
```

![Project Screenshot](/images/ping-app-db.png)

## Step 6: VPC Peering

1. Go to N. Virginia on VPC service 
2. Click on  Peering Connection → Click on Create Peering Connection 
    - Name → PEER-1
    - **VPC ID (Requester) → vpc-a**
    - **Account →** My account
    - **Region →** Another Region
    - Select VPC region → Asia Pacific (Mumbai) (ap-south-1)
    - VPC ID → vpc-00c698bbbe32ff7bc
    - Click on Create Peering Connection
3. Your Request send to the vpc-b 
4. Go to Mumbai region on VPC service 
5. Click on  Peering Connection → Select Request
6. Click on Action → Select Accept Request 
7. Now your Peering of 2 VPC is Done

## Step 7: Edit Route & then Connect to db-server

1. Go to N. Virginia region → VPC service 
2. Click on Route Table → Select Private-RT 
3. Click on Edit Route → add new 
    - Click on Add Route
    - **Destination → 172.17.0.0/20** (this range is vpc-b under private-subnet)
    - Target → Peering Connection (select id)
    - click on Save Changes

![Project Screenshot](/images/PEER-route-1.png)

4. Go to Mumbai region → VPC service 
5. Click on Route Table → Select default-RT 
6. Click on Edit Route → add new 
    - Click on Add Route
    - **Destination → 10.0.16.0/20** (this range is vpc-a under private-subnet)
    - Target → Peering Connection (select id)
    - click on Save Changes

![Project Screenshot](/images/PEER-route-2.png)
![Project Screenshot](/images/PEER-route-3.png)

7. Connect **db-server via ssh**
8. re connect then check internet access or not 

```bash
ping 10.0.29.219
#ping db to app server 
```

9. Now your db-server also connect to the app server

![Project Screenshot](/images/ping-db-to-app.png)

## Step 7: Delete All instance, NAT, IGW, & VPC-PEER

1. Go to VPC service of **N. Virginia**
2. Click on Peeing Connection 
    - select peer to you want to delete
    - Click on the Action → Delete Peering Connection
    - type delete then deleted

![Project Screenshot](/images/PEER-delete-1.png)
![Project Screenshot](/images/PEER-delete-2.png)

3. Go to EC2 service of N. Virginia region   
4. Select instance you want to deleted 
5. click on the **Instance state →** Click on Terminate instance 

![Project Screenshot](/images/us-instance-delete.png)

6. Go to EC2 service of Mumbai region 
7. Select instance you want to deleted 
8. click on the **Instance state →** Click on Terminate instance 

![Project Screenshot](/images/ap-instance-delete.png)

9. Go to VPC service of N. Virginia ****
10. Click on NAT Gateway → Select NAT 
11. Click on Delete NAT Gateway 

![Project Screenshot](/images/NAT-delete.png)

12. Click on **Elastic IPs →** Select IP 
13. Click on Release Elastic IP addresses → Click Release 

![Project Screenshot](/images/NAT-release-ip.png)

14. Click on Internet Gateway → Select which are delete 
15. Then Click on Action → Detach 1st

![Project Screenshot](/images/detach-IGW.png)

16. Re-select then click Action → Delete Internet Gateway

![Project Screenshot](/images/detete-IGW.png)

17. Go to VPC service of **N. Virginia**→ Select VPC 
18. Click on Action → select **Delete VPC**
19. Go to VPC service of **Mumbai**→ Select VPC 
20. Click on Action → select **Delete VPC**
21. You deleted VPC then automatically 5 resources will also be deleted
    - security group of vpc, public & private Subnet, then also public & private Route table
22. type delete then Click on Delete

![Project Screenshot](/images/delete-vpc-a.png)
![Project Screenshot](/images/delete-vpc-b.png)
