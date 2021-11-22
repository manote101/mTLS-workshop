# Step 0: environment setup

# Step 1: install Linkerd CLI
```sh

curl https://run.linkerd.io/install | sh -

export PATH=$PATH:~/.linkderd2/bin
```

# Step 2: curl & nginx deploy
```ShellSession
less manifests/curl.yaml
less manifests/nginx-deploy.yaml

kubectl deploy -f manifests/curl.yaml
kubectl deploy -f manifests/nginx-deploy.yaml
```

# Step 3: verifying (no) mTLS!

```ShellSession
kubectl exec <curl-pod> -it -- bin/sh
# run after you start tshark below
/$ curl http://nginx-deploy.default.svc.cluster.local:80    

kubectl exec <nginx-pod> -it -c linkerd-debug -- bin/bash

/$ tshark -i any -d tcp.port==80,http
```

# Step 4: deploy Linkerd + certs

```ShellSession
linkerd install | kubectl apply -f -

kubectl get secret -n linkerd

# Inject Linkerd sidecar containter to deployment

kubectl get deploy -o yaml | linkerd inject - | kubectl apply -f -
```

# Step 5: are TLS'd yet?

```ShellSession
# same steps as 3
kubectl exec <curl-pod> -it -- bin/sh
# run after you start tshark below
/$ curl http://nginx-deploy.default.svc.cluster.local:80

kubectl exec <nginx-pod> -it -c linkerd-debug -- bin/bash

/$ tshark -i any -d tcp.port==80,ssl | grep -v 127.0.0.1
```

# Step 6: bonus content

Buoyant Cloud to verify TLS!

