
# A demo of how to communicate with the Kubernetes API from within a pod running on the cluster

## Setup

1. Create a demo service account:
```kubectl apply -f 1_service_Account.yaml```
2. Create a role which allows certin actions to performed this role includes delete so be carefull, Referance:
  https://kubernetes.io/docs/reference/access-authn-authz/rbac/ &     
  https://kubernetes.io/docs/reference/access-authn-authz/authorization/
  
  	```kubectl apply -f 2_role.yaml```
  
3. Create a role bind to binf the service account we created with the role above:
	```kubectl apply -f 3_role_binding.yaml```
4. Create a demo pod that uses the above service account for testing:
	```kubectl apply -f 4_demo_pod.yaml```
	
## Test API communication
Run: ```kubectl exec -it busybox -- sh```

- On prompt (/ #) we add curl for generating an API call and jq for JSON formatting:
```apk add curl jq```

- Then we load the the service account token that was automaticlly mounted when the pod was created to an environment variable:
```KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)```

- Next we can use the loaded token to authenticate against the Kubernetes API, The following example get the current pod info:
```curl -sSk -H "Authorization: Bearer $KUBE_TOKEN" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_PORT_443_TCP_PORT/api/v1/namespaces/default/pods/$HOSTNAME | jq ```

- You can also delete this pod by running:
```curl -sSk -H "Authorization: Bearer $KUBE_TOKEN" https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_PORT_443_TCP_PORT/api/v1/namespaces/default/pods/$HOSTNAME -X DELETE| jq ```

**wait about 1 minute and the pod will terminate itself.**
	
## Teardown:
```kubectl delete -f 1_service_Account.yaml
kubectl delete -f 2_role.yaml
kubectl delete -f 3_role_binding.yaml
kubectl delete -f 4_demo_pod.yaml```
