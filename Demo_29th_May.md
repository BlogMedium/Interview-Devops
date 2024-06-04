## Solution

```

eksctl create iamserviceaccount \
  --region us-west-2 \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster new-metric-cluster\
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-only \
  --role-name AmazonEKS_EBS_CSI_DriverRole

eksctl create addon --name aws-ebs-csi-driver --cluster new-metric-cluster --service-account-role-arn arn:aws:iam::$(aws sts get-caller-identity --query Account --output text):role/AmazonEKS_EBS_CSI_DriverRole --force

eksctl utils associate-iam-oidc-provider --region=us-west-2 --cluster=new-metric-cluster --approve



```

## Difference between network load balancer and application load balancer.
Step1 : 
* Create ec2 instance using launch template
* Security group port 22 and port 80 inbound rules 
* with the below userdata
```
#!/bin/bash
sudo yum -y update
sudo yum -y install ruby
sudo yum -y install wget
sudo yum -y install httpd
TOKEN=`curl --silent -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 3600"`
REGION=`curl --silent -H "X-aws-ec2-metadata-token: $TOKEN"  http://169.254.169.254/latest/meta-data/placement/availability-zone | sed 's/\(.*\)[a-z]/\1/'`
INSTANCE_ID=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id)
# Log the metadata fetching results for debugging
echo "Instance ID: $INSTANCE_ID" >> /var/log/user-data.log
echo "Region: $REGION" >> /var/log/user-data.log
echo "<h1>$INSTANCE_ID</h1>" > /var/www/html/index.html
# Start the Apache service and enable it to start on boot
systemctl start httpd
systemctl enable httpd
```


## create ASG 

### Network loadbalancer

* Operates in network layer. uses tcp,tls and udp protocol
*  used for appliction that need millions of request per second
* uses static IP/elastic ip.

## Application loadbalancer

```
#!/bin/bash
sudo yum -y update
sudo yum -y install ruby
sudo yum -y install wget
sudo yum -y install httpd
mkdir 775 -p /var/www/html/register/index.html
echo "<h1>this is register group </h1>" > /var/www/html/register/index.html
# Start the Apache service and enable it to start on boot
systemctl start httpd
systemctl enable httpd

```

## application loadbalancer
* Operates in applicatio layer uses protocol like http/https
* path based routing

## The Linux boot process can be broken down in 4 simple stages:

1. BIOS

The BIOS (stands for "Basic Input/Output System") initializes the hardware and makes sure with a Power-on self test (POST) that all the hardware is good to go. The main job of the BIOS is to load up the bootloader.

2. Bootloader

The bootloader loads the kernel into memory and then starts the kernel with a set of kernel parameters. One of the most common bootloaders is GRUB, which is a universal Linux standard.

3. Kernel

When the kernel is loaded, it immediately initializes devices and memory. The main job of the kernel is to load up the init process.

4. Init

Remember the init process is the first process that gets started, init starts and stops essential service process on the system. There are three major implementations of init in Linux distributions. We will go over them briefly and then dive into them in another course

## Dns

Let's look at an example of how your host finds a domain (catzontheinterwebz.com) with DNS.

Local DNS Server

First our host asks, "Where is catzontheinterwebz.com?", our local DNS server doesn't know so it goes and starts from the top of the funnel to ask the Root Servers. Keep in mind that our host is not making these requests to find catzontheinterwebz.com directly, most users talk to a recursive DNS server provided by their ISPs and that server is then tasked to find the location of catzontheinterwebz.com.

Root Servers

There are 13 Root Servers for the Internet, they are mirrored and distributed around the world to handle DNS requests for the Internet, so there are really hundreds of servers that are working, they are controlled by different organizations and they contain information about Top-Level Domains. Top-level domains are what you know as .org, .com, .net, etc addresses. So the Root Server doesn't know where catzontheinterwebz.com is, so it tells us ask the .com Top-Level Domain DNS Server at an IP address it gives us.

Top-Level Domain

So now we send another request to the name server that knows about ".com" addresses and asks if it knows where catzontheinterwebz.com is? The TLD doesn't have the catzontheinterwebz.com in their zone files, but it does see a record for the name server for catzontheinterwebz.com. So it gives us the IP address of that name server and tells us to look there.

Authoritative DNS Server

Now we send a final request to the DNS server that actually has the record we want. The name server sees that it has a zone file for catzontheinterwebz.com and there is a resource record for 'www' for this host. It then gives us the IP address of this host and we can finally see some cats on the Internet.


######################################################################################################



  





  


   
