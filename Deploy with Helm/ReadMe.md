## Task 1. Use Helm charts to deploy a solution on Kubernetes Engine
Typical Kubernetes systems consist of many different resources. Though you could maintain configuration files for each of these resources, managing versions of different resources and compatibility among those versions can be difficult. Helm is a popular tool used with Kubernetes for packaging resources.

In this task, you use Helm charts to deploy Redis, an open source in memory key-value database service, on a Kubernetes Engine cluster.

Connect to the lab GKE cluster
In Cloud Shell, type the following command to set the environment variable for the zone and cluster name.

`export my_zone=us-central1-a`
`export my_cluster=standard-cluster-1`
  
Configure kubectl tab completion in Cloud Shell.

`source <(kubectl completion bash)`
In Cloud Shell, configure access to your cluster for the kubectl command-line tool, using the following command:

gcloud container clusters get-credentials $my_cluster --zone $my_zone
Download the Helm binary, deploy a Helm Chart
You download helm, configure admin and service accounts and then initialize Helm for your Kubernetes cluster.

Execute the following command to download the Helm installation shell script.

curl -LO https://git.io/get_helm.sh
Note: You can find information on this, and other methods, for installing Helm on the Helm installation page, https://helm.sh/docs/using_helm/#installing-helm.

Make the installation script executable.

chmod 700 get_helm.sh
Execute the Helm installation script.

./get_helm.sh
Ensure your user account has the cluster-admin role in your cluster.

kubectl create clusterrolebinding user-admin-binding \
   --clusterrole=cluster-admin \
   --user=$(gcloud config get-value account)
Create a Kubernetes service account called Tiller. this will be used by Tiller, the server side of Helm, for deploying Helm charts.

kubectl create serviceaccount tiller --namespace kube-system
Grant the Tiller service account the cluster-admin role in your cluster:

kubectl create clusterrolebinding tiller-admin-binding \
   --clusterrole=cluster-admin \
   --serviceaccount=kube-system:tiller
Execute the following commands to initialize Helm using the Tiller service account.

helm init --service-account=tiller
Click Check my progress to verify the objective.
Create a tiller service account and initialize the Helm

Execute the following commands to update the Helm repositories.

helm repo update
In Cloud Shell, execute the following command to verify the Helm installation and configuration:

helm version
The output should look like the example, although you will see a newer version.

Output (do not copy)

Client: &version...{SemVer:"v2.16.10", GitCommit:"be3...State:"clean"}
Server: &version...{SemVer:"v2.16.10", GitCommit:"be3...State:"clean"}


Execute the following command to deploy a set of resources to create a Redis service on the active context cluster:

helm install stable/redis
A Helm chart is a package of resource configuration files, along with configurable parameters. This single command deployed a collection of resources.

Task 2. Validate and test a solution deployed using Helm
In this task you use kubectl to validate that Helm has deployed the resource components for the Redis application and you then use Redis to store and retrieve sample data to confirm that it has been successfully deployed.

Use kubectl to inspect the Kubernetes resources deployed via Helm
You use kubectl to list the Kubernetes Services, StatefulSets, ConfigMaps and Secrets that were deployed using Helm.

A Kubernetes Service defines a set of Pods and a stable endpoint by which network traffic can access them. In Cloud Shell, execute the following command to view Services that were deployed through the Helm chart:

kubectl get services
The output should look like the example.

Output (do not copy)

NAME                           TYPE       CLUSTER-IP    EXTERNAL-IP  PORT(S)   AGE
contrasting-...redis-headless  ClusterIP  None          <none>       6379/TCP  26s
contrasting-...redis-master    ClusterIP  10.11.244.2   <none>       6379/TCP  26s
contrasting-...redis-slave     ClusterIP  10.11.240.101 <none>       6379/TCP  26s
kubernetes                     ClusterIP  10.11.240.1   <none>       443/TCP   10m

A Kubernetes StatefulSet manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods. In Cloud Shell, execute the following commands to view a StatefulSet that was deployed through the Helm chart:

kubectl get statefulsets
The output should look like the example.

Output (do not copy)

NAME                                       Ready   AGE
contrasting-waterbuffalo-redis-master      1/1     1m
contrasting-waterbuffalo-redis-slave       2/2     1m

A Kubernetes ConfigMap lets you store and manage configuration artifacts, so that they are decoupled from container-image content. In Cloud Shell, execute the following commands to view ConfigMaps that were deployed through the Helm chart:

kubectl get configmaps
The output should look like the example.

Output (do not copy)

NAME                                    DATA      AGE
contrasting-waterbuffalo-redis           3         2m
contrasting-waterbuffalo-redis-health    3         2m

A Kubernetes Secret, like a ConfigMap, lets you store and manage configuration artifacts, but it's specially intended for sensitive information such as passwords and authorization keys. In Cloud Shell, execute the following commands to view some of the Secret that was deployed through the Helm chart:

kubectl get secrets
The output should look like the example.

Output (do not copy)

NAME                            TYPE                               DATA    AGE
contrasting-waterbuffalo-redis  Opaque                               1      3m
default-token-vl8bh             kubernetes.io/service-account-token  3      12m

Click Check my progress to verify the objective.
Install redis and check resources- services, secrets,deployments and configmaps

You can inspect the Helm chart directly using the following command:

helm inspect stable/redis
If you want to see the templates that the Helm chart deploys you can use the following command:

helm install stable/redis --dry-run --debug
Test Redis functionality
You store and retrieve values in the new Redis deployment running in your Kubernetes Engine cluster.

Execute the following command in the Cloud Shell to store the service ip-address for the Redis cluster in an environment variable.

export REDIS_IP=$(kubectl get services -l app=redis -o json | jq -r '.items[].spec | select(.selector.role=="master")' | jq -r '.clusterIP')
Retrieve the Redis password and store it in an environment variable.

export REDIS_PW=$(kubectl get secret -l app=redis -o jsonpath="{.items[0].data.redis-password}"  | base64 --decode)
Display the Redis cluster address and password.

echo Redis Cluster Address : $REDIS_IP
echo Redis auth password   : $REDIS_PW
Open an interactive shell to a temporary Pod running inside your GKE cluster, passing in the Redis cluster address and password as environment variables.

kubectl run redis-test --rm --tty -i --restart='Never' \
    --env REDIS_PW=$REDIS_PW \
    --env REDIS_IP=$REDIS_IP \
    --image docker.io/bitnami/redis:4.0.12 -- bash
Enter the following command in the interactive shell to connect to the Redis cluster using the redis-cli.

redis-cli -h $REDIS_IP -a $REDIS_PW
In the redis cli set a key value.

set mykey this_amazing_value
This will display OK if successful.

In the redis cli retrieve the key value.

get mykey
This will return the value you stored indicating that the Redis cluster can successfully store and retrieve data.
