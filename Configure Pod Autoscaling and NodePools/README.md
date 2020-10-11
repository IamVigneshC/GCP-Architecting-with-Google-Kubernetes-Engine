
## Task 1. Connect to the lab GKE cluster and deploy a sample workload
In this task, you connect to the lab GKE cluster and create a deployment manifest for a set of Pods within the cluster.

Connect to the lab GKE cluster
In Cloud Shell, type the following command to set the environment variable for the zone and cluster name.

export my_zone=us-central1-a
export my_cluster=standard-cluster-1

Configure tab completion for the kubectl command-line tool.

source <(kubectl completion bash)

Configure access to your cluster for kubectl:

gcloud container clusters get-credentials $my_cluster --zone $my_zone

Deploy a sample web application to your GKE cluster
You will deploy a sample application to your cluster using the web.yaml deployment file that has been created for you:

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      run: web
  template:
    metadata:
      labels:
        run: web
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: web
        ports:
        - containerPort: 8080
          protocol: TCP

This manifest creates a deployment using a sample web application container image that listens on an HTTP server on port 8080.

In Cloud Shell enter the following command to clone the repository to the lab Cloud Shell.

git clone https://github.com/GoogleCloudPlatform/training-data-analyst

Create a soft link as a shortcut to the working directory.

ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s

Change to the directory that contains the sample files for this lab.

cd ~/ak8s/Autoscaling/

To create a deployment from this file, execute the following command:

kubectl create -f web.yaml --save-config

Create a service resource of type NodePort on port 8080 for the web deployment.

kubectl expose deployment web --target-port=8080 --type=NodePort

Verify that the service was created and that a node port was allocated:

kubectl get service web

Your IP address and port number might be different from the example output.

Output (do not copy)

NAME      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
web       NodePort   10.11.246.185   <none>        8080:32056/TCP   12s


## Task 2. Configure autoscaling on the cluster
In this task, you configure the cluster to automatically scale the sample application that you deployed earlier.

Configure autoscaling
Get the list of deployments to determine whether your sample web application is still running.

kubectl get deployment

The output should look like the example.

Output (do not copy)

NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    1/1     1            1           5m48s

If the web deployment of your application is not displayed, return to task 1 and redeploy it to the cluster.

To configure your sample application for autoscaling (and to set the maximum number of replicas to four and the minimum to one, with a CPU utilization target of 1%), execute the following command:

kubectl autoscale deployment web --max 4 --min 1 --cpu-percent 1

When you use kubectl autoscale, you specify a maximum and minimum number of replicas for your application, as well as a CPU utilization target.

Get the list of deployments to verify that there is still only one deployment of the web application.

kubectl get deployment

Output (do not copy)

NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    1/1     1            1           8m21s

Inspect the HorizontalPodAutoscaler object
The kubectl autoscale command you used in the previous task creates a HorizontalPodAutoscaler object that targets a specified resource, called the scale target, and scales it as needed. The autoscaler periodically adjusts the number of replicas of the scale target to match the average CPU utilization that you specify when creating the autoscaler.

To get the list of HorizontalPodAutoscaler resources, execute the following command:

kubectl get hpa

The output should look like the example.

Output (do not copy)

NAME      REFERENCE        TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
web       Deployment/web   0%/1%     1         4         1          1m

To inspect the configuration of HorizontalPodAutoscaler in YAML form, execute the following command:

kubectl describe horizontalpodautoscaler web

The output should look like the example.

Output (do not copy)

Name:                                                  web
Namespace:                                             default
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Tue, 08 Sep 2020...
Reference:                                             Deployment/web
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  0% (0) / 1%
Min replicas:                                          1
Max replicas:                                          4
Deployment pods:                                       1 current / 1 desired
Conditions:
  Type            Status  Reason               Message
  ----            ------  ------               -------
  AbleToScale     True    ScaleDownStabilized  recent recommendations [...]
  ScalingActive   True    ValidMetricFound     the HPA was able to [...]
  ScalingLimited  False   DesiredWithinRange   the desired count [...]
Events:           <none>

To view the configuration of HorizontalPodAutoscaler in YAML form, execute the following command:

kubectl get horizontalpodautoscaler web -o yaml

The output should look like the example.

Output (do not copy)

apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  annotations:
    autoscaling.alpha.kubernetes.io/conditions:  [...]   autoscaling.alpha.kubernetes.io/current-metrics: [...]
  creationTimestamp: 2018-11-14T02:59:28Z
  name: web
  namespace: default
  resourceVersion: "14588"
  selfLink: /apis/autoscaling/v1/namespaces/[...]
