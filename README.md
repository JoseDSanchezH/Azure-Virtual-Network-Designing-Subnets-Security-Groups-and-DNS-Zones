# Azure-Virtual-Network-Designing-Subnets-Security-Groups-and-DNS-Zones

## Objective
In this lab, I designed a scalable, secure, and organized virtual network in Azure, just like a real company would when moving to the cloud or expanding infrastructure for the company. The fictional or global organization JoseCyber has existing workloads and users but expects to grow, which means more departments, applications, and devices in the future are going to be needed to configure and implement. In order to prepare for that, the company needs a networking structure that prevents IP conflicts, supports clear segmentation, and can scale up easily without redesigning everything later. For this project, I'm creating 2V Nets, one for core services and one for manufacturing.

Task 1: Create a virtual network with subnets using the portal.

Task 2: Create a virtual network and subnets using a template.

Task 3: Create and configure communication between an Application Security Group and a Network Security Group.

Task 4: Configure public and private Azure DNS zones.


Task 1



I went to my Azure portal and searched for virtual Networks. I clicked on Create and completed the Basic Tab with the following: 



Resource Group: JoseCyberHomeLabs

Name: JoseCyberHL

Region: US West 

<img width="820" height="423" alt="image (2)" src="https://github.com/user-attachments/assets/a54a6669-b6f5-4dc0-aebf-a175d95bdfe5" />


1) I then changed the IP Addresses tab and changed the IP Address to 10.200.0.0/16
2) I then selected + Add a subnet. 

Subnet: SharedServicesSubnet 

Options: Subnet name ( SharedServicesSubnet)

Starting Address: 10.20.10.0

Size: /24
<img width="614" height="224" alt="image" src="https://github.com/user-attachments/assets/b81d69ea-9f4e-4a80-b4fa-a016771a04ce" />

<img width="592" height="265" alt="image (1)" src="https://github.com/user-attachments/assets/972e4a4c-65e8-4f94-89c5-9bc994bc352c" />


Subnet: DatabaseSubnet

Options: Subnet name (DatabaseSubnet) 

Starting Address: 10.20.20.0

Size: /24



Task 2:

Now I need to create a manufacturing VNet virtual network and associate it with my subnets. The organization Jose Cyber anticipates growth in the manufacturing offices, so the subnets are sized for that growth. I will need to use the template that I've already created for my core services VNet.

I located the template.json file and exported it to my downloads folder.
I then edited the file using Visual Studio Code to change all occurrences below.

1) replaced all occurrences of "JoseCyberHL" with "JoseCyberManVnet"

2) replaced all occurrences of "10.20.0.0" with "10.30.0.0."

3)Change all occurrences of SharedServicesSubnet to SensorSubnet1.

4) Change all occurrences of 10.20.10.0/24 to 10.30.20.0/24.

5) Change all occurrences of DatabaseSubnet to SensorSubnet2.

6) Change all occurrences of 10.20.20.0/24 to 10.30.21.0/24.

<img width="538" height="344" alt="image (3)" src="https://github.com/user-attachments/assets/7f4fd558-0272-4fcf-9b5c-8312cd027b99" />





Back to the Portal on Azure, I need to deploy the custom template I worked on. Click on "Build your own Template in the editor." 

<img width="802" height="379" alt="image (4)" src="https://github.com/user-attachments/assets/8b85452d-4bd4-4126-b78e-8ac55bd08bbf" />


I click on Load file, I then select the Template.json File with my Manufacturing changes, then select Save. 
<img width="627" height="295" alt="image (5)" src="https://github.com/user-attachments/assets/a26ee620-35d7-4267-a518-3675aa7c9f4e" />


I ran into an error. 
<img width="707" height="316" alt="image (6)" src="https://github.com/user-attachments/assets/261ba0da-26a3-44c6-9582-a45e93896410" />


 "code": "InvalidTemplate",

  "message": "Deployment template language expression evaluation failed: 'The language expression 'resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworks_JoseCyberManVnet_name'), 'SensorSubnet1)' is not valid: the string constant at position '109' is not terminated. String constants must be enclosed within a pair of single quotes. Please see https://aka.ms/arm-functions for usage details."

I see that I erased the quotation mark at the end of the syntax in Python. I was able to fix it by going back into the template.json and correcting it. 

"id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworks_JoseCyberManVnet_name'), 'SensorSubnet1')]",



Let's review and create the deployment. 

<img width="721" height="276" alt="image (7)" src="https://github.com/user-attachments/assets/88a1d024-a1f1-468b-b88c-06d2557a354a" />








Task 3: In this task, I need to create an application security group and a network security group. An application security group is like a tag for network interfaces, such as VMs, that group workloads by their function, not by IP addresses. Let's create two groups. 

asg-web for Web Servers
asg-db for Databases

When I make the rules for my web servers, I don't need to remember all IP addresses; I can just make the rules for the group name.


Network security groups are similar to a firewall that protects my virtual network, VNet, or subnet. This controls the traffic that can come in and what can go out.

I will be creating an inbound rule allowing ASG traffic to the web group on port 80, which is HTTP, and port 443, HTTPS. Because those ports are used for web traffic only, the web servers should be able to reach the protected subnet no one else.

I will also be adding an outbound rule denying Internet access. I do not want anything in this subnet to go out to the Internet, which means I want my servers safe from downloading bad software by mistake, sending sensitive data out, and talking to anything outside of my secure network.


