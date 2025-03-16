## Jobs

```yml
# Job POD
apiVersion: v1
kind: Pod
metadata:
  name: math-pod
spec:
  containers:
    - name: math-operation
      image: ubuntu
      restartPolicy: Never
      command: ['expr', '3', '+', '2']
```

```yml
# Job
apiVersion: batch/v1
kind: Job
metadata:
  name: math-job
template:
  spec:
    completions: 3 # Default = 1. Jobs will be executed 1 by 1 until the number of 'completions' is achieved succesfully (with 'status' = 'completed')
    parallelism: 3 # Default = 1. It allows the parallel execution of several jobs, overwritting the 1 by 1 execution
    containers:
      - name: math-operation
        image: ubuntu
        restartPolicy: Never
        command: ['expr', '3', '+', '2']
```

```yml
# CronJob
apiVersion: batch/v1
kind: CronJob
metadata:
  name: busy-cronjob
spec:
  schedule: '*/1 * * * *' # Docs: https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/#writing-a-cronjob-spec
  jobTemplate:
    spec:
      completions: 1
      parallelism: 1
      template:
        spec:
          containers:
            - name: busy
              image: busybox:1.28
              imagePullPolicy: IfNotPresent
              command:
                - /bin/sh
                - -c
                - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

```bash
k create -f <job-definition-file>
k get pods
k logs <pod-name>
k delete job <job-name>

k run nginx --image=busybox:1.28 --restart=Always  # (pod)
k run nginx --image=busybox:1.28 --restart=Never  # (job)
k run nginx --image=busybox:1.28 --restart=OnFailure  # (job)
k run nginx --image=busybox:1.28 --restart=Never --schedule="* * * * *" # (cronJob)
k run nginx --image=busybox:1.28 --restart=OnFailure --schedule="* * * * *" # (cronJob)
```
