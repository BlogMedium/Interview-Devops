# Interview-Devops

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

## 2. what is network policy in kubernetes and  use case like egress ingress

* Kubernetes Network Policy is a resource type used to define how groups of pods are allowed to communicate with each other and other network endpoints. It provides a way to enforce rules governing traffic to and from pods, similar to how firewalls operate in traditional networks.

With Network Policies, we can:

* Isolate Pods: You can define rules to isolate pods within the same cluster or namespace, ensuring that they can only communicate with certain other pods.

* Control Traffic: Network Policies allow you to control inbound and outbound traffic to pods based on IP address, port, and other criteria.

* Increase Security: By enforcing network policies, you can enhance the security of your Kubernetes cluster by restricting unauthorized communication and minimizing the attack surface.

## Demo 

* Deny all connections
* allow specific ports
* allow from specific namespace
* allow specific pod from specific namespace

  ```
1. Create the namespace
   kubectl apply -f namespace.yaml
2. Deploy the nginx and busybox pods:
   kubectl apply -f nginx-pod.yaml
   kubectl apply -f busybox-pod.yaml
3. deploy the network policy
   kubectl apply -f deny-all-networkpolicy.yaml
```

## testing
Test Connectivity Using curl

After ensuring the pods are running and the NetworkPolicy is applied, you can use the curl command from the busybox pod to test connectivity to the nginx pod.

Get the IP address of the nginx pod
```
kubectl get pod nginx -n demo-namespace -o jsonpath='{.status.podIP}'
```
Exec into the busybox pod and use curl to test connectivity
```
kubectl exec -n demo-namespace -it busybox -- /bin/sh
```
Once inside the busybox pod, run:
curl http://10.244.0.10

If the NetworkPolicy is correctly allowing ingress traffic, you should see the HTML content served by the nginx pod. If it is blocking the traffic, curl will not be able to connect.


```
ranjiniganeshan@Ranjinis-MacBook-Pro network-policy-deny % kubectl get pod nginx -n demo-namespace -o jsonpath='{.status.podIP}'
192.168.84.199%                                                                                                              
ranjiniganeshan@Ranjinis-MacBook-Pro network-policy-deny % kubectl exec -n demo-namespace -it busybox -- /bin/sh
/ # 
/ # wget http://192.168.84.199
Connecting to 192.168.84.199 (192.168.84.199:80)
wget: can't open 'index.html': File exists
/ # exit
command terminated with exit code 1

```

##################################################################################################################################################




 
  

https://www.youtube.com/watch?v=Ptx1CcZSi6g



