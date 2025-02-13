# Interview-Devops

* docker install on amazon linux for below practice


```
sudo amazon-linux-extras enable docker
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user
docker --version
docker info
```

## 1. What is multistage docker build?
   
   A multi-stage build is a process that allows you to break the steps in building a Docker image into multiple stages. This will enable you to create images that include only the dependencies that are necessary for the desired functionality of the final application, cutting down on both time and space. With a multi-stage build, you will first build the image that contains only the dependencies needed to build your application. Then, after the image has been built, you can add in any additional layers needed to create your application and configure it for deployment.

### Java Multi-stage Docker Build Example
```
HelloWorld.java
class HelloWorld {
    public static void main(String[] a) {
    System.out.println("Hello world!");   
    }
}
```

### create a Dockerfile with the following content in it
```
FROM openjdk:11-jdk
COPY HelloWorld.java .
RUN javac HelloWorld.java
CMD java HelloWorld
```
Build the image with the following command
```
docker build -t helloworld:huge .
```
### modify our Dockerfile with the following content to show how multi-stage Docker build works
```
FROM openjdk:11-jdk AS build
COPY HelloWorld.java .
RUN javac HelloWorld.java
 
FROM openjdk:11-jre AS run
COPY --from=build HelloWorld.class .
CMD java HelloWorld
```

### Build the image with the following command,
```
docker build -t helloworld:small .
```
 ## compare both images. Check the images created with the following command, 
```
docker images
```
### conclusion

There is a large difference in size between the two images. This difference allows you to separate the build and runtime environments in the same Dockerfile. Use build environment as a dependency [COPY --from=build HelloWorld.class .] while creating the Dockerfile with the approach of multi-stage docker build. This will help minimize the size of Docker images.

######################################################################################################
## Difference between COPY and ADD in dockerfile

Dockerfile for COPY
```
FROM Centos:7
COPY /testdir /testdir
```

Docker file for ADD

case 1: 
```
FROM Centos:7
ADD /testdir /testdir
```
case 2:

```
FROM Centos:7
ADD testdir.app.tar.gz /testdir
```
case3:
```
FROM Centos:7
ADD https://testdir.app.tar.gz /testdir
```

## Difference between ENTRYPOINT and CMD
dockerfile

```
# Use the official Nginx base image
FROM nginx:latest

# Copy custom configuration file to the container
COPY nginx.conf /etc/nginx/nginx.conf

# Copy website files to the container (optional)
COPY html/ /usr/share/nginx/html/

# Expose port 80
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

docker run ubuntu command
docker run ubuntu sleep 5

```
FROM ubuntu
CMD sleep 5 
```
docker build -t ubuntu-sleeper .
docker run ubuntu-sleeper

```
FROM ubuntu
ENTRYPOINT ["sleep"] 
```
If the argument is not passed the above docker run will exit

```
FROM ubuntu
ENTRYPOINT ["sleep"] 
CMD ["5"]
```
We can override 5 by passing 
docker run --entrypoint sleep2.0 ubuntu-sleeper 10
#########################################################################################################################################################################################

## How to restrict pull images from dockerhub to kubernetes

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

##########################################################################################################################################################################################