In the portal, I need to search for application security groups. Then I would have to create and provide the basic information, which is down below. 

Setting: Value
Subscription:	My subscription
Resource group:	JoseCyberHomeLabs
Name:	asg-web
Region:	West US

<img width="554" height="258" alt="image (8)" src="https://github.com/user-attachments/assets/b22003a5-9c2e-4f29-849d-cc0d86e72155" />
<img width="534" height="269" alt="image (9)" src="https://github.com/user-attachments/assets/848ce888-068f-473a-a1eb-e92bb34814c4" />

<img width="712" height="178" alt="image (10)" src="https://github.com/user-attachments/assets/7ef899ff-d4b4-46c3-b69f-2ce0f3e0af37" />






Now, following the same steps I took, I need to create a network security group and associate it with JoseCyberHomelabs with the settings below.
Setting	Value
Subscription	My subscription
Resource group: JoseCyberHomeLabs
Name	myNSGSecure
Region	East US

<img width="754" height="249" alt="image (11)" src="https://github.com/user-attachments/assets/8c040641-9e23-4510-8da0-ec9640cce8b8" />
<img width="678" height="290" alt="image (12)" src="https://github.com/user-attachments/assets/2d7b7c06-e67c-4012-9755-7a7c9ee10422" />


I will need to associate it with my subnet. I will need to go to the resource --> Settings --> Subnets --> Associate. 

<img width="577" height="193" alt="image (13)" src="https://github.com/user-attachments/assets/f2d8cb3c-1422-42d1-a344-be994bb02d0e" />
<img width="567" height="245" alt="image (14)" src="https://github.com/user-attachments/assets/f4e7917a-4701-4040-a8ef-716096241e5a" />
<img width="562" height="184" alt="image (15)" src="https://github.com/user-attachments/assets/c73d6429-65ef-411c-9fab-b213daf4b87e" />
<img width="562" height="184" alt="image (16)" src="https://github.com/user-attachments/assets/dc54b1de-c1b3-4238-b5ef-b82666406114" />









Now, in my Inbound Security rule, I can add a new rule to allow ASG Traffic. 

Setting	Value
Source	Application security group
Source application security groups	asg-web
Source port ranges	*
Destination	Any
Service	Custom (notice your other choices)
Destination port ranges	80,443
Protocol	TCP
Action	Allow
Priority	100
Name	AllowASG
<img width="446" height="676" alt="image (17)" src="https://github.com/user-attachments/assets/da1747aa-a222-47c2-9d8d-b00c3eec23a0" />

Great, now I want to configure my outbound NSG rule that denies internet access. 
Setting	Value
Source	Any
Source port ranges	*
Destination	Service tag
Destination service tag	Internet
Service	Custom
Destination port ranges	*
Protocol	Any
Action	Deny
Priority	4096
Name	DenyInternetOutbound

<img width="742" height="407" alt="image (18)" src="https://github.com/user-attachments/assets/096b6960-0cae-4e51-9c0e-cca1e6456d6d" />

<img width="1600" height="397" alt="image (19)" src="https://github.com/user-attachments/assets/7920b871-a397-402a-9fc0-7dd7da9019fe" />





Task 4: We actually created this DNS Zone previously in another lab project. Let's do it once more. Property	Value
Subscription	My subscription
Resource group	JoseCyberHomeLabs
Name	JoseCyber.blog
Region	West US

<img width="445" height="357" alt="image (20)" src="https://github.com/user-attachments/assets/e1e8fdfc-346b-4bad-af58-f4af6760a736" />


Review and Create, I then went to "go to resource."

<img width="583" height="173" alt="image (21)" src="https://github.com/user-attachments/assets/308ad89f-122a-4e19-b18c-e3fe174782f6" />




I then went to Record Sets and created a record set. I then configured the settings as seen below. 

Name:	www
Type:	A
TTL:	1
IP address:  10.1.1.4
<img width="1600" height="773" alt="image (22)" src="https://github.com/user-attachments/assets/fdc719fa-7e68-4c62-bbde-cf51ecdd0ca8" />


I then created a Private DNS Zone. 
In the portal, I searched for and selected "Private DNS Zones. "
I then selected + Create
And input the information as follows: 
Subscription	My subscription
Resource group	JoseCyberHomeLabs
Name	private.josecyber.blog
Region	West US

<img width="1090" height="666" alt="image (23)" src="https://github.com/user-attachments/assets/a1550c12-a1e6-4974-a597-bcffadc4c9f7" />



I then went to the resource page and clicked on DNS Management --> Recordsets --> +Add. 
<img width="1289" height="560" alt="image (24)" src="https://github.com/user-attachments/assets/a229ecc9-41d3-4a02-9bb0-4001f32cd33d" />


Input the following configurations below: 

Name:	sensorvm
Type:	A
TTL: 1
IP address:	10.1.1.4
<img width="479" height="635" alt="image (25)" src="https://github.com/user-attachments/assets/363b6544-b458-4011-891a-3167ee8b8221" />






I built a virtual network in Azure to simulate a company environment in the cloud. I created multiple subnets to organize departments and prevent IP conflicts, making the setup easier to manage and troubleshoot in the future. I also configured security groups to control network traffic and protect key resources, web and data servers. I was able to configure network security groups and application security groups by applying rules to groups instead of individual machines. Finally, I set up Azure DNS for internal and external name resolution, ensuring smooth and secure communication between systems.
