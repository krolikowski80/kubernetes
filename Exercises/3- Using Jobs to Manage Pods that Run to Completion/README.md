# Using Jobs to Manage Pods that Run to Completion

Instructions
1. Create a Namespace for the resources you will create in this lab step and change your default kubectl context to use the Namespace:

Copy code
1
2
3
4
# Create namespace
kubectl create namespace jobs
# Set namespace as the default for the current context
kubectl config set-context $(kubectl config current-context) --namespace=jobs
alt

alt

 

2. Create a Job named one-off that sleeps for 30 seconds:

Copy code
1
kubectl create job one-off --image=alpine -- sleep 30
alt

The Job will immediately start running a single Pod by default.

 

3. Read through the spec of the Job to see what other fields can be configured:

Copy code
1
kubectl get jobs one-off -o yaml | more
Some important fields to highlight are:

backoffLimit: Number of times a Job will retry before marking a Job as failed
completions: Number of Pod completions the Job needs before being considered a success
parallelism: Number of Pods the Job is allowed to run in parallel
spec.template.spec.restartPolicy: Job Pods default to never attempting to restart. Instead, the Job is responsible for managing the restart of failed Pods.
Also, note the Job uses a selector to track its Pods.

 

4. Use explain to see what other Job spec fields can be specified:

Copy code
1
kubectl explain job.spec | more
alt

The activeDeadlineSeconds and ttlSecondsAfterFinished are useful for automatically terminating and deleting Jobs.

 

5. Create a Job that has a Pod that always fails:

Copy code
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
cat << 'EOF' > pod-fail.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pod-fail
spec:
  backoffLimit: 3
  completions: 6
  parallelism: 2
  template:
    spec:
      containers:
      - image: alpine
        name: fail
        command: ['sleep 20 && exit 1']
      restartPolicy: Never
EOF
kubectl create -f pod-fail.yaml
alt

The Pod will always fail after sleeping for 20 seconds due to the exit 1 command (returning a non-zero exit code is treated as a failure). The Job allows for two Pods to run in parallel.

 

6. Watch the describe output for the Job to see how the Job progresses:

Copy code
1
watch kubectl describe jobs pod-fail
alt

Notice the Pod Statuses shows 2 Running Pods and there is always 0 Succeeded. In the Events, you can see the job-controller automatically provisions new Pods as prior Pods fail. Eventually, the following event appears indicating the backoff limit was exceeded, and the Job stops retrying:

alt

 

7. Press ctrl+c to stop watching the output.

 

8. Get the Pods in the jobs namespace:

Copy code
1
kubectl get pods
alt

All of the Pods associated with the Jobs you ran are listed. The Pods will remain until you delete them or the Job associated with them. Setting a Job's ttlSecondsAfterFinished can free you from manually cleaning up the Pods.

 

9. Create a CronJob that runs a Job every minute:

Copy code
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
cat << 'EOF' > cronjob-example.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cronjob-example
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - image: alpine
            name: fail
            command: ['date']
          restartPolicy: Never
EOF
kubectl create -f cronjob-example.yaml
alt

The CronJob spec is mainly a schedule with a template for creating Jobs (jobTemplate). There are a few other fields though. For more information about CronJob fields, issue kubectl explain cronjob.spec. To learn more about the cron schedule syntax, see here.

 

10. Observe the Events of the CronJob to confirm that it is creating Jobs every minute: 

Copy code
1
watch kubectl describe cronjob cronjob-example
alt

