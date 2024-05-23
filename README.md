# Interview-Devops
1. what is multistage docker build?
   
   A multi-stage build is a process that allows you to break the steps in building a Docker image into multiple stages. This will enable you to create images that include only the dependencies that are necessary for the desired functionality of the final application, cutting down on both time and space. With a multi-stage build, you will first build the image that contains only the dependencies needed to build your application. Then, after the image has been built, you can add in any additional layers needed to create your application and configure it for deployment.

## Java Multi-stage Docker Build Example
</code>
HelloWorld.java
class HelloWorld {
    public static void main(String[] a) {
    System.out.println("Hello world!");   
    }
}
</code>

## 
create a Dockerfile with the following content in it
FROM openjdk:11-jdk
COPY HelloWorld.java .
RUN javac HelloWorld.java
CMD java HelloWorld
Build the image with the following command,

docker build -t helloworld:huge .
## modify our Dockerfile with the following content to show how multi-stage Docker build works

FROM openjdk:11-jdk AS build
COPY HelloWorld.java .
RUN javac HelloWorld.java
 
FROM openjdk:11-jre AS run
COPY --from=build HelloWorld.class .
CMD java HelloWorld

## Build the image with the following command,

docker build -t helloworld:small .

 ## compare both images. Check the images created with the following command, 

docker images

##conclusion
There is a large difference in size between the two images. This difference allows you to separate the build and runtime environments in the same Dockerfile. Use build environment as a dependency [COPY --from=build HelloWorld.class .] while creating the Dockerfile with the approach of multi-stage docker build. This will help minimize the size of Docker images.

