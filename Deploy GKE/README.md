## Deploy GKE clusters
Using the GCP Console and Cloud Shell to deploy GKE clusters.

Use the GCP Console to deploy a GKE cluster
In the GCP Console, on the Navigation menu (9a951fa6d60a98a5.png), click Kubernetes Engine > Clusters

Click Create cluster to begin creating a GKE cluster.

Examine the console UI and the controls to change the cluster name, the cluster location, Kubernetes version, the number of nodes, and the node resources such as the machine type in the default node pool.

Clusters can be created across a region or in a single zone. A single zone is the default. When you deploy across a region the nodes are deployed to three separate zones and the total number of nodes deployed will be three times higher.

Change the cluster name to standard-cluster-1 and zone to us-central1-a. Leave all the values at their defaults and click Create.
The cluster begins provisioning.

Note: You need to wait a few minutes for the cluster deployment to complete.
When provisioning is complete, view the Kubernetes Engine > Clusters page 


Click the cluster name standard-cluster-1 to view the cluster details

You can scroll down the page to view more details.

Click the Storage and Nodes tabs under the cluster name (standard-cluster-1) at the top to view more of the cluster details.

## Modify GKE clusters
It is easy to modify many of the parameters of existing clusters using either the GCP Console or Cloud Shell. Use the GCP Console to modify the size of GKE clusters.

In the GCP Console, click Edit at the top of the details page for standard-cluster-1.
Scroll down to the Node Pools section and click default pool.
In the GCP Console, click Edit at the top of the details page.
In the Size section, change the number of nodes from 3 to 4.

Scroll to the bottom and click Save.
In the GCP Console, on the Navigation menu (9a951fa6d60a98a5.png), click Kubernetes Engine > Clusters.
When the operation completes, the Kubernetes Engine > Clusters page should show that standard-cluster-1 now has four nodes.


## Deploy a sample workload
Using the GCP console you will deploy a Pod running the nginx web server as a sample workload.

In the GCP Console, on the Navigation menu( Navigation menu), click Kubernetes Engine > Workloads.
Click Deploy to show the Create a deployment wizard.

Click Continue to accept the default container image, nginx.latest, which deploys a Pod with a single container running the latest version of nginx.

Scroll to the bottom of the window and click the Deploy button leaving the Configuration details at the defaults.
When the deployment completes your screen will refresh to show the details of your new nginx deployment.


## View details about workloads in the GCP Console
You shall view details of your GKE workloads directly in the GCP Console.

In the GCP Console, on the Navigation menu (Navigation menu), click Kubernetes Engine > Workloads.

You may see Pods (3/3) as the default deployment will start with three pods but will scale back to 1 after a few minutes. You can continue with the lab.
In the GCP Console, on the Kubernetes Engine > Workloads page, click nginx-1.
This displays the overview information for the workload showing details like resource utilization charts, links to logs, and details of the Pods associated with this workload.


In the GCP Console, click the Details tab for the nginx-1 workload. The Details tab shows more details about the workload including the Pod specification, number and status of Pod replicas and details about the horizontal Pod autoscaler.

Click the Revision History tab. This displays a list of the revisions that have been made to this workload.

Click the Events tab. This tab lists events associated with this workload.

And then the YAML tab. This tab provides the complete YAML file that defines this components and full configuration of this sample workload.

Still in the GCP Console's Details tab for the nginx-1 workload, click the Overview tab, scroll down to the Managed Pods section and click the name of one of the Pods to view the details page for that Pod.

Note:

The default deployment will start with three pods but will scale back to 1 after a few minutes so you may need to refresh the Overview page to make sure you have a valid Pod to inspect.

The Pod Details page provides information on the Pod configuration and resource utilization and the node where the Pod is running.

In the Pod details page, you can click the Events and Logs tabs to view event details and links to container logs in Cloud Operations.



Click the YAML tab to view the detailed YAML file for the Pod configuration.


