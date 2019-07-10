Kubernetes secret objects let you store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys. Putting this information in a secret is safer and more flexible than putting it verbatim in a Pod definition or in a container image .

### Creating your own Secrets

- Creating secrets from files:

```
kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
```

### Creating a Secret Manually

- You can also create a Secret in a file first, in json or yaml format, and then create that object.

- The Secret contains two maps: data and stringData. The data field is used to store arbitrary data, encoded using base64.

- The stringData field is provided for convenience, and allows you to provide secret data as unencoded strings.

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

- For certain scenarios, you may wish to use the stringData field instead. This field allows you to put a non-base64 encoded string directly into the Secret, and the string will be encoded for you when the Secret is created or updated.

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret1
type: Opaque
data:
  username: root
  password: mypassword
```

### Decoding a Secret

- Get the secret detail in yaml format:

```
~/Documents/GitHub/k8s-info/ckad/secrets >>> kubectl get secrets mysecret1 -o yaml
apiVersion: v1
data:
  password: bXlwYXNzd29yZA==
  username: cm9vdA==
kind: Secret
metadata:
  creationTimestamp: "2019-06-03T04:58:08Z"
  name: mysecret1
  namespace: default
  resourceVersion: "13591"
  selfLink: /api/v1/namespaces/default/secrets/mysecret1
  uid: 2e3bbc89-85bc-11e9-b0a9-080027d717d4
type: Opaque
```

- Decode the secret as below:

```
echo 'bXlwYXNzd29yZA==' | base64 --decode
```

### Using Secrets

- Secrets can be mounted as data volumes to be used by a container in a pod

- Secrets can be be exposed as environment variables to be used by a container in a pod


#### Using Secrets as Files from a Pod

- This is an example of a pod that mounts a secret in a volume:

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-secret-as-vol
spec:
  containers:
  - name: pod-with-secret-as-vol
    image: busybox
    command: ['sh', '-c', "echo $(ls -l /etc/foo) && sleep 3600"]
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```

- You can also specify the permission mode bits files part of a secret will have. If you don’t specify any, 0644 is used by default.

- Inside the container that mounts a secret volume, the secret keys appear as files and the secret values are base-64 decoded and stored inside these files.

#### Using Secrets as Environment Variables

- This is an example of a pod that mounts a secret in a volume:

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-secret-as-env
spec:
  containers:
  - name: pod-with-secret-as-env
    image: busybox
    command: ['sh', '-c', "echo $(env) && sleep 3600"]
    env:
    - name: USER_NAME
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysecret
          key: password
```

- Reading all the items from a secret as env variables:

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-secret-as-env1
spec:
  containers:
  - name: pod-with-secret-as-env1
    image: busybox
    command: ['sh', '-c', "echo $(env) && sleep 3600"]
    envFrom:
      - secretRef:
          name: mysecret
```


### Few important points:

- Secret volume sources are validated to ensure that the specified object reference actually points to an object of type Secret. Therefore, a secret needs to be created before any pods that depend on it.

- Secret API objects reside in a namespace . They can only be referenced by pods in that same namespace.

- Individual secrets are limited to 1MiB in size. This is to discourage creation of very large secrets which would exhaust apiserver and kubelet memory.

- Secrets must be created before they are consumed in pods as environment variables unless they are marked as optional.

- When a pod is created via the API, there is no check whether a referenced secret exists. Once a pod is scheduled, the kubelet will try to fetch the secret value. If the secret cannot be fetched because it does not exist or because of a temporary lack of connection to the API server, kubelet will periodically retry. It will report an event about the pod explaining the reason it is not started yet. Once the secret is fetched, the kubelet will create and mount a volume containing it.
