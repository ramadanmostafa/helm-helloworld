# Hello World

If you are using Amazon EKS, your configuration will look like:

```
http_proxy="http://proxy.example.com:8080"
https_proxy="http://proxy.example.com:8080"
no_proxy="<VPC CIDR RANGE>,<EKS Service CIDR Range>,<VPC Endpont DNS Names>,localhost,127.0.0.1"

kubectl -n kube-system create confimap proxy-config \
  --from-literal=http_proxy="$http_proxy" \
  --from-literal=HTTP_PROXY="$http_proxy" \
  --from-literal=https_proxy="$https_proxy" \
  --from-literal=HTTPS_PROXY="$http_proxy" \
  --from-literal=no_proxy="$no_proxy" \
  --from-literal=NO_PROXY="$no_proxy"
```

Once the ConfigMap is created in the kube-system namespace, we need to update the kube-proxy and aws-node DaemonSets.

```
kubectl patch -n kube-system -p '{ "spec": {"template": { "spec": { "containers": [ { "name": "aws-node", "envFrom": [ { "configMapRef": {"name": "proxy-config" } } ] } ] } } } }' daemonset aws-node
kubectl patch -n kube-system -p '{ "spec": {"template":{ "spec": { "containers": [ { "name": "kube-proxy", "envFrom": [ { "configMapRef": {"name": "proxy-config" } } ] } ] } } } }' daemonset kube-proxy
kubectl set env daemonset/kube-proxy --namespace=kube-system --from=configmap/proxy-config --containers='*'
kubectl set env daemonset/aws-node --namespace=kube-system --from=configmap/proxy-config --containers='*'
```

Now, the pods on your Amazon EKS cluster will leverage the HTTP Proxy within your environment.