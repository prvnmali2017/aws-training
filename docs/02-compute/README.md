
## Compute
### AWS Training Module 2

---

## Agenda

- Virtual machines
- Identity and Access management
- Virtual networks
- Auto Scaling and Load-balancing

--

## Prerequisites

- Browser and Internet access
- SSH client (e.g. [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) on Windows)

---

# Elastic Compute Cloud

--

## [Elastic Compute Cloud (EC2)](http://aws.amazon.com/ec2/)

- One of the core services of AWS
- Virtual machines (or [*instances*](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Instances.html)) as a service
- Dozens of [*instance types*](http://aws.amazon.com/ec2/instance-types/) that vary in performance and cost
- Instance is created from an [*Amazon Machine Image (AMI)*](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html), which in turn can be created again from instances

--

![AWS Region map](/images/aws_map_regions.png)

[Regions](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) and CDN [Edge Locations](http://aws.amazon.com/about-aws/global-infrastructure/)

Notes: Regions: Frankfurt, Ireland, US East (N. Virginia), US West (N. California), US West (Oregon), South America (Sao Paulo), Tokyo, Singapore, Sydney. Special regions are **GovCloud** and **Beijing**.

--

![AWS EU Region map](/images/aws_map_regions_eu.png)

[Regions and Availability Zones (AZ)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)

Notes: We will only use Ireland (eu-west-1) region in this workshop. See also [A Rare Peek Into The Massive Scale of AWS](http://www.enterprisetech.com/2014/11/14/rare-peek-massive-scale-aws/).

--

## Networking in AWS

- Regions and availability zones
- [*Security groups*](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html) provide port-level firewalls to instances
- More detailed IP subnetting with [Virtual Private Cloud (VPC)](http://aws.amazon.com/vpc/)

--

## Exercise: Launch an EC2 instance

1. Log-in to [gofore-crew.signin.aws.amazon.com/console](https://gofore-crew.signin.aws.amazon.com/console)
2. Switch to **Ireland** region and go to EC2 dashboard
3. Launch a new EC2 instance according instructor guidance
  - In *"Configure Instance Details"*, pass a [*User Data*](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) script under *Advanced*
  - In *"Configure Security Group"*, use a recognizable, unique name

<pre><code data-trim="" class="shell">
#!/bin/sh
# When passed as User Data, this script will be run on boot
touch /new_empty_file_we_created.txt
echo "It works!" > /it_works.txt
</code></pre>

--

## Exercise: SSH into the instance

SSH into the instance (find the IP address in the EC2 console)

    # Windows Putty users must convert key to .ppk (see notes)
    ssh -i your_ssh_key.pem ubuntu@instance_ip_address

View instance metadata

    curl http://169.254.169.254/latest/meta-data/

View your [*User Data*](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html) and find the changes your script made

    curl http://169.254.169.254/latest/user-data/
    ls -la /

Notes: You will have to reduce keyfile permissions `chmod og-xrw mykeyfile.pem`. If you are on Windows and use Putty, you will have to convert the .pem key to .ppk key using [puttygen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) (Conversions -> Import key -> *.pem file -> Save private key. Now you can use your *.ppk key with Putty: Connection -> SSH -> Auth -> Private key file)

--

## Exercise: Security groups

Setup a web server that hosts the id of the instance

    mkdir ~/webserver && cd ~/webserver
    curl http://169.254.169.254/latest/meta-data/instance-id > index.html
    python -m SimpleHTTPServer

Configure the security group of your instance to allow inbound requests to your web server from **anywhere**. Check that you can access the page with your browser.

--

## Exercise: Security groups

Delete the previous rule. Ask a neighbor for the name of their security group, and allow requests to your server from your **neighbor's security group**.

Have your neighbor access your web server from his/her instance.

    # Private IP address of the web server (this should work)
    curl 172.31.???.???:8000
    # Public IP address of the web server (how about this one?)
    curl 52.??.???.???:8000

--

## [Elastic Block Store (EBS)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html)

- Block storage service (virtual hard drives)
- Disks (or [*volumes*](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumes.html)) are attached to instances
- [*Snapshots*](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html) can be taken from volumes
- Alternative to EBS is ephemeral [*instance store*](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html)

--

## EC2 cost

- Instances are billed every starting [*instance-hour*](http://aws.amazon.com/ec2/pricing/)
- Purchasing options of [*On-Demand Instances*, *Reserved Instances*, *Spot Instances*](http://aws.amazon.com/ec2/purchasing-options/)
- Storage costs for volumes, snapshots and images
- Traffic costs (more the further the traffic is towards the Internet)

---

# Identity and Access Management

--

## [Identity and Access Management (IAM)](http://aws.amazon.com/iam/)

- Manage AWS user [*credentials*](http://docs.aws.amazon.com/IAM/latest/UserGuide/Using_ManagingLogins.html) for Web console and API access
- Fine-grained access [*policies*](http://docs.aws.amazon.com/IAM/latest/UserGuide/policies.html) to services and resources
- Provide temporary access with [*roles*](http://docs.aws.amazon.com/IAM/latest/UserGuide/roles-toplevel.html)

Notes: Always use roles, do not store credentials inside the instances, or [something bad](http://www.browserstack.com/attack-and-downtime-on-9-November) might happen.

--

## Quiz: Users on many levels

Imagine running a content management system, discussion board or blog web application in EC2. How many **different types** of user accounts you might have?

---

# Virtual Private Cloud

--

## [Virtual Private Cloud (VPC)](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html)

- Heavy-weight virtual IP networking for EC2 and RDS instances. Integral part of modern AWS, all instances are launched into VPCs (*not true for [EC2-classic](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-vpc.html)*)
- An AWS root account can have many VPCs, each in a specific region
- Each VPC is divided into [*subnets*](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Subnets.html), each bound to an availability zone
- Each instance connects to a subnet with a [*Elastic Network Interface*](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html)

--

![VPC with Public and Private Subnets](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/images/nat-instance-diagram.png)

[VPC with Public and Private Subnets](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Scenario2.html)

--

## Access Control Lists

- Network [*Access Control List (ACL)*](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ACLs.html) provide a second layer of security
- See [Comparison of Security Groups and Network ACLs](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Security.html#VPC_Security_Comparison)

--

![VPC with Public and Private Subnets and Hardware VPN Access](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/images/Case3_Diagram.png)

[VPC with Public and Private Subnets and Hardware VPN Access](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Scenario3.html)

--

## Recap

- Instance and Elastic Network Interface
- Region and Availability Zone
- VPC, Subnet, Route Table
- Network ACL, Instance Security Group
- Internet Gateway, Virtual Private Gateway, NAT instance

--

## Quiz: Separating environments

You have *Development* and *Production* environments.

Both of them have *web servers* and *database servers*.

How would you separate them?


---

# [Auto Scaling](http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/WhatIsAutoScaling.html)

--

![Static provisioning](/images/provisioning_static.png)

Problem of traditional capacity planning

--

![Elastic provisioning](/images/provisioning_elastic.png)

Provisioning capacity as needed

--

- Vertical scaling (*scale up, scale down*)
- Horizontal scaling (scale out, scale in)

- 1 instance 5 hours = 5 instances 1 hour

--

## Auto Scaling instances

- *Auto Scaling Group* contains instances whose lifecycles are automatically managed by CloudWatch alarms or schedule
- A *scaling policy* describes how the group scales in or out. You should always have policies for both directions. [*Policy cooldowns*](http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/Cooldown.html) control the rate in which scaling happens.
- A [*launch configuration*](http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/LaunchConfiguration.html) describes the configuration of the instance. Having a good AMI and bootstrapping is crucial.

--

![Auto Scaling Group Lifecycle](http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/images/as-lifecycle-basic-diagram.png)

[Auto Scaling Group Lifecycle](http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/AutoScalingGroupLifecycle.html)

--

## [Elastic Load Balancer](http://aws.amazon.com/elasticloadbalancing/)

- Route traffic to an Auto Scaling Group (ASG)
- Runs health checks to instances to decide whether to route traffic to them
- Spread instances over multiple AZs for higher availability
- ELB scales itself. Never use ELB IP address. Pre-warm before flash traffic.

--

![ELB Architecture](http://awsmedia.s3.amazonaws.com/2012-02-24-techdoc-elb-arch.png)

[Best practices in ELB](https://aws.amazon.com/articles/1636185810492479)

--

![Autoscaling with alarms](/images/aws_workshop_arch_3_alarms.png)



## Exercise: Monkey time!

![Chaos Monkey](/images/netflix-chaos-monkey.jpg)

Be a [Chaos Monkey](https://github.com/Netflix/SimianArmy/wiki/Chaos-Monkey): terminate all of your `aws-workshop-ui` instances!

---

# Public networking

--

- [Route 53](http://aws.amazon.com/route53/) Domain Name System (DNS)
- [CloudFront](http://aws.amazon.com/cloudfront/) Content Delivery Network (CDN)
- [Elastic IP](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)

---

# Recap

--

## Recap

- [EC2](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html): Region, Availability Zone, Instance, Security group, AMI, [EBS](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html) volume, EBS snapshot, Instance store
- [IAM](http://docs.aws.amazon.com/IAM/latest/UserGuide/IAM_Introduction.html): User, User group, Policy, Permission, Role, API access key, Root account
- [VPC](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html): VPC, Subnet, Route table, ACL, NAT instance, Internet gateway, Virtual Private gateway
- [Auto Scaling](http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/WhatIsAutoScaling.html): Auto Scaling Group, Launch configuration, Scaling policy, [Elastic Load Balancer](http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/elastic-load-balancing.html)