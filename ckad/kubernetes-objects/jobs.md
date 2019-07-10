- A Job creates one or more Pods and ensures that a specified number of them successfully terminate.

- When a specified number of successful completions is reached, the task (ie, Job) is complete. Deleting a Job will clean up the Pods it created.

**A Sample Job**

```
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
  ```

### Parallel Jobs

There are three main types of task suitable to run as a Job:

**Non-parallel Jobs**
- Normally, only one Pod is started, unless the Pod fails.

- The Job is complete as soon as its Pod terminates successfully.

- For a non-parallel Job, you can leave both .spec.completions and .spec.parallelism unset. When both are unset, both are defaulted to 1.


**Parallel Jobs with a fixed completion count:**
- Specify a non-zero positive value for .spec.completions.

- The Job represents the overall task, and is complete when there is one successful Pod for each value in the range 1 to .spec.completions.

- For a fixed completion count Job, you should set .spec.completions to the number of completions needed. You can set .spec.parallelism, or leave it unset and it will default to 1.

**Parallel Job**

```
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-job
spec:
  parallelism: 3
  template:
    spec:
      containers:
      - name: parallel-job
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```

**Parallel Job with Fixed Completion Count**

```
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-fixed-count-job
spec:
  completions: 4
  template:
    spec:
      containers:
      - name: parallel-fixed-count-job
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```
