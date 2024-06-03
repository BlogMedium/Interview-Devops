## Headless Service 

1. pod to pod communication using pod's ip address or pod name

![pod communication using ip](https://github.com/BlogMedium/Interview-Devops/blob/main/pod_communication.jpeg)

2. pod to pod communication using cluster ip
   
![pod communiation using service ip](https://github.com/BlogMedium/Interview-Devops/blob/main/serviceip.jpeg)

![pod communiation using service ip with dns ](https://github.com/BlogMedium/Interview-Devops/blob/main/dns.jpeg)

## stateless service


![stateless](https://github.com/BlogMedium/Interview-Devops/blob/main/stateless.jpeg)

## stateful service


![stateful](https://github.com/BlogMedium/Interview-Devops/blob/main/stateful.jpeg)

# stateful with persistent volume

![stateful_pv](https://github.com/BlogMedium/Interview-Devops/blob/main/statefil_pv.jpeg)

## difference between stateful and stateless

# stateless
* does not depend on state
* Does not save client data generated in one session for use in next session with that client.
* ex nginx and web servers
* you can simply scale stateless applciation
* needs no external storage

  # stateful
  
  * depend on state of applciation.
  * saves data from each session to be used in the next session for the client
  * ex db, kafka, elasticsearch
  * needs persistent volume to store user data

## How to Troubleshoot Kubernetes ImagePullBackOff





    




