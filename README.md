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

```
ranjiniganeshan@Ranjinis-MacBook-Pro network-policy-deny % kubectl get pod nginx -n demo-namespace -o jsonpath='{.status.podIP}'
192.168.84.199%                                                                                                              
ranjiniganeshan@Ranjinis-MacBook-Pro network-policy-deny % kubectl exec -n demo-namespace -it busybox -- /bin/sh
/ # 
/ # wget http://192.168.84.199
Connecting to 192.168.84.199 (192.168.84.199:80)
wget: can't open 'index.html': File exists
/ # exit
command terminated with exit code 

```

##################################################################################################################################################
## configmap

It allows you to decouple environment-specific configuration from your container images, so your applications are more portable and easier to manage.

* Decoupling Configuration: Allows separation of configuration data from container images.
* Key-Value Pairs: Stores data as key-value pairs.
* Volume or Environment Variables: Can be used to inject configuration data into pods as environment variables or as configuration files mounted in volumes.

demo



#####################################################################################################################################################

Service account is a Kubernetes resource that gives pods access to Kubernetes APIs. It acts as an identity for processes running inside pods, providing them with the necessary credentials and permissions to authenticate and access specific resources within the cluster.

example AWS IAM roles, Google Cloud Service Accounts, and Azure Managed Identities.

When you, as a user, access a Kubernetes cluster, you typically authenticate through your user account defined in the kubeconfig file.

However, when it comes to applications running inside Kubernetes pods or external applications, they need a secure mechanism to authenticate and interact with the Kubernetes API server. This is where Kubernetes service accounts come into play.

By associating a pod with a Kubernetes service account, you provide the application running inside the pod with the necessary credentials and permissions to authenticate and access specific resources in the API server.


When a Service Account is created in Kubernetes, a token secret is automatically generated and associated with that Service Account. This token secret contains a JSON Web Token (JWT) used for authenticating the Service Account to the Kubernetes API server. Kubernetes automatically mounts this token secret into pods using the Service Account. Applications within these pods can then utilize the token for authentication purposes.


```
kubectl get sa ## shows default service account

kubectl describe po test-sa-pod | grep -i "service account"

 kubectl create sa application-sa

kubectl get sa application-sa -o yaml

Add a Service Account to a Pod

kubectl run nginx \
      --image=nginx \
      --overrides='{ "spec": { "serviceAccountName": "application-sa" } }' \ 
      --dry-run=client -o yaml > nginx.yaml

kubectl describe pod nginx | grep -i "service account"

```
############################################################################
## How to monitor the Kubernetes cluster? how to deploy monitoring tools

container Insights : CloudWatch Container Insights to collect, aggregate, and summarize metrics and logs from your containerized applications and microservices.

![cloudwatch](https://github.com/BlogMedium/Interview-Devops/assets/32661402/ad97b5ac-6272-45dc-9fcb-a998e9ed993a)

Steps 

1. Associate CloudWatch Policy to our EKS Worker Nodes Role. attach CloudWatchAgentServerPolicy
2. Install Container Insights
   * Creates the Namespace amazon-cloudwatch.
   * service account
   * role
   * role binding

Template. Replaced Cluster Name and Region
curl -s https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluentd-quickstart.yaml | sed "s/{{cluster_name}}/<REPLACE_CLUSTER_NAME>/;s/{{region_name}}/<REPLACE-AWS_REGION>/" | kubectl apply -f -

Replaced Cluster Name and Region

3. verify daemonsets
kubectl -n amazon-cloudwatch get daemonsets

4. deploy sample app

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-nginx-deployment
  labels:
    app: sample-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sample-nginx
  template:
    metadata:
      labels:
        app: sample-nginx
    spec:
      containers:
        - name: sample-nginx
          image: stacksimplify/kubenginx:1.0.0
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "5m" 
              memory: "5Mi"
            limits:
              cpu: "10m"
              memory: "10Mi"       
---
apiVersion: v1
kind: Service
metadata:
  name: sample-nginx-service
  labels:
    app: sample-nginx
spec:
  selector:
    app: sample-nginx
  ports:
  - port: 80
    targetPort: 80         

```



######################################################################################################################################################################

Python program to start and stop the ec2 instance

```
import boto3

# Create a session using your AWS credentials
session = boto3.Session(
    region_name='us-west-2'
)

# Create an EC2 client
ec2 = session.client('ec2')

# Function to start an instance
def start_instance(instance_id):
    ec2.start_instances(InstanceIds=[instance_id])
    print(f"Instance {instance_id} started.")

# Function to stop an instance
def stop_instance(instance_id):
    ec2.stop_instances(InstanceIds=[instance_id])
    print(f"Instance {instance_id} stopped.")

# Example usage
if __name__ == "__main__":
    # Replace 'INSTANCE_ID' with the ID of your EC2 instance
    instance_id = 'i-023890615d977a3c2'

    
    
    
    
    # Start the instance
    #start_instance(instance_id)

    # Stop the instance
    stop_instance(instance_id)
```
######################################################################
Python program to check if the string is palindrome or not

```
def is_palindrome(s):
    # Convert the string to lowercase and remove spaces
    s = s.lower().replace(" ", "")
    
    # Compare the string with its reverse
    return s == s[::-1]

# Example usage:
input_string = input("Enter a string: ")
if is_palindrome(input_string):
    print("The string is a palindrome.")
else:
    print("The string is not a palindrome.")
```
####################################################################
### RBAC 
### Authentication and Authorization

Once a request is authenticated, the authorization process determines whether the client has the necessary permissions to perform the requested action on the specified Kubernetes resources.

Every request involves an action, such as:

Listing pods
Creating deployments
Deleting services
Updating Deamonset
Viewing pod logs etc.





how to handle disaster recovery





