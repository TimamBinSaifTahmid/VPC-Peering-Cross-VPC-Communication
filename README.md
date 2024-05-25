# VPC-Peering-Cross-VPC-Communication
Establishing and Connecting Private Subnets Across Separate VPCs Using AWS VPC Peering
# INTRODUCTION
In today’s interconnected digital landscape, the ability to efficiently set up, configure, and secure 
cloud infrastructure is a vital skill for any IT professional. Imagine you’re tasked with establishing 
communication between two servers, each isolated in its own virtual private cloud (VPC), while ensuring 
secure and seamless connectivity. This scenario is not just a theoretical exercise but a practical necessity 
in many enterprise environments, where cross-VPC communication underpins critical business operations

# Task Description:
![alt text](https://github.com/TimamBinSaifTahmid/VPC-Peering-Cross-VPC-Communication/blob/main/problem.png?raw=true)
We aim to create two VPCs, named VPC-1 and VPC-2. Within VPC-1, we will deploy an application server in a private subnet.
Similarly, VPC-2 will host an Nginx server, also situated in a private subnet. The primary objective is to establish a 
peering connection between the application server in VPC-1 and the Nginx server in VPC-2. This connection will enable the 
application server to send a telnet request to the private IP address of the Nginx server on port 80, thereby facilitating 
communication between the two servers.

# Step-by-Step Guide:
**Creating Two Separate VPC in AWS:**
Navigate to AWS and search for “VPC” using the search bar. The VPC dashboard will appear. Click on the “Create VPC” button 
located at the top right corner of the page. A page similar to the one shown below will then appear.

![alt text](https://github.com/TimamBinSaifTahmid/VPC-Peering-Cross-VPC-Communication/blob/main/create_vpc.png?raw=true)

Now, select the “VPC only” mode to customize the VPC settings manually. Assign a name for the VPC (vpc-1). For the IPv4 CIDR 
block, either manually input the IPv4 CIDR address or network address for the entire VPC, or choose to have AWS IPAM auto-allocate 
the IPv4 CIDR. Keep the default setting of “No IPv6 CIDR” as we do not require IPv6 for this case. Select the tenancy as “default.” 
Click the “Create VPC” button to finalize the creation of vpc-1. Repeat these steps to create vpc-2.

**Creating Public and Private Subnet Under Each VPC:**
For this task, we need to create both a public and a private subnet under each VPC. The public subnet is necessary to facilitate 
access and configuration of the private subnet within each VPC. From the AWS VPC dashboard, select “Subnets” and click the “Create Subnet” 
button located at the top right corner of the page. A page similar to the one shown below will then appear.

![alt text](https://github.com/TimamBinSaifTahmid/VPC-Peering-Cross-VPC-Communication/blob/main/create_subnet.png?raw=true)

Next, select the appropriate VPC ID (vpc-1) from the “VPC ID” section. In the subnet section, assign a suitable name (vpc-1-public-subnet).
Choose an availability zone close to your location, and try to select the same availability zone for each subnet to avoid potential issues.
Leave the IPv4 VPC CIDR block as default, as it will be automatically assigned based on the VPC selected in the VPC ID section. Assign an 
IPv4 subnet CIDR network address (10.1.1.0/24), which is smaller than the VPC network CIDR. Here, the /24 indicates that the first three octets
of the address represent the network, and the last octet represents the IPs, allowing for a total of 256 possible IP addresses within this subnet. 
Click the “Create Subnet” button to finalize.


Similarly, create the private subnet for vpc-1, as well as the public and private subnets for vpc-2.


A critical point to note is that for VPC peering to work successfully, the private subnet CIDR blocks of both VPCs must be different. If 
the private subnet CIDR blocks of both VPCs match, VPC peering will not function.


**Creating EC2 Servers of both VPC:**
For VPC-1, we need to create two EC2 instances: a bastion server and a private EC2 instance named “app server,” which will reside in the 
private subnet. Similarly, for VPC-2, we need to create two EC2 instances: a bastion server and a private Nginx server, both residing 
in the private subnet. This results in a total of four EC2 instances.


To begin, navigate to the search panel and search for “EC2.” This will bring up the EC2 dashboard. From the dashboard, select “Instances” a
nd click the “Launch Instance” button located at the top right corner. A page similar to the one shown below will then appear.

![alt text](https://github.com/TimamBinSaifTahmid/VPC-Peering-Cross-VPC-Communication/blob/main/name_tag.png?raw=true)

![alt text](https://github.com/TimamBinSaifTahmid/VPC-Peering-Cross-VPC-Communication/blob/main/net-setting.png?raw=true)

In the “Name and Tags” section, assign a meaningful name to the EC2 instance (e.g., vpc-1-bastion-server). In the “Application and OS Images” 
section, select an Ubuntu image and choose the free tier eligible AMI. For the instance type, keep the default selection of t2.micro as it is free tier eligible.


In the “Key Pair” section, click “Create new key pair,” select the RSA option, and then click the “Create key pair” button.


For the network configuration, click the “Edit” button and change the VPC from the default to the respective VPC (vpc-1). Select the appropriate
subnet (vpc-1-public-subnet). If you are creating an instance in the public subnet (vpc-1-public-subnet), enable “Auto-assign Public IP.” For 
instances in a private subnet (vpc-1-private-subnet), disable “Auto-assign Public IP.”


Keep the firewall settings as default, which includes SSH access to connect using the SSH protocol. Leave the volume settings as default and click the “Launch Instance” button.


Repeat these steps to create the remaining instances: the app server (in the private subnet of vpc-1), vpc-2-bastion-server (in the public subnet of vpc-2),
and the Nginx server (in the private subnet of vpc-2).

**Configure Route Tables and IGW to Connect with Public Bastion Servers:**

If we attempt to connect to the bastion servers in both VPCs, we may encounter an error indicating that the instance is not associated with a public subnet
or public IP addresses. Despite creating the EC2 bastion servers in public subnets, the issue arises because we did not add an Internet Gateway (IGW) to the 
route tables of the public subnets. Without the IGW, traffic cannot pass from the bastion server to the user’s public IP address.


To resolve this, go to the VPC dashboard, select “Internet Gateways,” and click the “Create internet gateway” button. Assign a name to the Internet Gateway 
and click “Create internet gateway.”

![alt text](https://github.com/TimamBinSaifTahmid/VPC-Peering-Cross-VPC-Communication/blob/main/internet_gateway.png?raw=true)

Next, we need to create route tables to associate the Internet Gateway with the public subnet. From the VPC dashboard, select “Route Tables” and click the “Create route table” 
button at the top right corner of the page. Assign a name to the route table (e.g., vpc-1-public-rt) and select the appropriate VPC.

![alt text](https://github.com/TimamBinSaifTahmid/VPC-Peering-Cross-VPC-Communication/blob/main/route_table.png?raw=true)

Similarly, we need to create the following route tables: a private route table under VPC-1 (vpc-1-private-rt), a public route table under VPC-2 (vpc-2-public-rt), and a private 
route table under VPC-2 (vpc-2-private-rt). After creating these route tables, select each route table, go to the “Subnet associations” section, and associate the appropriate 
subnet with the corresponding route table.

![alt text](https://github.com/TimamBinSaifTahmid/VPC-Peering-Cross-VPC-Communication/blob/main/subnet_asc.png?raw=true)

Now, select the public route table of VPC-1 (vpc-1-public-rt). In the “Routes” section, click “Edit routes.” Then, add a new route with the destination 0.0.0.0/0. In the “Target” 
section, choose the Internet Gateway option and select the Internet Gateway that was created earlier. Do the same for public route of VPC -2.

![alt text](https://github.com/TimamBinSaifTahmid/VPC-Peering-Cross-VPC-Communication/blob/main/edit_route.png?raw=true)

To allow SSH requests to reach the public bastion server, we add the route 0.0.0.0/0 in the route table. This route ensures that any IP address not covered 
by other routes in the table is directed through the Internet Gateway (IGW). This setup allows the bastion server to respond to SSH requests and send the response
back to the requester’s public IP via the IGW.


You can connect to the bastion server using SSH and the PEM key generated during the creation of the EC2 bastion instances. We need to use

```
chmod 400 my_key.pem
```
chmod 400 to change it permission to read only mode else it will not work. Use the following command to connect to the vpc-2 bastion server:

```
ssh -i my_key.pem ubuntu@<public ip of bastion server>
```


**Configure VPC -2 Nginx Server:**

To connect to the Nginx server, we must first connect through the bastion server using the PEM key. Therefore, we need to copy the PEM key to the bastion server.
This can be achieved using the following command:

```
scp -i my_key.pem my_key.pem ubuntu@<public ip of bastion server>:
```
After transferring mykey.pem to the bastion server, access the bastion server via SSH. Execute the ls command to confirm whether mykey.pem has been successfully transferred to the bastion server. 
Subsequently, utilize the provided command to establish a connection to the EC2 instance hosting the Nginx server.
```
ssh -i my_key.pem ubuntu@<private ip of nginx server>
```
After successfully logging into the Nginx server, attempts to update its repositories using “sudo apt update” will fail. This is because the Nginx server resides in a private subnet without 
a public IP address, hindering its communication with the outside world. To resolve this issue, we must create a NAT Gateway in the public subnet and add it to the route table of the private subnet.
This allows the private IP of the Nginx server to be translated to a public IP, enabling communication with the external network.


To proceed, navigate to the VPC dashboard and select “NAT Gateways.” Click on the “Create NAT Gateway” button located at the top right corner of the page.

![alt text](https://github.com/TimamBinSaifTahmid/VPC-Peering-Cross-VPC-Communication/blob/main/nat_gateway.png?raw=true)

To proceed, assign a name to the NAT Gateway and select the public subnet of VPC-2. Set the connectivity type to “public” and assign an Elastic IP to ensure 
a static IP address. Click the “Create NAT Gateway” button to finalize the setup.


Next, in the route table of VPC-2’s private subnet, add a route for destination 0.0.0.0/0 with the target set to the NAT Gateway created earlier. This route ensures 
that any requests not specified by other routes in the table will be routed through the NAT Gateway.


Now, reconnect to the Nginx server from the bastion server and execute the following command to install and run the Nginx server:

```
sudo apt install nginx
```

**Connect Two VPC Using Peering Connection of AWS:**

Now, connect to the bastion server of VPC-1 via SSH and establish a connection to the app server using the bastion server. Upon attempting to send a ping request 
from the app server of VPC-1 to the Nginx server of VPC-2, it will fail due to the absence of a peering connection between the two VPCs.


To rectify this, navigate to the VPC dashboard and select “Peering Connections.” Click on the “Create Peering Connection” button to initiate the process. You will be presented 
with a page similar to the one shown below.

![alt text](https://github.com/TimamBinSaifTahmid/VPC-Peering-Cross-VPC-Communication/blob/main/peer_con.png?raw=true)

To proceed, assign a name to the peering connection. In the “VPC ID” section, select the VPC from which you are initiating the connection, and specify the VPC ID of the other VPC.
Click the “Create Peering Connection” button to create the peering connection.


Next, modify the route tables of the private subnets. In the private route table of VPC-1, add a route for destination 10.2.0.0/16 and select the peering connection as the target.
Repeat this process for the private route table of VPC-2.


For the final step, configure the inbound rules of the security group associated with the Nginx EC2 instance. Navigate to the EC2 dashboard, select the Nginx EC2 instance, 
and access the security section. Choose the security group associated with the instance and click on the “Edit inbound rules” button. Add a new rule of type “Custom TCP” for 
port 80 and specify the source as the bastion IP address. This will allow ICMP packets for ping and TCP port 80 for the Nginx server.

![alt text](https://github.com/TimamBinSaifTahmid/VPC-Peering-Cross-VPC-Communication/blob/main/inbound_rule.png?raw=true)

Now if we connect to vpc-1 app server and write the below command we can see that the connection between vpc-1 private ec2 instace(app server) and vpc-2 private instace (nginx server) has created successfully

```
ping <private ip of nginx server>
curl <private ip of nginx server>:80
```
by using curl command we can see the html view of nginx welcome page.

**Conclusion:**
In conclusion, the successful establishment of seamless cross-VPC communication underscores the importance of robust infrastructure design and meticulous configuration in 
modern cloud environments. By leveraging AWS VPC peering, we have bridged the gap between separate VPCs, facilitating secure and efficient communication between private subnets. 
This achievement not only enhances operational efficiency but also reinforces the resilience and scalability of cloud-based architectures. As organizations continue to embrace cloud technology, 
the ability to navigate complex networking challenges and implement reliable inter-VPC connectivity remains a cornerstone of effective cloud management. Through this task, we have demonstrated 
the practical application of VPC peering, empowering businesses to build interconnected, resilient, and agile cloud infrastructures to support their evolving needs.
