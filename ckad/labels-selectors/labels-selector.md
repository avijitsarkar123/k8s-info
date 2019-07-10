- Labels are key/value pairs that are attached to objects, such as pods.
- Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system.

**Sample Label**

```
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```

**Label Selector Examples**

- Equality based:

  ```
  kubectl get pods -l environment=production,tier=frontend
  ```

- Set based:

  ```
  kubectl get pods -l 'environment in (production),tier in (frontend)'
  ```

  ```
  kubectl get pods -l 'environment in (production, qa)'
  ```

  ```
  kubectl get pods -l 'environment,environment notin (frontend)'
  ```

**Service and ReplicationController**

- The set of pods that a service targets is defined with a label selector. Similarly, the population of pods that a replicationcontroller should manage is also defined with a label selector.

- Labels selectors for both objects are defined in json or yaml files using maps, and only equality-based requirement selectors are supported:

```
selector:
    component: redis
```

- Newer resources, such as Job, Deployment, Replica Set, and Daemon Set, support set-based requirements as well.

```
selector:
  matchLabels:
    component: redis
  matchExpressions:
    - {key: tier, operator: In, values: [cache]}
    - {key: environment, operator: NotIn, values: [dev]}
```
