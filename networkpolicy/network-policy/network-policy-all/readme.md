```
kubectl apply -f nginx-pod.yaml
kubectl apply -f busybox-pod.yaml
kubectl apply -f allow-ingress-from-busybox.yaml

testing
kubectl get pods -n demo-namespace

kubectl get pod nginx -n demo-namespace -o jsonpath='{.status.podIP}'
kubectl exec -n demo-namespace -it busybox -- /bin/sh
curl http://10.244.0.10

```

If the NetworkPolicy is correctly allowing ingress traffic, you should see the HTML content served by the nginx pod, which looks like the default nginx welcome page.
