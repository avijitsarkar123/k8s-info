https://medium.com/@andrewhibbert/passing-the-cka-and-ckad-d1c073dc9907

```
export KUBE_EDITOR="nano"
```

```
kubectl delete pod <xyz> --grace-period=0 --force
```

```
kubectl explain deployment.spec --recursive
```

```
kubectl config get-contexts: list all contexts
kubectl config current-context: get the current context
kubectl config use-context: change the current context
kubectl config set-context: change an element of a context
```

Create a deployment using `kubectl`

```
kubectl run nginx --image=nginx
```
