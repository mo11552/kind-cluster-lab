# Kubernetes Kind Cluster Lab

A beginner Kubernetes lab that runs locally using Kind and Docker Desktop.

## What I Practiced

- Creating a local Kubernetes cluster with Kind
- Deploying a standalone Nginx Pod
- Creating and scaling a Deployment
- Testing Kubernetes self-healing
- Exposing Pods through a ClusterIP Service
- Accessing the application using port forwarding

## Project Files

- `kind-config.yaml` — Defines the Kind cluster
- `pod.yaml` — Creates a standalone Nginx Pod
- `deployment.yaml` — Manages multiple Nginx Pods
- `service.yaml` — Provides one stable endpoint for the Pods

## Requirements

- Docker Desktop
- Kind
- kubectl

## Create the Cluster

```bash
kind create cluster --name kind-lab
```

## Deploy the Application

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

## Check the Resources

```bash
kubectl get nodes
kubectl get deployments
kubectl get pods
kubectl get services
```

## Access Nginx

```bash
kubectl port-forward service/nginx-service 8080:80
```

Open:

```text
http://localhost:8080
```

## Test Self-Healing

Delete one of the Pods:

```bash
kubectl delete pod <pod-name>
```

Kubernetes automatically creates a replacement because the Deployment maintains the desired number of replicas.

## Scale the Application

```bash
kubectl scale deployment nginx-deployment --replicas=3
```

## Clean Up

```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
kind delete cluster --name kind-lab
```


## Kubernetes API Resources Lab

This lab explores how Kubernetes resources are defined, validated, and managed through the API server.

### Concepts Practiced

- Kubernetes resource structure: `apiVersion`, `kind`, `metadata`, and `spec`
- Discovering resources with `kubectl api-resources`
- Exploring YAML fields with `kubectl explain`
- Namespaced versus cluster-wide resources
- Server-side YAML validation with `--dry-run=server`
- Creating and inspecting a standalone Nginx Pod

### Commands

```bash
kubectl api-resources
kubectl explain pod
kubectl explain pod.spec.containers
kubectl apply -f api-resources/pod.yaml
kubectl get pods
kubectl describe pod api-demo
kubectl apply --dry-run=server -f api-resources/pod.yaml


## Nginx ReplicaSet Lab

Created an Nginx ReplicaSet to practice Kubernetes workload management.

### What I practiced

- Created a dedicated namespace
- Defined a ReplicaSet using YAML
- Used matching labels and selectors
- Maintained three Nginx Pods
- Tested automatic Pod replacement
- Scaled the ReplicaSet from three to five Pods
- Reapplied the YAML to restore the desired state
- Used namespaces to isolate resources

### Apply the lab

```bash
kubectl apply -f replicasets/nginx-replicaset.yaml
kubectl get replicasets,pods -n replicaset-practice

## Init and Ephemeral Containers Lab

### Concepts practiced

- Created an init container that runs before the main container
- Used an `emptyDir` volume to pass a generated file to Nginx
- Confirmed the init container finished with `Completed`
- Used an ephemeral BusyBox container to troubleshoot a running Pod
- Accessed Nginx from the ephemeral container through the Pod network

### Lab file

- `init-containers/init-demo.yaml`

### Commands

```bash
kubectl apply -f init-containers/init-demo.yaml
kubectl get pod init-demo
kubectl exec init-demo -c web-server -- cat /usr/share/nginx/html/index.html
kubectl debug init-demo --image=busybox:1.36 --target=web-server -- sh -c 'wget -qO- http://127.0.0.1'
```


## Kubernetes Resource Management Lab

### Concepts practiced

- Viewed a Kubernetes node and its internal IP address
- Set CPU and memory requests and limits
- Compared Guaranteed, Burstable, and BestEffort Pods
- Checked the QoS class assigned by Kubernetes
- Added a label to a running Pod
- Selected a Pod using its label

### Lab file

- `resource-management/qos-pods.yaml`

### Commands

```bash
kubectl apply -f resource-management/qos-pods.yaml
kubectl get pods -l qos-demo
kubectl get pods -l qos-demo -o custom-columns=NAME:.metadata.name,QOS-CLASS:.status.qosClass
kubectl label pod burstable-pod environment=training
kubectl get pods -l environment=training
```