spec:
  maxReplicas: 4
  minReplicas: 1
  scaleTargetRef:
    apiVersion: extensions/v1beta1
    kind: Deployment
    name: web
  targetCPUUtilizationPercentage: 1
status:
  currentCPUUtilizationPercentage: 0
  currentReplicas: 1
  desiredReplicas: 1

Test the autoscale configuration
You need to create a heavy load on the web application to force it to scale out. You create a configuration file that defines a deployment of four containers that run an infinite loop of HTTP queries against the sample application web server.

You create the load on your web application by deploying the loadgen application using the loadgen.yaml file that has been provided for you.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: loadgen
spec:
  replicas: 4
  selector:
    matchLabels:
      app: loadgen
  template:
    metadata:
      labels:
        app: loadgen
    spec:
      containers:
      - name: loadgen
        image: k8s.gcr.io/busybox
        args:
        - /bin/sh
        - -c
        - while true; do wget -q -O- http://web:8080; done

To deploy this container, execute the following command:

kubectl apply -f loadgen.yaml

After you deploy this manifest, the web Pod should begin to scale.

Click Check my progress to verify the objective.
Deploying the loadgen application

Get the list of deployments to verify that the load generator is running.

kubectl get deployment

The output should look like the example.

Output (do not copy)

NAME      READY   UP-TO-DATE   AVAILABLE   AGE
loadgen   4/4     4            4           26s
web       1/1     1            1           14m

Inspect HorizontalPodAutoscaler.

kubectl get hpa

Once the loadgen Pod starts to generate traffic, the web deployment CPU utilization begins to increase. In the example output, the targets are now at 35% CPU utilization compared to the 1% CPU threshold.

Output (do not copy)

NAME      REFERENCE        TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
web       Deployment/web   35%/1%    1         4         1          8m

After a few minutes, inspect the HorizontalPodAutoscaler again.

kubectl get hpa

The autoscaler has increased the web deployment to four replicas.

Output (do not copy)

NAME      REFERENCE        TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
web       Deployment/web   88%/1%   1         4         4          9m

To stop the load on the web application, scale the loadgen deployment to zero replicas.

kubectl scale deployment loadgen --replicas 0

Get the list of deployments to verify that loadgen has scaled down.

kubectl get deployment

The loadgen deployment should have zero replicas.

Output (do not copy)

NAME      READY   UP-TO-DATE   AVAILABLE   AGE
loadgen   0/0     0            0           3m40s
web       2/4     4            2           18m
web       2/4     4            2           18m

Note: You need to wait 2 to 3 minutes before you list the deployments again.
Get the list of deployments to verify that the web application has scaled down to the minimum value of 1 replica that you configured when you deployed the autoscaler.

kubectl get deployment

You should now have one deployment of the web application.

Output (do not copy)

NAME      READY   UP-TO-DATE   AVAILABLE   AGE
loadgen   0/0     0            0           6m28s
web       4/4     4            4           20m

Task 3. Manage node pools
In this task, you create a new pool of nodes using preemptible instances, and then you constrain the web deployment to run only on the preemptible nodes.

Add a node pool
To deploy a new node pool with three preemptible VM instances, execute the following command:

gcloud container node-pools create "temp-pool-1" \
--cluster=$my_cluster --zone=$my_zone \
--num-nodes "2" --node-labels=temp=true --preemptible

If you receive an error that no preemptible instances are available you can remove the --preemptible option to proceed with the lab.

Get the list of nodes to verify that the new nodes are ready.

kubectl get nodes

You should now have 4 nodes.

Your names will be different from the example output.

Output (do not copy)

NAME                                       STATUS   ROLES    AGE   VERSION
gke-standard-cluster-1-default-pool...xc   Ready    <none>   33m   v1.15.12-gke.2
gke-standard-cluster-1-default-pool...q8   Ready    <none>   33m   v1.15.12-gke.2
gke-standard-cluster-1-temp-pool-1-...vj   Ready    <none>   32s   v1.15.12-gke.2
gke-standard-cluster-1-temp-pool-1-...xj   Ready    <none>   37s   v1.15.12-gke.2

All the nodes that you added have the temp=true label because you set that label when you created the node-pool. This label makes it easier to locate and configure these nodes.

To list only the nodes with the temp=true label, execute the following command:

kubectl get nodes -l temp=true

