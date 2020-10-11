
# Creating Google Kubernetes Engine Deployments

• Create deployment manifests, deploy to cluster, and verify Pod rescheduling as nodes are disabled

• Trigger manual scaling up and down of Pods in deployments

• Trigger deployment rollout (rolling update to new version) and rollbacks

• Perform a Canary deployment


## Create deployment manifests and deploy to the cluster

You create a deployment manifest for a Pod inside the cluster.

Connect to the lab GKE cluster
In Cloud Shell, type the following command to set the environment variable for the zone and cluster name.

` export my_zone=us-central1-a`
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

`cd ~/ak8s/Deployments/`

Create a deployment manifest

You will create a deployment using a sample deployment manifest called nginx-deployment.yaml that has been provided for you. This deployment is configured to run three Pod replicas with a single nginx container in each Pod listening on TCP port 80.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80

To deploy your manifest, execute the following command:

`kubectl apply -f ./nginx-deployment.yaml`


To view a list of deployments, execute the following command:

`kubectl get deployments`

The output should look like this example.

NAME               READY      UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/3        3            0           3s

Wait a few seconds, and repeat the command until the number listed for CURRENT deployments reported by the command matches the number of DESIRED deployments.
The final output should look like the example.

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           42s

## Manually scale up and down the number of Pods in deployments
Sometimes, you want to shut down a Pod instance. Other times, you want ten Pods running. In Kubernetes, you can scale a specific Pod to the desired number of instances. To shut them down, you scale to zero.

You scale Pods up and down in the Google Cloud Console and Cloud Shell.

Scale Pods up and down in the console
Switch to the Google Cloud Console tab.
On the Navigation menu , click Kubernetes Engine > Workloads.
Click nginx-deployment (your deployment) to open the Deployment details page.
At the top, click ACTIONS > Scale.

Type 1 and click SCALE.
This action scales down your cluster. You should see the Pod status being updated under Managed Pods. You might have to click Refresh.

Scale Pods up and down in the shell
Switch back to the Cloud Shell browser tab.

In the Cloud Shell, to view a list of Pods in the deployments, execute the following command:

`kubectl get deployments`

Output

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           3m

To scale the Pod back up to three replicas, execute the following command:

`kubectl scale --replicas=3 deployment nginx-deployment`

To view a list of Pods in the deployments, execute the following command:

`kubectl get deployments`

Output

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           4m

## Trigger a deployment rollout and a deployment rollback

A deployment's rollout is triggered if and only if the deployment's Pod template (that is, .spec.template) is changed, for example, if the labels or container images of the template are updated. Other updates, such as scaling the deployment, do not trigger a rollout.

You trigger deployment rollout, and then you trigger deployment rollback.

Trigger a deployment rollout
To update the version of nginx in the deployment, execute the following command:

`kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record`

This updates the container image in your Deployment to nginx v1.9.1.


To view the rollout status, execute the following command:

`kubectl rollout status deployment.v1.apps/nginx-deployment`

The output should look like the example.

Output

Waiting for rollout to finish: 1 out of 3 new replicas updated...
Waiting for rollout to finish: 1 out of 3 new replicas updated...
Waiting for rollout to finish: 1 out of 3 new replicas updated...
Waiting for rollout to finish: 2 out of 3 new replicas updated...
Waiting for rollout to finish: 2 out of 3 new replicas updated...
Waiting for rollout to finish: 2 out of 3 new replicas updated...
Waiting for rollout to finish: 1 old replicas pending termination...
Waiting for rollout to finish: 1 old replicas pending termination...
deployment "nginx-deployment" successfully rolled out

To verify the change, get the list of deployments.

`kubectl get deployments`

The output should look like the example.

Output

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           6m

View the rollout history of the deployment.

`kubectl rollout history deployment nginx-deployment`

The output should look like the example. Your output might not be an exact match.

Output

deployments "nginx-deployment"
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record=true

Trigger a deployment rollback
To roll back an object's rollout, you can use the kubectl rollout undo command.

To roll back to the previous version of the nginx deployment, execute the following command:

`kubectl rollout undo deployments nginx-deployment`

View the updated rollout history of the deployment.

`kubectl rollout history deployment nginx-deployment`

The output should look like the example. Your output might not be an exact match.

Output

deployments "nginx-deployment"
REVISION  CHANGE-CAUSE
2         kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.9.1 --record=true
3         <none>

View the details of the latest deployment revision