---

## Kubernetes Controllers Assignment — 09/23

### ReplicaSet

A ReplicaSet maintains a specified number of identical Pods. The ReplicaSet’s selector must match the labels in the Pod template so it knows which Pods to manage.

Tasks completed:

- Created an `nginx` namespace
- Created an Nginx ReplicaSet with three replicas
- Scaled it to four replicas with `kubectl`
- Updated the manifest to seven replicas and reapplied it
- Confirmed the Pods were owned by the ReplicaSet
- Deleted the ReplicaSet and its managed Pods

### Deployment

A Deployment manages ReplicaSets, while each ReplicaSet manages its Pods. When the container image changes, the Deployment creates a new ReplicaSet and gradually replaces the old Pods.

Tasks completed:

- Created an Nginx Deployment with two replicas
- Confirmed the `Available` and `Progressing` conditions
- Scaled the Deployment to seven Pods and then down to five
- Updated Nginx from `1.20.0` to `1.21.0` using `kubectl`
- Updated Nginx to `1.22.0` through the YAML manifest
- Viewed the rollout history
- Rolled back to `1.21.0` and then to `1.20.0`
- Set `revisionHistoryLimit: 10`

### Rollout commands

- `kubectl rollout status` — checks the progress of a rollout
- `kubectl rollout history` — displays previous rollout revisions
- `kubectl rollout undo` — rolls back to an earlier revision
- `kubectl rollout pause` — pauses an active rollout
- `kubectl rollout resume` — continues a paused rollout
- `kubectl rollout restart` — restarts the resource’s Pods

### DaemonSet

A DaemonSet ensures that selected nodes run one copy of a Pod. The node was labeled `workload=monitoring`, and `nodeSelector` restricted the `node-monitor` Pod to that node.

### Job

A Job runs a task until it completes successfully. The `hello-job` BusyBox container printed a completion message and the current date before entering the `Completed` state.

### Assignment files

- `assignments/09-23-controllers/replicaset.yaml`
- `assignments/09-23-controllers/deployment.yaml`
- `assignments/09-23-controllers/daemonset.yaml`
- `assignments/09-23-controllers/job.yaml`

---

## Kubernetes Storage and Configuration Assignment — 10/04

### emptyDir volume

An `emptyDir` volume is created when a Pod starts and deleted when the Pod is removed. All containers in that Pod can mount and share it.

The `shared-storage-pod` used:

- A writer container that stored timestamps
- A reader container that read the same file
- One shared `emptyDir` volume

### PersistentVolume and PersistentVolumeClaim

A PersistentVolume provides cluster storage. A PersistentVolumeClaim requests storage for a Pod.

The lab created:

- A 1 GiB PersistentVolume
- A 1 GiB PersistentVolumeClaim
- A Pod that mounted the claim
- A persistent test file containing `Persistent storage is working`

Both the PV and PVC reached the `Bound` state.

### Projected volume

A projected volume combines multiple configuration sources into one mounted directory. The lab projected:

- ConfigMap data
- Secret data
- Pod name, namespace, and labels from the Downward API

### ConfigMaps and Secrets

ConfigMaps store non-sensitive configuration, such as application modes and colors.

Secrets store sensitive values such as credentials. Kubernetes Secrets provide controlled delivery to Pods, but they should still be protected with access controls and encryption. Real passwords and API keys must never be committed to Git.

The lab created ConfigMaps and Secrets using both `kubectl` and YAML manifests.

### Commands and arguments

The container’s `command` replaced the image’s default entrypoint. The `args` field supplied the script executed by that command.

### Environment variables

The lab demonstrated:

- Direct environment-variable values
- ConfigMap-based variables
- Secret-based variables
- Dependent variables using `$(VARIABLE_NAME)`
- Pod information supplied by the Downward API

The Pod printed its message, configuration, Pod name, and namespace successfully.

### Assignment files

- `assignments/10-04-storage-config/emptydir-volume.yaml`
- `assignments/10-04-storage-config/persistent-volume.yaml`
- `assignments/10-04-storage-config/projected-volume.yaml`
- `assignments/10-04-storage-config/configmap-secret.yaml`
- `assignments/10-04-storage-config/commands-env.yaml`