You should see only the two nodes that you added.

Your names will be different from the example output.

Output (do not copy)

NAME                                       STATUS   ROLES    AGE     VERSION
gke-standard-cluster-1-temp-pool-1-...vj   Ready    <none>   3m26s   v1.15.12-gke.2
gke-standard-cluster-1-temp-pool-1-...xj   Ready    <none>   3m31s   v1.15.12-gke.2

Control scheduling with taints and tolerations
To prevent the scheduler from running a Pod on the temporary nodes, you add a taint to each of the nodes in the temp pool. Taints are implemented as a key-value pair with an effect (such as NoExecute) that determines whether Pods can run on a certain node. Only nodes that are configured to tolerate the key-value of the taint are scheduled to run on these nodes.

To add a taint to each of the newly created nodes, execute the following command.
You can use the temp=true label to apply this change across all the new nodes simultaneously.

kubectl taint node -l temp=true nodetype=preemptible:NoExecute

To allow application Pods to execute on these tainted nodes, you must add a tolerations key to the deployment configuration.

Edit the web.yaml file to add the following key in the template's spec section:

tolerations:
- key: "nodetype"
  operator: Equal
  value: "preemptible"

The spec section of the file should look like the example.

...
    spec:
      tolerations:
      - key: "nodetype"
        operator: Equal
        value: "preemptible"
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: web
        ports:
        - containerPort: 8080
          protocol: TCP

To force the web deployment to use the new node-pool add a nodeSelector key in the template's spec section. This is parallel to the tolerations key you just added.

     nodeSelector:
        temp: "true"

Note: GKE adds a custom label to each node called cloud.google.com/gke-nodepool that contains the name of the node-pool that the node belongs to. This key can also be used as part of a nodeSelector to ensure Pods are only deployed to suitable nodes.

The full web.yaml deployment should now look as follows.

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      run: web
  template:
    metadata:
      labels:
        run: web
    spec:
      tolerations:
      - key: "nodetype"
        operator: Equal
        value: "preemptible"
      nodeSelector:
        temp: "true"
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: web
        ports:
        - containerPort: 8080
          protocol: TCP

To apply this change, execute the following command:

kubectl apply -f web.yaml

If you have problems editing this file successfully you can use the pre-prepared sample file called web-tolerations.yaml instead.

Click Check my progress to verify the objective.
Manage node pools

Get the list of Pods.

kubectl get pods

Your names might be different from the example output.

Output (do not copy)

NAME                   READY     STATUS    RESTARTS   AGE
web-7cb566bccd-pkfst   1/1       Running   0          1m

To confirm the change, inspect the running web Pod(s) using the following command

kubectl describe pods -l run=web

A Tolerations section with nodetype=preemptible in the list should appear near the bottom of the (truncated) output.

Output (do not copy)

<SNIP>

Node-Selectors:  temp=true
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
                 nodetype=preemptible
Events:

<SNIP>

The output confirms that the Pods will tolerate the taint value on the new preemptible nodes, and thus that they can be scheduled to execute on those nodes.

To force the web application to scale out again scale the loadgen deployment back to four replicas.

kubectl scale deployment loadgen --replicas 4

You could scale just the web application directly but using the loadgen app will allow you to see how the different taint, toleration and nodeSelector settings that apply to the web and loadgen applications affect which nodes they are scheduled on.

Get the list of Pods using thewide output format to show the nodes running the Pods

kubectl get pods -o wide

This shows that the loadgen app is running only on default-pool nodes while the web app is running only the preemptible nodes in temp-pool-1.

The taint setting prevents Pods from running on the preemptible nodes so the loadgen application only runs on the default pool. The toleration setting allows the web application to run on the preemptible nodes and the nodeSelector forces the web application Pods to run on those nodes.

NAME        READY STATUS    [...]         NODE
Loadgen-x0  1/1   Running   [...]         gke-xx-default-pool-y0
loadgen-x1  1/1   Running   [...]         gke-xx-default-pool-y2
loadgen-x3  1/1   Running   [...]         gke-xx-default-pool-y3
loadgen-x4  1/1   Running   [...]         gke-xx-default-pool-y4
web-x1      1/1   Running   [...]         gke-xx-temp-pool-1-z1
web-x2      1/1   Running   [...]         gke-xx-temp-pool-1-z2
web-x3      1/1   Running   [...]         gke-xx-temp-pool-1-z3
web-x4      1/1   Running   [...]         gke-xx-temp-pool-1-z4
