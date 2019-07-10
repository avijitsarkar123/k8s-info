A security context defines privilege and access control settings for a Pod or Container. Security context settings include:

- Discretionary Access Control: Permission to access an object, like a file, is based on user ID (UID) and group ID (GID).

- Security Enhanced Linux (SELinux): Objects are assigned security labels.

- Running as privileged or unprivileged.

- Linux Capabilities: Give a process some privileges, but not all the privileges of the root user.

- AppArmor: Use program profiles to restrict the capabilities of individual programs.

- Seccomp: Filter a process’s system calls.

- AllowPrivilegeEscalation: Controls whether a process can gain more privileges than its parent process. This bool directly controls whether the no_new_privs flag gets set on the container process. AllowPrivilegeEscalation is true always when the container is: 1) run as Privileged OR 2) has CAP_SYS_ADMIN.


### A Pod with security context

```
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
```

- The runAsUser field specifies that for any Containers in the Pod, all processes run with user ID 1000.

- The runAsGroup field specifies the primary group ID of 3000 for all processes within any containers of the Pod. If this field is ommitted, the primary group ID of the containers will be root(0).

- Any files created will also be owned by user 1000 and group 3000 when runAsGroup is specified.

- Since fsGroup field is specified, all processes of the container are also part of the supplementary group ID 2000.

- The owner for volume /data/demo and any files created in that volume will be Group ID 2000.
