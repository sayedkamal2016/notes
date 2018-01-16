# Amazon Web Services - Web Hosting & Cloud Computing

[Udemy](https://www.udemy.com/amazon-web-services-for-web-hosting-cloud-computing/)

## 101: What is the Cloud?

* **IaaS** (infrastructure), **PaaS** (platform), **SaaS** (software)
* **Rails** (verticals of services) and **tiers** (horizontals of same type)

## 102: Scalability and costs in the Cloud

* Think **operating expenditures**, not **capital expenditures** (upfront costs).
* Usually cheaper to use AWS services than building your own services on EC2 instances (but understand their limitations)
* Benefits: new services, economy of scale, autoscaling, regional servers, high availability

## 103: Cloud impacts on Architecture

* Leverage services wisely; buy rather than build
  - Know costs for each (e.g., storage and bandwidth)
  - EC2 probably main cost; minimize always-on
* Modularize heavily; use provided services wherever posible
* Know each service's limits
* Multi-AZ deployments for HA
* See AWS whitepapers for relevant architectures

## 104: Creating a cloud account with AWS

* Use email address that can be shared across organization

## 201: AWS value propositions

* EC2 charges are hourly, have data transfer costs, and other parameters

## 202: AWS Regions & Availability Zones

* **AZ**: single data center
  - Most services deploy to single AZ
  - HA achieved by deploying to multiple AZs
* **Region**: collection of AZs connected by high-bandwidth, low latency fiber
  - E.g., US East is default region
  - different regions can have different costs
* **Edge locations** push content close to users (often outside region)
* See "AWS Global Infrastructure" page

## 203: Intro to Service Families

* **EC2**: virtual servers
* **AutoScaling**
  - Can span multiple AZs
  - No additional cost
* **Elasticache**: caching via Redis or memcached
  - Memcached cluster
  - Useful for user sessions, caching DB queries
* **S3**: object storage
  - WORM: write once read many filesystem
  - Natively web accessible
* **EBS**: network attached storage
  - User-provisioned block storage
  - Attached to single EC2 instance
  - Can snapshot to S3
* **CloudFront**: CDN
  - Uses edge locations
  - Pull based (lazily added)
* **DynamoDB**: key-value (NoSQL) store
  - Specify read and write throughputs
  - Automatically replicate and reshard to multiple AZs
* **RDS**: fully manage SQL services
* **SQS**: extremely simple queues
  - Pull-based (requires polling)
* **SNS**: notification service for interally-bound msgs
  - Push-based via email, HTTP, SQS, SMS
* **SES**: email service for end-user emails
* **ELB**: elastic load balancers
  - Round-robin or sticky sessions
* **VPC**: user-defined private networks
  - No additional costs
  - Security groups/access control
* **Route 53**: distributed DNS service
* **CloudFormation**: templatize entire stacks of resources
  - Can version control
  - 200-300 predefined templates
* **CloudWatch**: monitor resources, billing
  - Control AutoScaling groups
  - Send notifications
  - Supports most AWS services
* **IAM**: Identity and Access Management
  - Controls actions, but not specific instances (w/ exceptions)

## 204: Roll Your Own vs AWS-Supplied Services

* Can easily install own services via EC2
  - Gain control
  - Can cost 4-15x more
  - Must additionally manage/admin
  - Steps:
    1. Spin up EC2 instance
    2. Install load balancer (e.g., HAProxy or NGINX)
    3. Configure software to run in EC2
    4. Configure DNS
* Only roll your own if AWS service limitations unacceptable
* When roll your own, account for:
  - HA (avoid SPOF)
  - DR (have backups)
  - Elasticity

## 205: Interfaces (web GUI, API, SDK)

* Web console missing functionality
* REST API, which CLI tools use
  - 100% feature support
* Lang-specific SDK
* Eclipse Plugin (instructor recommends)
* Instructor demo of IAM & policy generator

## 206: Intro to Authentication and Authorization

* Both cloud layer (AWS) and non-cloud layer (application, OS, server logins)
* **Master Account** is the keys to the kingdom
  - Basically, root level access
  - Put on a disk and store in a safe!
* **MFA** (multifactor authentication) is rotating codes, locks down individual API calls
* Master & IAM account access have different login URLs
* Use **bastion host**: single host accessible to outside world, other host accessible from it
* Launch EC2 instances into IAM Roles to avoid copying credentials everwhere
* Create & use "sshers" group to control access

## 301: EC2 instance types, key pairs, user-data

* Instance types are the "hardware footprint"
* Public/private key pairs control first-time access to new instance
* **Security Groups**: semi-stateful firewalls around instances
* **user-data**: store config/scripts per instance
  - Useful for bootstrapping
* EC2 min 1hr charge, so avoid repeat restarting
* `ssh -i foo.pem`
* EC2 Security Groups
  - Control ports, source, TCP or UDP
  - **CIDR blocks**. E.g.,
    - `122.43.0.0/16` (64k addresses)
    - `122.43.65.0/24` (255 addresses)
    - `122.43.65.8/32` (1 address)
    - `0.0.0/0` (allows all addresses)
* CloudFormation's **cfn-init** for initializing on first boot (e.g., install packages, set hostname)

## 302: EC2 Disk instances

* Ephemeral ("instance") store vs EBS
* **Hypervisor** makes slices of hardware resources available to VMs
* **Ephemeral store**:
  - Cannot be shared
  - Fast (local to EC2)
  - Lifetime tied to instance
  - Sequential I/O best
* EBS:
  - Remote rack, but still fast
  - Lifetime independent
  - Best for random I/O
  - Snapshots
  - Good for boot volume
  - Can only be attached to one EC2 instance at a time

## 303: Spinning up your first EC2 server & SSHing in

* Amazon Linux AMI
* Linux t1.micro (free)
* `chmod 400 foo.pem`
* Best practice: `sudo yum update`
* `curl http://<ip>/latest/meta-data/`
* EBS volume will be ~$0.10/month
* When take down, don't forget to down down EBS dist!

## 304: EC2 Gotchas

* Don't change default group (no access)
* Use elastic IPs, not public/private DNS
* Use "Terminate on Delete"
* Use "Termination Protection" for production machines
* Tag resources so can manage
* Don't copy credentials into instances or AMIs; leverage IAM "EC2 Roles" or user-data
* Use CloudFormation templates
* Don't use EC2 when there's a service available
* Don't oversize your machine; drive it like you stole it
* Don't manually provision if you will scale beyond 5-10 servers. Use Chef, Puppet, or at least CloudFormation

## 305: Templatizing Servers w/ AMIs

* **Instance types** are hardware footprnts; hypervisor offers resources (disk, cpu, ram, network) to VMs
* **Amazon Machine Image** (**AMIs**) are software footprint (OS, packages, user software)
* Amazon AMIs (trusted) vs Community AMIs (untrusted)
* Marketplace pre-packaged appliances
* MyAMIs
  - Options: base, partially configured, fully configured
  - Stored in S3

## 306: EBS Snapshots, Attaching, Detaching

* Find AZ for EC2 (e.g., us-east-2c)
* Go to "Elastic Block Storage" > "Volumes"
* "Provision IOPs" means more throughput (we'll use "Standard")
* AWS should provide commands for formatting, mounting
* To remove EBS from EC2 instance, it's better to unmount from terminal than to use AWS Console:
  ```bash
  $ sudo umount /mnt/newdisk
  $ sudo umount /dev/sdf
  ```
  Then, select volume in AWS Console and "Detach Volume"
* To create AMI:
  1. Go to "Running Instances"
  2. Select instance, right-client and select "Create Image (EBS AMI)"
  3. Fill out form
  4. Go to "Images" > "AMIs" (just book volume)
  * Cleanup:
    1. "Images" > "AMIs" > select and "Deregister"
    2. "EBS" > "Snapshots" > select and "Delete"
    3. "EBS" > "Volumes" > select and delete
    4. "Network & Security" > "Security Groups" > select and "Delete Security Group"

## 307: Pricing Model for EC2

* On Demand: most expensive, no up-front fee
* Reserved: up-front fee, lower hourly
* Spot: cheapest, bust can be taken away
* Amazon applies on-demand automatically
* Can sell reservations on marketplace
* Spot use cases:
  - Batch jobs
  - Offline processing
  - Big data analytics
  - Test/dev
* Spot best practices:
  - Break work into smallest pieces possible
  - Return useful results ASAP
  - Minimize data in & out
* Tip: Can use reserved for base traffic, then use spot and on-demand for peak

## 308: Making an AMI