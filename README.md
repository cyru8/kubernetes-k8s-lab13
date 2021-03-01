# kubernetes-k8s-lab13

This scenario explores the basic techniques for observing the state of Kubernetes using metrics.

In the following steps you will learn how to:

    access metrics information produced by each cAdvisor in each Kubelet,
    inspect resources using the Resource Metrics API,
    viewing metrics reported by Metrics Server.

######

Your Kubernetes Cluster

For this scenario, Katacoda has just started a fresh Kubernetes cluster for you. Verify it's ready for your use.

kubectl version --short && \
kubectl get componentstatus && \
kubectl get nodes && \
kubectl cluster-info

The Helm {https://helm.sh/}  package manager used for installing applications on Kubernetes is also available.

helm version --short

Kubernetes Dashboard

You can administer your cluster with the kubectl CLI tool or use the visual Kubernetes Dashboard. Use this script to access the protected Dashboard.

token.sh

######
Sample Application

Before exploring observability topics, start a small application to provide something to observe.

Run 3 instances of the random-logger container to start generating continuously random logging events.

kubectl create deployment random-logger --image=chentex/random-logger

Scale to 3 instances.

kubectl scale deployment/random-logger --replicas=3

The 3 pods will start shortly.

kubectl get pods

    Thank you to Vicente Zepeda for providing this beautifully simple container. The source code is here <<https://github.com/chentex/random-logger>>.

######

Resource Inspection
General Inspection of a Cluster

When you first start interacting with a running cluster there are a few commands to help you get oriented with its health and state.

Inspect the cluster general state.

kubectl cluster-info

Inspect this Kubernetes cluster only Worker node.

kubectl describe node node01

For deeper details, you can generate a complete dump of the cluster state to a directory.

kubectl cluster-info dump --all-namespaces --output-directory=cluster-state --output=json

This creates a directory where each file is a report on all the nodes and namespaces.

tree cluster-state

There is a wealth of information you can mine.

    Show me all the container images names in the kube-system namespace. jq '.items[].status.containerStatuses[].image' cluster-state/kube-system/pods.json

    Show me when all the container images were started in the default namespace. jq '.items[].status.containerStatuses[] | [.image, .state[].startedAt]' cluster-state/default/pods.json

General Inspection for a Deployment

The running state of an application can be observed through a variety of kubectl describe commands across various resources.

Inspect the last deployment.

kubectl describe deployment random-logger

Specifically, the replica state.

kubectl describe deployments | grep "Replicas:"

Inspect the 3 pods.

kubectl get pods

kubectl describe pods
Events

Kubernetes also maintains a history of events.

kubectl get events

Scaling is a type of event. Scale down the Pod from 3 down to 2.

kubectl scale deployment/random-logger --replicas=2

Notice the last event will reflect the scaling request.

kubectl get events --sort-by=.metadata.creationTimestamp

These events are not to be confused with security audit logs which are also recorded.
Inspecting Containers

You can also typically get into a running container and inspect it as well. Get the name of the first Pod.

POD=$(kubectl get pod  -o jsonpath="{.items[0].metadata.name}")

Inspect the script contents inside the container file system.

kubectl exec $POD -- cat entrypoint.sh

Or, shell into the container.

kubectl exec -it $POD -- /bin/sh

and come back out with the exit command.

There is a wealth of helpful Linux commands to give you information about the Linux containers. Here are just a few.

kubectl exec $POD -- uptime

kubectl exec $POD -- ps

kubectl exec $POD -- stat -f /

kubectl exec $POD --container random-logger -- lsof

kubectl exec $POD --container random-logger -- iostat

When the Pod has more than one container, the specific container

kubectl exec $POD --container random-logger -- ls -a -l
######



