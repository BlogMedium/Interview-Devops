```
kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
nslookup flask-normal
nslookup flask-headless

kubectl run -i --tty --image nginx:alpine test-pod --restart=Never --rm -- sh
curl flask-normal
curl flask-headless:5000
```
