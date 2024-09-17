Canary deployment using the nginx-ingress controller
====================================================

> In the following example, we shift traffic between 2 applications using the
[canary annotations of the Nginx ingress
controller](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#canary).

## Steps to follow

1. app-v1 (cas version 6) is serving traffic
1. deploy app-v2
1. create a new "canary" ingress with traffic splitting enabled
1. wait enought time to confirm that version 2 is stable and not throwing
   unexpected errors
1. delete the canary ingress
1. point the main application ingress to send traffic to version 2
1. shutdown version 1

## In practice

```bash
# Deploy app-v1 and expose the service via an ingress
# All traffic to CAS 6.0.0 now
$ kubectl apply -f ./app-v1.yaml -f ./ingress-v1.yaml

# Deploy app-v2
# The CAS 7.0.0 is running without any traffic
$ kubectl apply -f ./app-v2.yaml

# In a different terminal you can check that requests are responding with version 1
$ while sleep 0.1; do curl "${ingres_endopint}/andytest/"; done

# Create a canary ingress in order to split traffic: 90% to v1（CAS 6.0.0）, 10% to v2（CAS 7.0.0）
$ kubectl apply -f ./ingress-v2-canary.yaml

# Now you should see that the traffic is being splitted
$ while sleep 0.1; do curl "${ingres_endopint}/andytest/"; done

# When you are happy, delete the canary ingress
$ kubectl delete -f ./ingress-v2-canary.yaml

# Then finish the rollout, set 100% traffic to version 2（CAS 7.0.0）
$ kubectl apply -f ./ingress-v2.yaml

# Shutdown CAS 6.0.0 resources
```

### Cleanup

```bash
$ kubectl delete ns andy-test
```
