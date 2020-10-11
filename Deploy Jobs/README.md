In GKE, a Job is a controller object that represents a finite task. Jobs manage a task as it runs to completion, rather than managing an ongoing desired state such as the maintaining the total number of running Pods.

CronJobs perform finite, time-related tasks that run once or repeatedly at a time that you specify using Job objects to complete their tasks.

## Define and deploy a Job manifest
In GKE, a Job is a controller object that represents a finite task.

Create a Job, inspect its status, and then remove it.

In Cloud Shell, type the following command to set the environment variable for the zone and cluster name.

`export my_zone=us-central1-a`
`export my_cluster=standard-cluster-1`

Configure kubectl tab completion in Cloud Shell.

`source <(kubectl completion bash)`

In Cloud Shell, configure access to your cluster for the kubectl command-line tool, using the following command:

`gcloud container clusters get-credentials $my_cluster --zone $my_zone`

In Cloud Shell enter the following command to clone the repository to the lab Cloud Shell.

`git clone https://github.com/GoogleCloudPlatform/training-data-analyst`

Create a soft link as a shortcut to the working directory.

`ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s`

Change to the directory that contains the sample files for this lab.

`cd ~/ak8s/Jobs_CronJobs`

### Create and run a Job
You will create a job using a sample deployment manifest called example-job.yaml that has been provided for you. This Job computes the value of Pi to 2,000 places and then prints the result.

apiVersion: batch/v1
kind: Job
metadata:
  #Unique key of the Job instance
  name: example-job
spec:
  template:
    metadata:
      name: example-job
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl"]
        args: ["-Mbignum=bpi", "-wle", "print bpi(2000)"]
      #Do not restart containers after they exit
      restartPolicy: Never

To create a Job from this file, execute the following command:

`kubectl apply -f example-job.yaml`


To check the status of this Job, execute the following command:

`kubectl describe job example-job`

You will see details of the job, including the Pod statuses indicating how many jobs are still running, how many completed successfully and how many failed.

Output

...
Start Time:     Thu, 20 Dec 2018 14:34:09 +0000
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
...

To view all Pod resources in your cluster, including Pods created by the Job which have completed, execute the following command:

`kubectl get pods`

Your Pod name might be different from the example output. Make a note of one of the Pod names.

Output

NAME                   READY     STATUS      RESTARTS   AGE
example-job-sqljc      0/1       Completed   0          1m

### Clean up and delete the Job
When a Job completes, the Job stops creating Pods. The Job API object is not removed when it completes, which allows you to view its status. Pods created by the Job are not deleted, but they are terminated. Retention of the Pods allows you to view their logs and to interact with them.

To get a list of the Jobs in the cluster, execute the following command:

`kubectl get jobs`

The output should look like the example.

Output

NAME           COMPLETIONS   DURATION       AGE
example-job    1/1           75s            2m5s  

To retrieve the log file from the Pod that ran the Job execute the following command. You must replace [POD-NAME] with the node name you recorded in the last task

`kubectl logs [POD-NAME]`

The output will show that the job wrote the first two thousand digits of pi to the Pod log.

To delete the Job, execute the following command:

`kubectl delete job example-job`

If you try to query the logs again the command will fail as the Pod can no longer be found.

## Define and deploy a CronJob manifest
You can create CronJobs to perform finite, time-related tasks that run once or repeatedly at a time that you specify.

In this task, you create and run a CronJob, and then you clean up and delete the Job.

### Create and run a CronJob
The CronJob manifest file example-cronjob.yaml has been provided for you. This CronJob deploys a new container every minute that prints the time, date and "Hello, World!".

apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo "Hello, World!"
          restartPolicy: OnFailure

Note

CronJobs use the required schedule field, which accepts a time in the Unix standard crontab format. All CronJob times are in UTC:

The first value indicates the minute (between 0 and 59).
The second value indicates the hour (between 0 and 23).
The third value indicates the day of the month (between 1 and 31).
The fourth value indicates the month (between 1 and 12).
The fifth value indicates the day of the week (between 0 and 6).
The schedule field also accepts * and ? as wildcard values. Combining / with ranges specifies that the task should repeat at a regular interval. In the example, */1 * * * * indicates that the task should repeat every minute of every day of every month.

To create a Job from this file, execute the following command:

`kubectl apply -f example-cronjob.yaml`


To get a list of the Jobs in the cluster, execute the following command:

`kubectl get jobs`

The output should look like the example.

Output

NAME               COMPLETIONS    DURATION     AGE
hello-1545013620   1/1            2s           18s

To check the status of this Job, execute the following command, where [job_name] is the name of your job:

`kubectl describe job [job_name]`

You will see details of the job, including the Pod statuses showing that one instance of this job was run.

Output

...
Start Time:     Thu, 20 Dec 2018 15:24:03 +0000
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
...
...Created pod: hello-1545319920-twkhl

Make a note of the name of the Pod that was used by this job.

View the output of the Job by querying the logs for the Pod. Replace [POD-NAME] with the name of the Pod you recorded in the last step.

`kubectl logs [POD-NAME]`

This will display the output of the shell script configured in the CronJob:

Output

Thu Dec 20 15:31:16 WET 2018
Hello,World!

To view all job resources in your cluster, including all of the Pods created by the CronJob which have completed, execute the following command:

`kubectl get jobs`

Your job names might be different from the example output. By default Kubernetes sets the Job history limits so that only the last three successful and last failed job are retained so this list will only contain the most recent three of four jobs.

Output

NAME               COMPLETIONS   DURATION     AGE
hello-1545013680   1             1            2m
hello-1545013740   1             1            1m
hello-1545013800   1             1            42s

### Clean up and delete the Job
In order to stop the CronJob and clean up the Jobs associated with it you must delete the CronJob.

To delete all these jobs, execute the following command:

`kubectl delete cronjob hello`

To verify that the jobs were deleted, execute the following command:

`kubectl get jobs`

The output should look like the example.

Output

No resources found.

All the Jobs were removed.