`kubectl rollout history deployment/nginx-deployment --revision=3`

The output should look like the example. Your output might not be an exact match but it will show that the current revision has rolled back to nginx:1.7.9.

Output

deployments "nginx-deployment" with revision #3
Pod Template:
  Labels:       app=nginx
        pod-template-hash=3123191453
  Containers:
   nginx:
    Image:      nginx:1.7.9
    Port:       80/TCP
    Host Port:  0/TCP
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>

## Define the service type in the manifest

You create and verify a service that controls inbound traffic to an application. Services can be configured as ClusterIP, NodePort or LoadBalancer types. In this lab you configure a LoadBalancer.

Define service types in the manifest

A manifest file called service-nginx.yaml that deploys a LoadBalancer service type has been provided for you. This service is configured to distribute inbound traffic on TCP port 60000 to port 80 on any containers that have the label app: nginx.

apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 60000
    targetPort: 80

In the Cloud Shell, to deploy your manifest, execute the following command:

`kubectl apply -f ./service-nginx.yaml`

This manifest defines a service and applies it to Pods that correspond to the selector. In this case, the manifest is applied to the nginx container that you deployed in task 1. This service also applies to any other Pods with the app: nginx label, including any that are created after the service.


Verify the LoadBalancer creation
To view the details of the nginx service, execute the following command:

`kubectl get service nginx`

The output should look like the example.

Output
NAME      CLUSTER_IP      EXTERNAL_IP      PORT(S)   SELECTOR    AGE
nginx     10.X.X.X        X.X.X.X          60000/TCP    run=nginx   1m

When the external IP appears, open http://[EXTERNAL_IP]:60000/ in a new browser tab to see the server being served through network load balancing.
It may take a few seconds before the ExternalIP field is populated for your service. This is normal. Just re-run the kubectl get services nginx command every few seconds until the field is populated.

## Perform a canary deployment

A canary deployment is a separate deployment used to test a new version of your application. A single service targets both the canary and the normal deployments. And it can direct a subset of users to the canary version to mitigate the risk of new releases. The manifest file nginx-canary.yaml that is provided for you deploys a single pod running a newer version of nginx than your main deployment. In this task, you create a canary deployment using this new deployment file.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-canary
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        track: canary
        Version: 1.9.1
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.1
        ports:
        - containerPort: 80

The manifest for the nginx Service you deployed in the previous task uses a label selector to target the Pods with the app: nginx label. Both the normal deployment and this new canary deployment have the app: nginx label. Inbound connections will be distributed by the service to both the normal and canary deployment Pods. The canary deployment has fewer replicas (Pods) than the normal deployment, and thus it is available to fewer users than the normal deployment.

Create the canary deployment based on the configuration file.

`kubectl apply -f nginx-canary.yaml`


When the deployment is complete, verify that both the nginx and the nginx-canary deployments are present.

`kubectl get deployments`

Switch back to the browser tab that is connected to the external LoadBalancer service ip and refresh the page. You should continue to see the standard "Welcome to nginx" page.


![Image of DeployGKE](https://github.com/IamVigneshC/GCP-Architecting-with-Google-Kubernetes-Engine-/blob/main/Creating%20GKE%20Deployments/Deploy.PNG)


Switch back to the Cloud Shell and scale down the primary deployment to 0 replicas.

`kubectl scale --replicas=0 deployment nginx-deployment`

Verify that the only running replica is now the Canary deployment:

`kubectl get deployments`

Switch back to the browser tab that is connected to the external LoadBalancer service ip and refresh the page. You should continue to see the standard "Welcome to nginx" page showing that the Service is automatically balancing traffic to the canary deployment.

![Image of DeployGKE](https://github.com/IamVigneshC/GCP-Architecting-with-Google-Kubernetes-Engine-/blob/main/Creating%20GKE%20Deployments/Deploy.PNG)


Note: Session affinity
The Service configuration used in the lab does not ensure that all requests from a single client will always connect to the same Pod. Each request is treated separately and can connect to either the normal nginx deployment or to the nginx-canary deployment. This potential to switch between different versions may cause problems if there are significant changes in functionality in the canary release. To prevent this you can set the sessionAffinity field to ClientIP in the specification of the service if you need a client's first request to determine which Pod will be used for all subsequent connections.

For example:

apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: LoadBalancer
  sessionAffinity: ClientIP
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 60000
    targetPort: 80
