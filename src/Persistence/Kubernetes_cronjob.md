### Kubernetes cronjob
Kubernetes cronjobs are fundamentally the same as linux cronjobs that can be deployed with Kubernetes manifests. They perform actions on a schedule denoted by the contab syntax. [Crontab Guru](https://crontab.guru/) is a great resource for getting the cron schedule you want.  

Like every other object in Kubernetes, you declare your cronjob in a manifest and then submit it to the API server using `kubectl apply -f <cronjob_name>.yaml`. When creating a cronjob, you must specify what container image you want your cronjob to run inside of. 

```yaml
# Modified From https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/
apiVersion: batch/v1
kind: CronJob
metadata:
  name: anacronda
spec:
  schedule: "* * * * *" # This means run every 60 seconds
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: anacronda 
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

![](../images/Pasted%20image%2020240321160119.png)

# Defending
> Pull requests needed ❤️ 