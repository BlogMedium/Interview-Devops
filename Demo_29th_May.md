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

## The difference between Docker ENTRYPOINT and CMD

* ENTRYPOINT is the process that’s executed inside the container.
* CMD is the default set of arguments that are supplied to the ENTRYPOINT process.

# When to use ENTRYPOINT vs CMD?

example 1: Using Docker ENTRYPOINT

```
FROM alpine:latest
ENTRYPOINT ["ls"]
docker build -t entrypoint-demo:latest .

The ENTRYPOINT instruction means Docker runs the ls command when the container starts. As no CMD is set, the command is called without arguments.

You can pass arguments directly through to the command by appending them to your docker run statement:

docker run entrypoint-demo:latest -alh

```

Example 2 - Using Docker CMD

```
FROM alpine:latest
CMD ["ls"]

docker build -t cmd-demo:latest .

The output is the same as before. Because no ENTRYPOINT is set, the CMD of ls is appended to the default ENTRYPOINT, resulting in /bin/sh -c "ls" being executed—the ls command still runs, but as a shell subprocess.

Now observe what happens if you try to pass arguments to the ls command via docker run:

This doesn’t work because ls isn’t the ENTRYPOINT. The -alh is passed as an argument to the default /bin/sh -c entrypoint, which results in the container’s shell trying to evaluate -alh as a command.
```

Example: Using ENTRYPOINT and CMD together

```
FROM alpine:latest
ENTRYPOINT ["ls"]
CMD ["-alh"]

docker build -t entrypoint-cmd-demo:latest .
```
You can still supply custom arguments instead by overriding the CMD with docker run:
```
docker run entrypoint-cmd-demo:latest -p --full-time
```

** ENTRYPOINT isn’t usually overridden by end users, but CMD can easily be changed through docker run **

## How to restrict pull images form dockerhub to kubernetes

** Kubernetes deployments can specify an imagePullPolicy which influences how the image is pulled.** 

## Minize the docker image size

* Utilize a Minimal Base Image:
* Leverage Multi-Stage Builds:
* Minimize the layers
Every instruction in your Dockerfile creates a new layer in the image. Aim to reduce the number of layers for efficient caching and smaller image size. Combine multiple commands, particularly RUN commands, whenever possible.
* Employ .dockerignore:
This file instructs Docker to exclude specific files and directories from being copied into the image during the build process. This helps eliminate unnecessary files that bloat the image size.
* Remove Unnecessary Packages and Dependencies:
Carefully analyze the packages you install using your package manager (e.g., apt-get or apk add). Avoid installing unnecessary packages or development tools that won't be used in the final runtime environment. Utilize flags like --no-install-recommends with apt-get to prevent installation of recommended but non-essential packages.
* Clean Up After Installations:
After installing packages using your package manager, consider cleaning up the cache and temporary files generated during the installation process. This can free up space within the image. Tools like apt-get clean or apk del can be used for this purpose, but ensure they are used within the same RUN instruction as the installation to maintain proper caching behavior.

************************************************************************************************************************************

# Terraform Implicit Dependency

```
provider "aws" {
  region = var.aws_region
}
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}
resource "aws_instance" "example_a" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t2.micro"
}
resource "aws_instance" "example_b" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t2.micro"
}
resource "aws_eip" "ip" {
  vpc      = true
  instance = aws_instance.example_a.id
}
```

<p> The aws_eip resource type allocates and associates an Elastic IP to an EC2 instance. Since the instance must exist before the Elastic IP can be created and attached, Terraform must ensure that aws_instance.example_a is created before it creates aws_eip.ip. Meanwhile, aws_instance.example_b can be created in parallel to the other resources.</p>

##  Manage explicit dependencies

```

resource "aws_s3_bucket" "example" { }
resource "aws_instance" "example_c" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t2.micro"
  depends_on = [aws_s3_bucket.example]
}
module "example_sqs_queue" {
  source  = "terraform-aws-modules/sqs/aws"
  version = "3.3.0"
  depends_on = [aws_s3_bucket.example, aws_instance.example_c]
}

```
### how to add code in docker container








  





  


   
