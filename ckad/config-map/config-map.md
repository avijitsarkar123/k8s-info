## Create Config Map


### Using literals:

```
kubectl create configmap [config-name] --from-literal=[key]=[value]
```

Example:

```
kubectl create configmap myappconfig \
  --from-literal=log_level=info \
  --from-literal=app_env=qa
```

### Using a file:

```
kubectl create configmap [config-name] --from-file=[path-to-file]
```

Example:

**appconfig.properties**

```
log_level=info
app_env=qa
```

```
kubeclt create configmap myappconfig --from-file=appconfig.properties
```

### Combine multiple files from a directory

The configmap will be created using all the files in that directory.

```
kubectl create configmap game-config --from-file=[path-to-directory]
```



### Using declarative YAML file:

**config-map.yaml**

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: myappconfig
data:
  log_level: INFO
  app_env: qa
```

Create the config map using the below command:

```
kubectl create -f config-map.yaml
```

## Passing Config Map to Pod


### As Env Variable:

Pod definition will look like:

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-config-map-as-env
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo $(LOG_LEVEL) && sleep 3600"]
    env:
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: myappconfig
          key: log_level
```

### As Container Volume:

Pod definition will look like:

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-config-map-as-vol
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "echo $(ls /etc/config) && sleep 3600"]
    volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: myappconfig
```

This will create files under `mountPath` i.e. `etc/config` for each key/value pair

If a config-map is created by reading multiple files from a directory ad below:

```
kubectl create configmap [config-name] --from-file=[path-to-directory]
```

In that case each file in directory will get added to the config as key:

```
~/Documents/GitHub/k8s-info/ckad/config_map >>> kubectl describe configmaps game-config
Name:         game-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
game.properties:
----
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
ui.properties:
----
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice

Events:  <none>
```

Now when this config-map is volume mapped inside the container, the `mountPath` will have two files:

```
~/Documents/GitHub/k8s-info/ckad/config_map >>> kubectl exec -it pod-config-map-as-vol1 sh
/ # ls -l /etc/config
total 0
lrwxrwxrwx    1 root     root            22 Jun  2 05:40 game.properties -> ..data/game.properties
lrwxrwxrwx    1 root     root            20 Jun  2 05:40 ui.properties -> ..data/ui.properties
/ # cat /etc/config/game.properties
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30/
```

### Configure all key-value pairs in a ConfigMap as container environment variables

We can read and export all the config-map key/value as env variables:

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-config-map-as-env1
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', "env && sleep 3600"]
    envFrom:
      - configMapRef:
          name: game-config
```

You can see all the `game-config` config-map values are now available as env variables:

```
~/Documents/GitHub/k8s-info/ckad/config_map >>> kubectl logs pod-config-map-as-env1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=pod-config-map-as-env1
ui.properties=color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice

SHLVL=1
HOME=/root
game.properties=enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/```
