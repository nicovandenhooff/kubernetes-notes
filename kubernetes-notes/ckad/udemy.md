# Udemy notes

## Core Concepts

### YAML Basics

- YAML is a human-readable data serialization standard.
- YAML stands for "YAML Ain't Markup Language".
- YAML files have the `.yaml` or `.yml` extension.
- YAML is case-sensitive.
- YAML uses indentation to define the structure of the document.
- YAML uses colons to separate keys and values.
- YAML uses hyphens to create lists.

### Pods

- A `Pod` is the smallest deployable unit in Kubernetes.
- A `Pod` can contain one or more containers.
- An `initContainer` is a container(s) that run to completion before the main container starts.
  - An example use case for `initContainers` is to wait for a database to be ready before starting the main container.
- A `Pod` can have multiple containers within it ("multi-container pods").
  - There are 3 use case design patterns for multi-container pods:
    1. Sidecar: A sidecar container extends and enhances the main container (e.g. log server alongside a web server).  Side cars are implemented as a special case of `initContainer`s.
    2. Adapter: An adapter container standardizes and normalizes the output of the main container (e.g. adaptor to process logs).
    3. Ambassador: An ambassador container abstracts the network for the main container (e.g. proxy container).
- Environment variables are defined in the `env` section of a container definition, as a YAML list of key-value (`name` and `value`) pairs, but can also be defined as a `ConfigMap` or `secret`.
- Commands and arguments are defined in the `command` and `args` sections of a container definition, respectively and relate to the `ENTRYPOINT` and `CMD` instructions in a Dockerfile.

#### Example `kubectl` commands

```bash
# Create a pod
kubectl run nginx --image=nginx

# Get pods
kubectl get pods -o wide

# Describe a pod
kubectl describe pod nginx

# Pod logs
kubectl logs nginx

# Exec into a pod
kubectl exec -it nginx -- /bin/bash

# Delete a pod
kubectl delete pod nginx

# Create a pod from a YAML file
kubectl apply -f pod-definition.yaml

# Generate a pod YAML file
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod-definition.yaml

# Get a pod YAML file for a running pod
kubectl get pod nginx -o yaml > pod-definition.yaml
```

### ReplicaSets

- A `ReplicaSet` ensures that a specified number of pod replicas are running at any given time.
- Replication Controllers are the older version of ReplicaSets and are not recommended for use.
- The difference between a `ReplicaSet` and a `ReplicationController` is that a `ReplicaSet` supports the new set-based `selector` requirements.

#### Example `kubectl` commands

```bash
# create a ReplicaSet
kubectl create -f replicaset-definition.yaml

# get ReplicaSets
kubectl get replicaset

# describe a ReplicaSet
kubectl describe replicaset myapp-replicaset

# scale a ReplicaSet
kubectl scale --replicas=6 -f replicaset-definition.yaml

# replace a ReplicaSet
kubectl replace -f replicaset-definition.yaml

# delete a ReplicaSet
kubectl delete replicaset myapp-replicaset
```

### Deployments

- A `Deployment` is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features.
- A `Deployment` manages ReplicaSets and provides declarative updates to Pods.
- Deployments allow us to perform rolling updates and rollbacks (discussed later in these notes).

#### Example `kubectl` commands

```bash
# create a Deployment
kubectl create -f deployment-definition.yaml

# get Deployments
kubectl get deployments

# describe a Deployment
kubectl describe deployment myapp-deployment

# scale a Deployment
kubectl scale --replicas=6 -f deployment-definition.yaml

# replace a Deployment
kubectl replace -f deployment-definition.yaml

# delete a Deployment
kubectl delete deployment myapp-deployment
```

### Namespaces

- Namespaces provide a mechanism for isolating groups of resources within a single cluster.
- There are four initial namespaces: `default`, `kube-system`, `kube-public`, and `kube-node-lease`.
- The `default` namespace is the default namespace for objects with no other namespace.

#### Example `kubectl` commands

```bash
# get namespaces
kubectl get namespaces

# create a namespace
kubectl create namespace my-namespace

# get pods in a specific namespace
kubectl get pods --namespace=my-namespace

# get pods in all namespaces
kubectl get pods --all-namespaces
```

## Configuration

### ConfigMaps

- A `ConfigMap` is a Kubernetes resource that allows you to decouple non-confidential configuration data from image content to keep containerized applications portable.
- There are four ways you can use a `ConfigMap` to configure a container in a Pod:
  1. As a command-line argument.
  2. As environment variables.
  3. As a configuration file in a read-only volume.
  4. Write code to run inside the Pod that uses the Kubernetes API to read a `ConfigMap`.

#### Example `ConfigMap`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```

### Secrets

- `Secret`s are similar to ConfigMaps but are used to store sensitive information.
- It is important to note that Secrets are not encrypted by default, and only use base64 encoding to encode the data.
- Example use cases for Secrets include:
  - Set environment variables in a container.
  - Pull an image from a private registry.
  - Provide credentials (e.g. ssh keys) to a Pod.

#### Example `Secret`

- The data in a `Secret` must be base64 encoded.
- You can do this by calling `echo -n <value> | base64` in the terminal.
- For example:

```bash
$ echo -n 'admin' | base64
YWRtaW4=

$ echo -n '1f2d1e2e67df' | base64
MWYyZDFlMmU2N2Rm
```

- The `Secret` definition would look like this:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

- To decode the base64 encoded data, you can use the `base64` command:

```bash
$ echo 'YWRtaW4=' | base64 --decode
admin
```
