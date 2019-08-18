
# Docker demo

## create a container
```
docker run --rm -d nginx:latest
```

user browser to check http://localhost

## port mapping (highlight resource isolation)
```
docker run --rm -d -p 8888:80 nginx:latest
docker run --rm -d -p 9999:80 nginx:latest
```

use browser to check http://localhost:8888, http://localhost:9999

## stats
```
docker stats
```

## allocate cpu & memory
```shell
docker run -d --rm --cpus 0.1 -m 1g -p 2181:2181 zookeeper:latest
```

# Kubernetes demo

## 0, List deployment. List pods
use the following commands
```shell
kubectl get deploy
kubectl get pods
```
to show no deployments or pods currently.


## 1, Create deployments
```shell
kubectl create -f payment-deploy.yaml
kubectl create -f account-deploy.yaml
```
list deployment and pods again.
Pods are running, but for now, not accessible from outside of the cluster.

## 2, Create services
```shell
kubectl create -f payment-service.yaml
kubectl create -f account-service.yaml
```

then List services
```shell
kubectl get svc
```
and get the cluster IP using
```shell
minikube ip
```

Then use browser to verify pods are running.
```
http://{clusterip}:{servicePort}
```

## 3, Create ingress
```shell
kubectl create -f myapp-ingress.yaml
```
map paths to backend services;
so that for now, it is all backend services could be accessed via different paths of a same host.

## 4, Pretend we have a DNS server
add the following entry in `/etc/hosts` file
```
{clusterip} my.awesome.app
```
Then verify ingress is working by hitting the following urls in a browser
```
http://my.awesome.app/account
http://my.awesome.app/payment
```
Up to this point, deployment is done.

## 5, Deploy a new version
how about deploying new versions of one of those services?
Do we need to bring down the whole app? Let's say payment gateway is upgraded, so we have to upgrade our payment service as well.

To see it more clearly, let setup a constant checking on the endpoint by
```shell
while true; do curl -w "\n" http://my.awesome.app/payment; sleep 1; done
```

then start to deployment new version.

```shell
kubectl set image deployment payment payment-container=jnupeter/payment:v2
```

## 6, Zero down time deployment
The key point to zero down time deployment lies in
```shell
kubectl describe deploy payment
```

See the “RollingUpdateStrategy: 25% max unavailable, 25% max surge”

## 7, What if one of the pods dies?
```shell
kubectl get pods
```
delete one of the pods to simulate a pod dies
```shell
kubectl delete pod the-unlucky-pod
```
Then
```shell
kubectl get pods
```
should see a new one spin up.

# Linkerd demo

## install the demo app
```
curl -sL https://run.linkerd.io/emojivoto.yml \
  | kubectl apply -f -
```

then export web app to localhost:8080 with

```
kubectl -n emojivoto port-forward svc/web-svc 8080:80
```

## generate some errors
Click on a doughnut emoji

## watch the pods
```
kubectl -n emojivoto get pods
```
pay attention to the number of containers in those pods.

## add linkerd to the Emojivoto app
by running
```
kubectl get -n emojivoto deploy -o yaml \
  | linkerd inject - \
  | kubectl apply -f -
```

## watch the pods again
```
kubectl -n emojivoto get pods
```

pay attention to the number of containers, and use
```
kubectl -n emojivoto describe pod {pod-name}
```
what container has been injected?

## service dependency diagram
```
linkerd dashboard &
```
Then show the service dependencies, via menu "Namespace" -> "emojivoto".

Note that "meshed" tag.
## debugging services
