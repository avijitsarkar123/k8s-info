- A service account provides an identity for processes that run in a Pod.

- When you (a human) access the cluster (for example, using kubectl), you are authenticated by the apiserver as a particular User Account (currently this is usually admin, unless your cluster administrator has customized your cluster).

- Processes in containers inside pods can also contact the apiserver. When they do, they are authenticated as a particular Service Account (for example, default).

### Listing Service Accounts

```
kubectl get serviceAccounts
```

### Creating Service Accounts

- Without using yaml file:

```
kubectl create serviceaccount bot-account
```

```
kubectl get serviceaccounts bot-account -o yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2019-06-05T04:02:07Z"
  name: bot-account
  namespace: default
  resourceVersion: "20088"
  selfLink: /api/v1/namespaces/default/serviceaccounts/bot-account
  uid: af758438-8746-11e9-b0a9-080027d717d4
secrets:
- name: bot-account-token-gsfhc
```

- Creating service account using definition YAML:

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bot-service-account
```

```
kubectl create -f bot-service-account.yaml
```

- A token gets generated with the service account, the token is stored inside a secret

- To use a non-default service account, simply set the spec.serviceAccountName field of a pod to the name of the service account you wish to use.

- The service account has to exist at the time the pod is created, or it will be rejected.


### Using a Service Account

Add `spec.serviceAccount` in pod definition to add a service account to a pod.

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-secret-as-env
spec:
  serviceAccount: bot-service-account
  containers:
  - name: pod-with-secret-as-env
    image: busybox
    command: ['sh', '-c', "echo $(env) && sleep 3600"]
```
