## Kubernetes operator

* Kubernetes can manage the complete lifecycle of stateless applications in a fully automated way, because these applications don’t have business logic for deployment.
So basically, once you deployed the application, you don’t have to sit there and control that your application is running properly.

### StateFUL applications WITHOUT Operator 

* For stateful applications, like databases, the whole process isn’t as straightforward.
  They need more "hand-holding" during its whole lifecycle, because the replicas of stateful apps aren't identical.

* Operator solves this problem and basically replaces this "human" operator with a "software" operator.
 At its core it has the same control loop mechanism that Kubernetes has, that watches for changes in the application state.
It also uses CRDs, which is basically a custom K8s component. So, it takes the basic Kubernetes resources and its controller concept as a foundation to build upon, and on top of that includes application-specific knowledge to automate the entire life cycle of the application it "operates".

managing the complete apps for stateful application via ** kubernetes operator **

example : prometheus operator.


