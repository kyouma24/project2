kubernetes components:

**kube-apiserver:** The core component server that exposes the Kubernetes HTTP API. All components talks to kube-apiserver and it's what is orchestrating all the actions in the cluster

**etcd:** Consistent and highly-available key value store for all API server data.

**kube-scheduler:** Looks for Pods not yet bound to a node, and assigns each Pod to a suitable node.

**kube-controller-manager:** Runs controllers to implement Kubernetes API behavior

#### Node Components

Run on every node, maintaining running pods and providing the Kubernetes runtime environment:

**kubelet:** Ensures that Pods are running, including their containers.

**kube-proxy (optional):** Maintains network rules on nodes to implement Services.

**Container runtime:** Software responsible for running containers. Read Container Runtimes to learn more.

---

containerd is the containerruntime of EKS

---

**Docker is a tool that provides a standardized format for packaging applications and their dependencies into containers.** It also provides a way to manage and orchestrate containerized applications, allowing users to easily deploy and manage their applications using Docker tools such as Docker Swarm.

**Containerd is a container runtime that provides a lightweight and streamlined way to run containers.** It puts a lot of emphasis on the robustness, simplicity, and portability of containers.  
It takes care of things such as:

* Downloading container images.
* Uploading container images.
* Setting up networking between these containers so that they can communicate with each other or the outside world.
* Managing data and files stored inside these containers.
* Starting, stopping, and restarting containers.

---

The **Docker CLI** utility accepts the command. Then it figures out what we want to do. After it understands our intention, it passes this intention to the **Docker Daemon**. This daemon is a separate program (from **Docker CLI**) that **always runs in the background** and waits for **instructions**. After the **Docker Daemon receives** our desired **action**, it tells another app, called a **container runtime**, to pull in the container image. This container runtime is called **Containerd**

**Docker and Containerd in Kubernetes**

  
---

Think of the **Docker** as a big car with all of its parts: the **engine**, the **steering** wheel, the **pedals**, and so on. And if we need the **engine**, we can easily extract it and move it into another system.

This is exactly what happened when **Kubernetes** needed such an engine. They basically said, "Hey, we don't need the **entire car** that is **Docker**; let's just pull out its **container runtime/engine**, **Containerd**, and install that into **Kubernetes**.

---

Taint is the mosquito repellent spray sprayed on a node to repel the pod to be deployed.

By default, no pods are tolerant to taint, so they can't tolerate it and don't get deployed on the tainted node.

To be able to deploy a pod, we need a tolerant to pod so that it can tolerate the taint and get deployed

---

kubectl taint nodes node-name key=value:taint-effect

Example: **kubectl taint nodes node01 tier=frontend:NoExecute**

**taint-effect**: NoSchedule | PreferNoSchedule | NoExecute  
**No Schedule**: Pods don't get deployed on that node

**PreferNoSchedule**: System will try to not deploy on the tainted node but not guaranteed

**NoExecute**: New pods don't get deployed and the existing pods will get evicted and placed on a different node if they can't tolerate the taint

---

Tolerations are applied at pod level so that they can tolerate the taint on the node

To add tolerations, edit the definition.yml file under the spec, add tolerations

containers:  
tolerations:   
 \- key: "tier"  
 operator: "Equal"  
 value: "frontend"  
 effect: "NoExecute"  
  
remember, the tolerations needs to be in double quotes

---

Also, a strong point, if a pod has toleration for a tainted node, it don't mean that it will get placed only on that node. It means that an untolerated pod cannot be deployed on the tainted node.

  
to remove a taint from a node: 

kubectl taint nodes node-name key=value:taint-effect-

example: kubectl taint nodes node01 tier=frontend:NoExecute-

or kubectl taint nodes node01 tier=frontend-

---

**affinity rules** \- provide a granular expressions on where to place a pod by using the labels attached to the node by using different type of operators(In,NotIn,Exist)  
  
example :   
containers:   
**affinity**:   
 nodeAffinity:   
**requiredDuringSchedulingIgnoredDuringExecution**:  
 nodeSelectorTerms:  
 \- matchExpressions:   
 \- key: size  
 operator: In  
 values:  
 \- Large  
 \- Medium (optional if want to provide 2 labels)

---

**requiredDuringSchedulingIgnoredDuringExecution**: while scheduling new pods, the affinity rule will be applied but it won't affect the existing pods in the node

**preferredDuringSchedulingIgnoredDuringExecution**: while scheduling new pods, the affinity rule **try it's best** to **deploy** it on the **specified labeled node** or it will place in any node if not found. but it **won't affect** the **existing pods** in the **node**

  
**requiredDuringSchedulingRequiredDuringExecution**: existing pods are **evicted** and placed on the specified labeled node & while scheduling new pods, the affinity rule will be applied

---

Resource Requests: 

a pod can request resource it needs for cpu and memory. you can configure it by

containers:  
 resources:  
 requests:  
 memory: "4Gi"  
 cpu: 2  
  
so, 4GB and 2 counts of CPU will be allocated to that particular podResource Requests

---

Resource Limits:

a pod can consume all the memory and cpu available in a node which will cause other pods to not be deployed and will get killed due to OOM.  
when cpu hits, the node tries to throttle the cpu but if memory is consumed, it gets OOM killed  
  
limit is set to limit the resource being used by a particular pod.  
  
containers:

 resources:

 requests:

 memory: "4Gi"

 cpu: 2  
 limits:  
 memory: "5Gi"  
 cpu: 4

---

a kubelet can operate and manage a node independently.

Static Pods:  
the only thing that kubelet knows to do is create pods. we can configure the kubelet to read the pod manifest files from a specified directory.

Kubelet will check at regular intervals to check for the files and deploy these pods. When updated the file, it gets recreated by kubelet and when delete the manifest file from the specified directory, then the pod gets deleted.

---

how does kubelet know where to look for this file or how to configure a path? 

  
\--pod-manifest-path

or 

config=kubeconfig.yaml > staticPodPath: /path/to/manifest

---

static pod vs daemonsets

**static pods** are created by the **kubelet** and deploys **control plane** components as static pods

**Daemonsets** are created by the **kube-api-server**(**DaemonSet controller**) and deploys monitoring agents, logging agents on **nodes**.  
  
Both are **ignored** by **Kube-scheduler**

---

to check if a pod is a **static pod**  
k get pod podname -o yaml  
  
check under **ownerReferences.kind: Node**  
  
other way is you can see the **nodename appended** to the **podname**

---

PriorityClasses:   
you can prioritize pods to run above the other hierarchally using the range of numbers, from 1B to -2B  
higher number means higher priority

the core components of the kubernetes have priority class from 1B to 2B so they are always running no matter what

---

to get priority class:  
k get priorityclass

  
```html
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
preemptionPolicy: Never / PreemptLowerPriority
description: "This priority class should be used for XYZ service pods only."
```

  
to use priority class:

```html
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  priorityClassName: high-priority
```

  
when not defined, the priority class is set to 0  
we can set the priority class globally by **globalDefault: true** in PriorityClass manifest file. there can only be **one globalDefault** rule

---

effects of pod priority: controlled by "**preemptionPolicy: PreemptLowerPriority**"  
  
will kill the lower priority pods to deploy higher priority pods  
  
when set to "**preemptionPolicy: never**". The higher priority pods will wait for resource to be available or for lower priority jobs are completed  

---

Multiple Schedulers: you can have your own scheduler other than kube-scheduler to schedule your most critical and high priority applications.   
  
you can **configure** a pod to use the **newly** created scheduler to schedule pods. the default scheduler name is: **default-scheduler**

---

to run a custom scheduler, refer to documentation:

clone the repo, build a docker image, use the deployment to deploy scheduler, use configmap to pass the scheduler configuration, add the configmap as a volume and mounth the volume as volumemount as path for config-file-path: 

---

to use the custom scheduler to schedule the pods:

spec:   
 containers:  
 schedulerName: your-custom-scheduler  
if a scheduler is not configured correctly, then the pod will be in Pending state forever  

---

when pods are about to be scheduled, they are placed in a scheduling queue. They wait in this queue to be scheduled.  
  
in this scheduling stage, the pods goes through multiple stages that helps to figure out on which node place this pod.

---

after it's in the scheduling queue, it is sent to Filtering, then Scoring and finally Binding where binding a pod to a node takes place.

All of these stages have plugins installed that helps them do the actions they do.

For example: 

**Scheduling queue**: PrioritySort plugin which looks at the priorityclass

**Filtering**: NodeResourcesFit,NodeName,NodeUnschedulable(checks if a node is unschedulable)

**Scoring**: NodeResourcesFit,ImageLocality (checks if an image is locally present on a node)

**Binding**: DefaultBinder

---

Kubernetes lets us write our own scheduling plugin by using Scheduler Profiles  
You can choose which filters to use at what stage. You also have pre & post stage for all  
example: preFilter, postBind

Scheduling Profiles allow you to configure Plugins that implement different scheduling stages, including: `QueueSort`, `Filter`, `Score`, `Bind`, `Reserve`, `Permit`, and others. You can also configure the kube-scheduler to run different profiles

---

Admission Controllers is like a controller that does checks on few things before an action is taken place like crating a pod, like the AWS SCP or AWS Config + AWS Config Rules

There are some default rules already present in the controller like, AlwaysPullImages, DefaultStorageClass,EventRateLimit,NamespaceExists, etc.

---

to check the list of admission controllers that are enabled run:  
**kube-apiserver -h | grep enable-admission-plugins**

---

to monitor the kubernetes cluster, you need to use a thirdparty monitoring app, like, prometheus or metrics server.

---

to check the logs of a pod run:   
kubectl logs pod-name  
kubectl logs -f pod-name - to stream the logs

If a pod is a multi-container pod, then to view the logs of a particular container within the pod, use:

kubectl logs -f pod-name container-name

---

to update image of a deployment:   
k set image deployment/deployment-name container\_name=new\_image\_name

to check the status of the rollout:  
k rollout status deployment/deployment-name

to check the history of the rollout:  
k rollout history deployment/deploment-name

to rollback to the previous version:  
k rollout undo deployment/deployment-name

To see the details of each revision, run:

```html
kubectl rollout history deployment/nginx-deployment --revision=2
```

To rollback to a specific revision `--to-revision`:

```html
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

---

Deployment strategy: when updating the image of a pod, we can configure how to replace the pods. Either to destroy all and recreate the pods or to replace each pod with the newly updated configuration one by one

  
**deploymentStrategy: Recreate** \- will delete all the pods and recreate pods

**deploymentStrategy: RollingUpdate** \- will replace pods one by one

the default **deploymentStrategy is RollingUpdate**

---

during rollingupdate strategy: under the hood, the **deployment** creates a **new replicaset** under the same deployment name and creates pods in the **new replicaset** **one by one**.   
  
if you're quick enough, you can view the two replicasets and you can see the pods being shifted to newer replicaset, after updating the configuration  
  
**kubectl scale deployment/deployment\_name --replicas=3**

---

in a dockerfile, we have **ENTRYPOINT** & **CMD** parameter

With Entrypoint, the parameter passed in the ENTRYPOINT is the command that the container is initiated with. Example ENTRYPOINT: \["sleep"\]. The particular image will start with the command sleep

With **CMD** the arguments are passed and if want to override the value, need to pass the entire command. Example: CMD: \["sleep", "5"\]. This can be overriden by:

docker run container\_name --image=image\_name **sleep 10\.** Necessary to pass the sleep command again.

With ENTRYPOINT, we can just append 5 at the end and the result would be:  
docker run container\_name --image=image\_name **5:** the container will sleep for 5 minutes and exit

---

spec:  
 containers:  
 **args: \["10"\]** 
 **command: \["sleep"\]**

---

The `env` and `envFrom` fields have different effects.

`**env**`allows you to set environment variables for a container, specifying a value directly for each variable that you name

`**envFrom**`allows you to set environment variables for a container by referencing either a ConfigMap or a Secret. When you use `envFrom`, all the key-value pairs in the referenced ConfigMap or Secret are set as environment variables for the container. You can also specify a common prefix string

environment variables can be passed through within the manifest:

```html
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/hello-app:2.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"
```
  
  
---

there are other ways to pass the environment variables in the containers is by using the ConfigMap.  
  
ConfigMap is a configuration that takes the environment variables in the "**key: value**" pair and can be referenced in the pod specification. Example

```html
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "printenv"]
      image: busybox:latest
      envFrom:
        - configMapRef:
            name: myconfigmap
```

Secrets: 

```html
spec:
  containers:
  - name: envars-test-container
    image: nginx
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: backend-user
          key: backend-username
```

  
---

you can create configMap using imperative way:   
**k create configmap configmap\_name --from-literal=key=value**  
the **\--from-literal** stands for the key value pairs **in this line only**  
  
using a file:  
**k create configmap configmap\_name --from-file=/path/to/file**

---

you can create secrets using imperative way: 

**k create secrets generic secret\_name --from-literal=key=value**

the **\--from-literal** stands for the key value pairs **in this line only**

using a file:

**k create secrets generic secret\_name --from-file=/path/to/file**

---

to make use of a secret or configmap there are two steps involved:

**creating the secret/configmap & injecting it into the pod configuration** 
secrets and configmaps can only passed to the pod configuration

---

configmap and secrets follow similar structure:  
 can be passed as the entire env variable:   
spec:

 containers;

**envFrom**:

 \- **configMapRef/secretRef**:

 name: **configmap\_name/secret\_name**

Or pass a single variable:  
**env:** 
**\- name: APP\_COLOR** 
 **valueFrom:** 
 **secretKeyRef/configMapKeyRef:** 
 **name: app-secret/app-configmap** 
 **key: APP\_COLOR** 

---

Note on Secrets:  
Secrets are not encrypted. only encoded. Never check-in those into SCM tools  
Secrets are also not encrypted in etcd  
Anyone who is able to create pod/deploy in a ns will be able to access the secrets in the same ns  
 configure least-privilege access to Secrets-RBAC  
Use third party tools to store secrets

---

to encode a secret: **echo -n 'mysql' | base64**  
and to decode: **echo 'vhjfjgfk' | base64 --decode**

---

Since secrets stored are not secure, we use Secret Store CSI Driver to sort of retrieve secrets from the external APIs like the AWS secrets manager.  
  
With Secret Store CSI Driver, we don't need to create a Secrets resource, we rather mount the secrets as a volume to the pod as mountPath.  
the secrets are synced at regular intervals so it will always get the updated secreted and no worries of secrets being leaked as you won't be creating a secret or store the secrets in the etcd store

---

to create a secret:   
**apiVersion: v1** 
**kind: Secret** 
**metadata:** 
 **name: my-secret** 
**data:** 
 **key1: value1** 
 **key2: value2**

---

Scaling Cluster Infra: Scale the servers(add more servers or nodes)  
Scaling Workloads: Scaling the number of pods

---

To automate the scaling of cluster: Use **Cluster Autoscaler**  
To automate the scaling of the pods: Use **Horizontal Pod Autoscaler or HPA** 
  
To scale the cluster manually: You'd add new nodes and have them join the cluster, not the typically used way  
To scale the pods manually: You can use the kubectl scale deploy/deploy-name --replicase= command to do it

---

The Horizontal Pod Autoscaler(HPA): is like the autoscaling group in the AWS. It can: 

Observer metrics

Add / Delete pods

Balances thresholds

Tracks multiple metrics for scaling up and down  
  
To add hpa to a pod run:  
**k autoscale deployment/deployment-name --cpu-percent=50 --min=1 --max=10**

The Hpa looks at the resources requests & limits and calculates the cpu percentage (for example 50% across the deployment) and add pods upto a max of 10 and a min of 1 based on the cpu threshold  

---

To take a look at the hpa's configured, run:   
k get hpa

To delete the hpa:   
k delete hpa hpa-name  
  
this is the imperative way

  
---

```html
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 1
  maxReplicas: 3
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
 
A sample hpa manifest file, the declarative way.
Note: HPA relies on metrics servers to collect the metrics, so it should be running
```

---

In place resize of Pods: refers to adding more resource requests to the pods, opposite of hpa. Vertically scaling. Normally if you try to change the resource requests, the behaviour is to kill the existing pod and recreate the new pod with the new configuration.

---

VPA vs HPA:  
VPA increases CPU & memory of existing pods  
HPA adds/removes pods

VPA **can't handle** **traffic spikes** but HPA **can**.

VPA is best for **Stateful workloads**, **CPU/memory-heavy apps** (DBs, ML workloads)  
HPA is best for **webapps, microservices, stateless services**

VPA use cases: Database(Mysql, Postgresql), JVM-based apps, AI/ML workloads  
HPA use cases: Nginx, api services, message queues, microservices

---

VPA is not installed by default and needs to be enabled and deployed manually.  
VPA deployment deploys: VPArecommender, VPAupdater. VPAadmission-controller

**VPArecommender** checks cpu & memory and recommends to VPAUpdater

**VPAUpdater** updates the resource requests by the updatePolicy.updateMode set(Recreate, Off, initial, Auto)

**VPAAdmissionController** intervenes the pod recreation when the updatePolicy.updateMode is set to Auto and updates the resource requests in place

---

During updating nodes, if a node goes down, then the pods running on that node will be inaccessible. Kubernetes waits for **5m** for the pod/node to come back online. 

This 5m wait period is called, **\--pod-eviction-timeout**  and by default it's set to **5m**. You can change this by running:   
**kube-controller-manager --pod-eviction-timeout=10m**

---

when the node comes back online after the --pod-eviction-timeout, it comes back up with no pods.

---

So, during maintenance, we can drain the nodes: purposefully move the pods to a different node

**kubectl drain node\_name**

When you drain a node, the pods are gracefully terminated and deployed on a different node and marks the node as **Cordoned** or marked as **unschedulable** (meaning no pods can be scheduled on this node).

To schedule the pods on that node again, we need to uncordon the nodes by running:   
**kubectl uncordon node\_name**

By cordoning the node, you **marks the node** to be **unschedulable** and no new pods will be deployed on that node. 

**kubectl cordon node\_name**

---

Cluster Upgrade: It's not necessary that all the core components of the kubernetes cluster are to be upgraded or be at the same version. they could be of different version

However, the Kube-ApiServer always need to be at the highest version compared to the other components.

The Controller-manager & Kube-Scheduler must be at least **1 - 2 version** lower than the **kube-api-server**

The Kubelet & Kube-proxy must be at least **2 version** lower than the **kube-api-server**

  
---

The kubernetes cluster should always be upgraded to 1 minor versions only.

---

**Shortcuts:** 

To look for any nodes that have taints, run:   
**k describe node | grep Taints**

To check the latest available upgrade run:   
**kubeadm upgrade plan**

  
---

To backup a cluster there are mutliple ways to do it.

1\. **GitHub**: You can maintain the deployments as manifest files in the SCR but if somebody launches resources using the imperative way then those resources will be lost

2\. **Kube-Api-Server**: You can query the kube-api-server to store all information about the resources in to a yaml file. 

k get all --all-namespaces -o > all-resources.yaml

3\. **ETCD:** etcd stores all information about the resources in the **\--data-dir=/var/lib/etcd**

etcd provides a ETCDCTL utility that helps you to take snapshot of the data

**etcdctl snapshot save snapshot-name.db** \- to create a snapshot

**etcdctl snapshot status** **snapshot-name.db** \- to view the status of the snapshot

---

To restore a cluster, you first need to stop the kube-apiserver because kube-apiserver depends on the etcd cluster

**service kube-apiserver stop**

Then proceed with restoring with backing up:  
**etcdutl snapshot restore snapshot-name.db --data-dir=/path/to/backup**

When etcd initializes restore, a **new cluster** and a new data dir is created and then we configure the etcd.service to use the restored snapshot

Then, **restart** the **etcd service**

Once restarted, we start the **kube-apiserver** 

---

while running any etcdctl command, we need to specify the 

**\--endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/ca.crt --cert=/etc/etcd/etcd-server.crt --key=/etc/etcd/etcd-server.key**

* `**--endpoints**` points to the etcd server (default: localhost:2379)
* `**--cacert**` path to the CA cert
* `**--cert**` path to the client cert
* `**--key**` path to the client key

  
#### Using `**etcdutl**`

To restore a snapshot to a new data directory:

```html
etcdutl snapshot restore /backup/etcd-snapshot.db \  --data-dir /var/lib/etcd-restored
```

To use a backup made with `**etcdutl backup**`, simply copy the backup contents back into `**/var/lib/etcd**` and restart etcd

**Note:** `**etcdutl backup**` performs a raw file-level copy of etcd’s data and WAL files without needing etcd to be running.

---

To create a private key:   
**openssl genrsa -out my-bank.key 1024**

To generate a public key for the private key:   
**openssl rsa -in my-bank.key -pubout > mybank.pem**

---

CA: Certificate authority are the ones who verify your business and provide you a certificate.

To create a certificate signing request (CSR): we must create a csr request:  
**openssl req -new -key my-bank.key -out my-bank.csr -subj "/C=US/ST=CA/O-MyOrg,Inc./CN=my-bank,com"**

---

Every browser is inbuilt with a certificate validator. But how does the browser know that certificate validator is the CA?

The CAs themselves have a set of public & private keys. The CAs use their private keys to sign the certificate

The public keys of all the CAs are built in to the browser. The browser uses the public key of the CA to validate that the certificate was actually signed by the CA themselves

We can see them in the setting of the browser under trusted certificatesA tab

---

To decrypt and look at the certificates, look for the .service file or look for /etc/kubernetes/manifests directory for the manifest files.

To decrypt: **openssl x509 -in /etc/kubernetes/pki/apiserver.crt -test -noout**

This will give you the certificate details

Inspect the certificate properly, like the Alt dns names, Expiry date, IP addresses & Issuer of the CA

---

In case one of the component is down due to certificate issue, you can check the logs and find out or do a journalctl command

In case the main components like the kube-api-server or the etcd server is down, then you need to go one step down and look for all containers:   
**docker ps -a** & look for the logs of the container by **docker logs container\_name**

Or **crictl ps -a & crictl logs container\_name**

---

When a new admin comes into my team & if she needs access to the cluster. We need to get her a pair of certificate and key pair for her to access the cluster. 

She creates her own private key, generates a certificate signing request, and sends it to me. 

Since I'm the only admin, I then take the certificate signing request to my CA server, gets it signed by the CA server using the CA server's private key and root certificate, thereby generating a certificate and then sends the certificate back to her. 

She now has her own valid pair of certificate and key that she can use to access the cluster. The certificates have a validity period. It ends after a period of time. Every time it expires, she needs to follow the same procedure.

---

To manage the certificate signing requests, as well as to rotate certificates when they expire, Kubernetes has a built-in certificates API that can do this for you. 

With the **certificates API**, you now send a certificate signing request directly to Kubernetes through an **API call**. This time, when the administrator receives a certificate signing request, instead of logging onto the master node and signing the certificate by himself, he creates a **Kubernetes API object** called **CertificateSigningRequest**. 

Once the object is created, **all certificate signing requests can be seen by administrators** of the cluster. The request can be reviewed and approved easily using kubectl commands. This certificate can then be extracted and shared with the user.

**k get csr** : to get the CSRs

**k certificate approve cert-name** : to approve the certificate

---

All certificate related operations are carried out by **Controller Manager** such as **CSR-Approving & CSR-Signing**  
they need the CA server's root certificate and private key. under the flag:  
**\--cluster-signing-cert-file** & **\--cluster-signing-key-file**

---

**To create a signing request better use this below command:**   
cat <<EOF | kubectl create -f - apiVersion: certificates.k8s.io/v1beta1 kind: CertificateSigningRequest metadata: name: csr-controller spec: groups: - system:authenticated request: $(cat server.csr | base64 | tr -d '\\n') usages: - server auth EOF

---

actually when we run any **kubectl** commands, in the background, the --server, --client-key, --client-certificate, --certificate-authority are always applied.  
  
This configuration is saved in the kubeconfig file, then you can pass it as, **kubectl get pods --kubeconfig config-file-name**

By default, the kubectl looks for the config file in the **$HOME/.kube/**config-file directory. So if the file is created in this path, then you won't have to specify any file

---

The kubeconfig file consists of: 

apiVersion: **v1**  
kind: **Config**  
clusters:  
 \- name: cluster\_name  
 cluster:   
 certificate-authority  
contexts:  
 \- name:  
 context:  
users:  
 \- name:  
 user:

The **clusters, contexts & users** fields are **arrays**, so you can specify **multiple** users, clusters and contexts.

---

To view the current file being used:   
**k config view**

You can use **kubectl config \[command\]** for other configuration commands

  
To use a custom-config but not make any changes to the default config, you need set the variables in the **.bashrc** file, located in the root directory  
add the line: **export KUBECONFIG=/path/to/file**

Once added, save the file and to apply the changes, run: **source \~/.bashrc**

---

The **core** api is set to **v1**

There are other apis, like the **apps extensions, networking, storage, authentication, authorization**, et cetera. 

Within apps, we have **v1/**: **deployments, replica sets, stateful sets**. Within networking, you have network policies. Certificates have these certificate signing requests 

So the ones at the **top** are **API groups**, and the ones at the **bottom** are **resources** in those groups. Each resource in this has a set of actions associated with them.

Things that you can do with these resources, such as:  
**list** the deployments, get information about one of these deployments 

**create** a deployment

**delete** a deployment

**update** a deployment

**watch** a deployment

These are known as verbs: **list, create, delete, update, watch**

---

These APIs are categorized into two: the **core group** and the **named** group.   
The core group is where all core functionality exists, such as:   
**namespaces** 
**pods** 
**replication controllers** 
**events** 
**endpoints** 
**nodes** 
**bindings** 
**persistent volumes** 
**persistent volume claims** 
**conflict maps** 
**secrets** 
**services**

---

**kubectl proxy:** This command launches a proxy service locally on port 8001 and uses the credentials and that way you won't have to pass the certificate details in each command

---

There are different authorization mechanisms supported by Kubernetes: 

* Node authorization
* Attribute-based authorization(ABAC)
* Role-based authorization(RBAC)
* Webhook

**Webhook:** To manage authorization externally and not through the built-in mechanisms for instance, **Open Policy Agent** is a third-party tool that helps with admission control and authorization. Kubernetes make an API call to the Open Policy Agent with the information about the user and his access requirements, the agent decide if the user should be permitted or not. 

---

**Node Authorizer:** Any information about the node is handled by this authorizer  
example: **kubelet** needs access to the nodes, such as read & write services, pods, events, and all of this actions are authorized by **NodeAuthorizer.** They also belong to system:nodes group

**ABAC**: Where a user or a group of users is associated with a set of permissions. The disadvantage is that when you need to make changes in the policy, then you'll need to restart the **Kube API server** everytime

**RBAC**: With role-based access controls, instead of directly associating a user or a group with a set of permissions, we define a **role**, and we **attach users** to that **role**.

  
---

There are two modes: **AlwaysAllow** & **AlwaysDeny.** AlwaysAllow is enabled by default

The modes are set by the flag **\--authorization-mode=Node,RBAC,Webhook,** to the kube-api-server

When a module or mode denies a request, it goes to the next in queue and repeats until all modes are checked

---

**RBAC:** Since ABAC has a flaw that needs manually editing each user's definition file and also restarting the kube-apiserver all the time when a change is made, it is cumbersome to manage

With **RBAC** you create a **Role** and then **attach** a role to a **User** using the **RoleBinding** resource.

The **Role** definition consists of, apiVersion, kind:Role, metadata & rules. The rules in a list so, we can provide multiple rules for a single role.  
**rules:** 
**\- apiGroups: \[""\]** \# For the core api services, you can keep this as blank for other needs mentioning    
 **resources: \["pods"\]** 
 **verbs: \["create", "list"\]**

  
**Note:** The Role and the **RoleBinding** are namespace bound.

---

The **RoleBinding** resource is then used to bind the role to a User

apiVersion: rbac.authorization.k8s.io/v1

kind: RoleBinding

metadata:

 name: read-pods

 namespace: default

subjects:

\# You can specify more than one "subject"

\- kind: User

 name: jane # "name" is case sensitive

 apiGroup: rbac.authorization.k8s.io

roleRef:

 \# "roleRef" specifies the binding to a Role / ClusterRole

 kind: Role #this must be Role or ClusterRole

 name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to

 apiGroup: rbac.authorization.k8s.io

---

to view created roles: **k get roles/rolebindings, k describe role/rolebindings**

To check if you are authorized to perform an action. Kubernetes has a built-in way to check this:

**kubectl auth can-i** verb resource\_name **\--as user\_name**

example: **k auth can-i create delete deployments --as dev-user** (to check if dev-user can by using --as)

To check for the current user, run it without **\--as**

To check for actions in a particular namespace add:   
k auth can-i create delete deployments --as dev-user **\--namespace test**

---

You can further restrict the actions per resources and per namespaces.

metadata:   
 name:   
 **namespace: monitoring** \# restrict role to only monitoring namespace  
rules:  
\- apiGroups: \[""\]   
 resources: \["pods"\]  
 verbs: \["create", "list"\]  
**resourceNames: \["blue", "green"\]** \# the role can only access pods that are named blue / green.

---

You can also use the **\--as** flag with create or run commands like:   
k **\--as dev-user** create deployment nginx --image=nginx -n blue

k **\--as dev-user** run nginx --image=nginx -n blue

---

**ClusterRole** & **ClusterRoleBinding :** They are just like the Roles & RoleBindings except they are used for cluster-scoped resources, like: PV, namespaces, nodes, CSRs

To check for the api-resources that fall under ClusterRoles run:   
**k api-resources --namespaced=false**

**Note:** It doesn't mean that ClusterRole is only used for cluster-scoped resources, they can also be created for namespaced resources. Except, when created ClusterRole for namespaced resources, they are given access **across the namespaces**

---

To create a clusterrole or role using the imperative way: 

**k create clusterrole/role** role\_name **\--resource=pods,deployments --verb=get,list,watch,delete,create**

To create clusterrolebinding or rolebinding for the above created role:

**k create clusterrolebinding/rolebinding** rolebindingname **\--user=user\_name --role/clusterrole=role\_name**

---

There are two types of accounts in Kubernetes: ServiceAccount & UserAccount

**ServiceAccount**: Used by applications or a machine to perform actions on the cluster  
example: Prometheus pulls metrics using SA & Jenkins use SA to deploy applications

To create a serviceacount: **k create sa serviceaccount-name**

A **ServiceAccount** token is created **automatically** when a service account is created. However, the token is stored as a secret. 

So, whenever a **ServiceAccount** is created, a serviceaccount is created > a token is created > then a secret is created to store the token. The secret object is then linked to the **ServiceAccount**

The **token** can be used in return to make an **API** call to the kube-api-server

---

For **every namespace** in the cluster, a default **serviceaccount** is created. And whenever a pod is created, the **default service account** & it's **token** are automatically **mounted** as a **volume** to the pod.

When you describe the pod, you can see the serviceaccount token being mounted at:   
**/var/run/secrets/kubernetes.io/serviceaccount** 

The default service account has the least permissions just to do the basic tasks

**Note:** If you don't wish for the pod to have the default serviceaccount mounted, **add automountServiceAccountToken: false** under **spec**

---

If need to attach a custom service account to the pod add it to the definition file under spec:

spec:   
 containers:  
 **serviceAccountName:**

**Note:** You **cannot** change the **serviceaccount** of the **pod**, you must delete and recreate. However, in deployment, a **rollout** will be triggered and **new pods will be created** with the **new serviceaccount**

---

With the newer version v1.24: 

A ServiceAccount no longer creates a serviceAccountToken automatically. You must run **k create token sa-name** to create a token

The previous version of serviceAccountToken didn't have an expiry date and that changed too. Now the serviceAccountTokens are created using the TokenRequestAPI to obtain a token and have an expiry date & time, usually 1h.

---

To use a private docker image, you need to first login to the account using the **docker login your\_private\_registry.io**

Once logged in, you'll be able to pull and run images from the registry. Make sure to pass the registry name in the image section:   
**docker run your\_private\_registry.io/nginx**

---

But how will you pass the credentials for Kubernetes cluster?

We need to create a secret type called **docker-registry** & we'll call it **regcred.**

**docker-registry** is a built-in secret type that was build for storing docker credentials.

**kubectl create secret docker-registry regcred \\** 
**\--docker-server=your\_private\_registry.io \\** 
**\--docker-username=registry\_user \\** 
**\--docker-password=registry\_password \\** 
**\--docker-email=registry-user@org.com**

  
---

To pull now from this private registry, we need to use **imagePullSecrets** parameter in the definition file under spec.

spec:   
 containers:   
 imagePullSecrets:   
 \- name: private-secret-docker-registry

---

securityContext: A security context defines privilege and access control settings for a Pod or Container

When added securityContext to a pod, it is applied across the containers in the pod. If it's applied at the container & pod level, the container level will override the pod level security context.

For pod, it comes under **spec**.

For container, it comes under **spec.containers.securityContext** 

---

By default, all communication across all the pods are allowed, meaning all pods can talk to all of the pods present in the cluster.

To implement restriction or to restrict traffic flow from one of the pod and only allow traffic from a specific set of pods, we need to use **NetworkPolicy**

**NetworkPolicy** is a resource that controls the ingress & egress traffic to & from the pod based on the rules given.

NetworkPolicies allow you to specify rules for traffic flow within your cluster, and also between Pods and the outside world

Network policies are implemented by the network plugin. To use network policies, you must be using a networking solution which supports NetworkPolicy. Creating a NetworkPolicy resource without a controller that implements it will have no effect.

Some of the known NetworkPolicy applications are, **Calico, Romana, Weavenets, Kube-router** etc.

**Flannel do not support NetworkPolicies**

---

Suppose you have a three-tier architecture. A webapp, an API pod and a DB pod.

If you want all traffic to DB pod only to be passed through the API pod, then we need to look at the traffic from the DB pod's perspective.

From DB pod's POV, we need to **allow ingress** **only** from the API pod, so ingress.

Do we also need to configure egress rule so that the data goes back to API Pod? **No**, when an **ingress** is configured the egress is configured automatically.

---

```html
spec:
  podSelector:
    matchLabels:
      app: db-pod
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api-pod
      namespaceSelector: # This is not a list, hence pod & namespace needs to match
        matchLabels:
          name: prod
    ports:
    - protocol: TCP
      port: 3306
```

An example of a Network policy that selects the pod db and applies ingress policy to be allowed only on the port 3306 coming only from the pod "api-pod"

  
If namespaceSelector is passed as a list, then it'd mean, that any pod with the label, **app=api-pod** & any pod from the namespace, that has the label, **env=prod**, both will have access to it.

---

Suppose the DB needs to be accessed by a server that lives outside the kubernetes cluster environment, in that case, you'd allow the traffic from the **server's IP address** under the "- ipBlock"

ingress:  
\- from:   
 **\- ipBlock:** 
 **cidr: private\_ip/32**

---

ingress & egress rules work similarly, except, you'd write a **"- to"** block compared to **"- from"** block in the egress rules & the ports section remains the same in both

---

**Kubectx:** With this tool, you don't have to make use of lengthy “kubectl config” commands to switch between contexts. This tool is particularly useful to switch context between clusters in a multi-cluster environment.

**Installation:**

sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx  
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx

**Syntax:**

1. To list all contexts: **kubectx**
2. To switch to a new context: **kubectx <context\_name>**
3. To switch back to previous context: **kubectx -**
4. To see current context: **kubectx -c**

---

**Kubens:** This tool allows users to switch between namespaces quickly with a simple command.

**Installation:**

sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx  
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens

**Syntax:**

1. To switch to a new namespace: **kubens <new\_namespace>**
2. To switch back to previous namespace: **kubens -**

---

CustomResourceDefinitions: All objects in the clusters are a resource, like the Pods, Deployments, ReplicaSets. All of these are resources are controlled & managed by Controllers that were discussed earlier.  
For pods: Pod Controller  
Deployment: Deployment Controller  
ReplicaSet: ReplicaSet Controller

The **controllers** are **responsible** for maintaining and running the resources as desired in the definition file.

The controllers are written in **Go Language** and are available to use in built in Kubernetes.

---

Suppose you want to create a custom resource, like a **resource / kind** called: **TrainTicket** and if you try to create this resource, you'll be thrown an error saying that, **TrainTicket** resource doesn't exist in the **apps/v1 or v1** API group. Expected, since there's no such resource or a controller that can handle or maintain these type of requests/resources.

  
To be able to use **TrainTicket** resource, we first need to create a controller that will handle or maintain this resource. In other words, a **custom controller**

**TrainTicket** resource usage: When created TrainTicket resource, it calls an api in the background to book or cancel a ticket by creating or deleting the resource respectively.

---

We'll need to create a **CustomResourceDefinition/api-resource** for the TrainTicket resource. 

**CustomResourceDefinition** consists of two things: Custom-resource & Custom Controller

  
**CustomResource:** A _custom resource_ is an extension of the Kubernetes API that is not necessarily available in a default Kubernetes installation. It represents a customization of a particular Kubernetes installation.

**CustomController:** Custom resources let you store and retrieve structured data. When you combine a custom resource with a _custom controller_, custom resources provide a true _declarative API._ The Kubernetes controller keeps the current state of Kubernetes objects in sync with your declared desired state

Custom controllers can work with any kind of resource, but they are especially effective when combined with custom resources

---

A custom controller is basically an application that runs and watches it's kind for any changes and also does automation based on the requests it makes. 

In our example of TrainTicket, a custom controller called TrainTicket will look at the resources for TrainTicket and book, cancel and modify the tickets based on the desired requests. 

https://github.com/kubernetes/sample-controller  
We can clone this git and edit the controller.go file to write our own custom controller and run it.

Since Kubernetes is written on GoLang, writing custom controllers in GoLang would be much more supportive compared to other languages.

---

We don't want to package the controller and run it every time, since it's better to convert it into a docker image and run it as a pod in the cluster

---

Docker Volumes: There are two types of volumes, one is mount & other is bind.

**Mount Volume**: When you are mounting volume, a volume is created in the **/var/lib/docker/volume** directory and that volume is mounted inside the container.

**Bind Volume:** When you bind volume, a volume is binded of your specified directory inside the container

Example commands: 

**docker run -v volume1:/var/lib/mysql mysql** \- will create a directory in the volumes

**docker run -v /data/mysql:/var/lib/mysql mysql** \- will bind this specific folder inside container

**docker run --mount type=bind, source=/data/mysql, target=/var/lib/mysql msql** \- this is the new way of mounting or binding volumes

  
---

**StorageDrivers**: Docker will use different storage drivers based on the OS it's running on. Some of the common are: 

* AUFS - Ubuntu
* ZFS
* BTRFS
* Device Mapper
* Overlay
* Overlay2

---

Storage Drivers helps you to create storages for containers

Volume Drivers helps to create volumes on the machine where the containers are ran. Some of the common volume drivers are: 

* Local
* Azure File Storage
* Convoy
* DigitalOcean Block Storage
* Flocker
* gce-docker
* NetApp
* VMware vSphere Storage
* RexRay - EBS volumes, S3

---

CRI - Container Runtime Interface  
CNI - Container Network Interface  
CSI - Container Storage Interface

The C**X**I makes the api calls to create a volume, or a network or to run a container and the relevant interfaces takes the api calls from the Interfaces and **creates/deletes/updates** \- **volume/network/containers**

---

Volumes & Mounts: In kubernetes if you want to mount a volume from the local to the container you need to two different parameters.

The first one we need is the **volumes:** parameter that creates the volume.  
**volume:** 
**\- name: my-pod-volume** 
 **hostPath:** 
 **path: /data** 
 **type: Directory** #the directory must exist when passed this or pass DirectoryOrCreate

Then, we need to refer this volume and mount it to the container by mentioning the path that you want to mount from within the container.

Suppose you want to mount the **/opt** directory within the container with the **/data** volume that we created above, then:   
**spec:** 
 **containers:** 
 **volumeMounts:** 
 **\- mountPath: /opt** 
 **name: my-pod-volume**  
  
---

There are different ways to create volumes, the one that we discussed earlier **hostPath** will only work if you have a single node.

If you have multiple nodes and pods spread across all the nodes, then you need to use a CSI driver to use like AWS EBS

---

Persistent Volumes are a piece of storage **provisioned by an admin or created dynamically** by the cluster.

Just like how **CPU&memory** is made available by **nodes** to run pods, **Persistent Volumes** are of the same instead they provide **storage** to the pod **beyond the lifecycle** of it. You create storages and the pods make use of these Persistent Volumes.

They are just like any other resource within the cluster and can be provisioned & manipulated by API calls. 

Persistent Volumes support different types of storage backends, such as **NFS, iSCSI, AWS EBS, Azure Disk, or local disks.**

They are used in conjunction with **Persistent Volume Claims (PVCs)**, which are requests for storage by users or applications

---

Previously, we mounted the volumes directly into the pod using the hostPath on the local machine but like mentioned earlier, this is not scalable in production env as all of the pods won't be running on the same node.

And if were to make changes to the path, then changes need to be carried out across all the pods. Obviously this is not ideal in production environment

---

A sample definition file for PV: 

```html
apiVersion: v1
kind: PersistentVolume
metadata:
  name: foo-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath: 
    path: /tmp/data 
```

---

There are different type of accessModes for the Persistent Volumes created

1. **ReadWriteOnce:** the volume can be mounted as read-write by a single node. Many pods can access the volume but they should all be on the same node
2. **ReadWriteMany:** the volume can be mounted as read-write by many nodes
3. **ReadOnlyMany:** the volume can be mounted as read-only by many nodes
4. **ReadWriteOncePod:** the volume can be mounted as read-write only by a single pod in the cluster

The `ReadWriteOncePod` access mode is only supported for CSI volumes and Kubernetes version 1.22+. We need the CSIsidecars to the version or greater: 

* **csi-provisioner:v3.0.0+**
* **csi-attacher:v3.3.0+**
* **csi-resizer:v1.3.0+**

---

In the CLI, the access modes are abbreviated to:

* **RWO** \- ReadWriteOnce
* **ROX** \- ReadOnlyMany
* **RWX** \- ReadWriteMany
* **RWOP** \- ReadWriteOncePod

---

**PersistentVolumeClaims**: A **_PersistentVolumeClaim_ (PVC)** is a request for storage by a **user**. It is similar to a Pod. Pods consume node resources and **PVCs** consume **PV resources**. Pods can request specific levels of resources (CPU and Memory). Claims can request **specific size** and **access modes**

**Every PVC** is **bound** to a **single PV**. Kubernetes tries to find a PV that as sufficient **capacity, accessmodes, volumeModes, storageClass.**

You can also let the **PVC** select a **specific PV** by using the **labels** & **selectors. Kubernetes** will try to find the **optimal PV available** and if unavailable then it might go for a **larger size than requested** if **no PVs** are **available**

Once a PVC claims a PV, then another PVC **cannot claim** it until another PV is available. Until then, PVC will remain in the **Pending** state.

---

The **PVs** can be provisioned in two ways. **Static** & **Dynamic**

**Static:** The admin creates them through API calls and keeps the details of the storage and PVs available

**Dynamic**: When none of the PVs are available to **claim** then the **cluster** might go out and **create** a **PV** as requested by the **PVCs**. 

However, this provisioning is based on **StorageClasses**: the PVC must request a **storage class** and the administrator **must** have created and configured that class for dynamic provisioning to occur.

To enable dynamic storage provisioning based on storage class, the cluster administrator needs to enable the `DefaultStorageClass` admission controller on the API server

---

An example definition for PVC:

```html
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi  # requested size
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"       # to select a specific PV
    matchExpressions:
      - {key: environment, operator: In, values: [dev]} # match by expressions
```

---

When a PVC is deleted, you can decide what should happen with the PV. This action is controlled by **persistentVolumeClaimReclaimPolicy**: Retain (default)

There are 3 different Reclaim policy: 

* **Retain**: The volume is retained and not deleted and not available for other PVCs to claim
* **Delete:** The volume will be deleted upon deletion of PVC
* **Recycle** (deprecated): The data in the data volume will be scrubbed and will be made available for other PVCs to claim (`rm -rf /thevolume/*`)

---

**StorageClass:** StorageClass is used to provision volumes dynamically matching the claim requests. When using StorageClass, you don't need to configure PVs since PVCs makes use of the StorageClass to provision the PV and then we can bind it to a pod

There are different types of provisioners available for StorageClass, e.g., AWSEBS, GCE, AzureDisk

Each provisioner has it's own set of parameters set. 

```html
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer # will wait for pod using the PVC to be up before binding the volume
reclaimPolicy: Retain # default value is Delete
parameters:
  csi.storage.k8s.io/fstype: xfs
  type: io1 # additional parameter specific to provisioner
  tagSpecification_1: "name=volume"
  tagSpecification_2: "key2=value2"
allowedTopologies:
- matchLabelExpressions:
  - key: topology.ebs.csi.aws.com/zone # additional parameter specific to provisioner
    values:
    - us-east-2c
```

---

**volumeBindingMode:** Controls when volume binding and dynamic provisioning should occur. When unset, `Immediate` mode is used by default. The other option is , `WaitForFirstConsumer` .

`Immediate` : The PV will be created right after the PVC is created. For **storage backends** that are **topology-constrained** and **not** globally accessible from all Nodes in the cluster, PVs will be bound or provisioned without knowledge of the Pod's scheduling requirements. This may result in **unschedulable** Pods.

`WaitForFirstConsumer:` This mode will delay binding and provisioning of a volume until the pod using the PVC is created. PVs will be created confirming the topology constraints. 

---

Networking in linux:

Let's assume that there are two different servers or machines

**Switching**

How does the first server connect to the second server?  
We connect them to a switch and the switch creates a network containing the two systems. To connect them to a switch, we need an interface, physical or virtual.   
To list the interfaces in the host: **ip link** (for e.g., eth0, eth1)

Then we assign the systems on the ip address of the same system (ex: 10.0.01 server: **ip addr add 10.0.0.1/24 dev eth0**) Repeat the same for the other server

Now the systems can communicate with each other

---

Let's assume that is another network and there are two different servers in each network

How would you establish communication between the servers in different networks? 

**Routing**: Router is an intelligent device and helps connect two networks. Think of it as a separate computer between these networks and has many network ports

Since its attached to **two different networks**, it gets two IPs from each network

When **system A** is trying to connect to **system C** in a **different network**, how does the system know which router to use, like mentioned earlier, it can have many networking routes and it doesn't know which will let it connect to system A

---

Taking the below scenario, how does the system know where the router is and how to reach that router in order to talk to the other system?

**Gateway**: If network is a room then the gateway is a door to the outside world. The systems need to know where the door is to go through it.

To check for current routing configuration, run: **route**

To let the system A to reach the system C on a different network, we need to add a ip route:   
**ip route add system-C:cidr via route\_ip\_address\_networkC**

Similarly, if system C to send back data that is connected to system A, we need to an ip route for system C network router ip address  
**ip route add system-A:cidr via router\_ip\_address\_networkA**

---

To let the server connect to the internet:   
**ip addr add default/0.0.0.0 via router\_ip\_address**

If a server doesn't have internet access, then the place to look for is the gateway

---

To host a server as a router, we need to enable ip forwarding from that server. By default, ip forwarding is disabled and we can control this by editing the **/proc/sys/net/ipv4/ip\_forward** file to **1 (0 means no forwarding)**

However, simply editing the file doesn't persist the changes on reboot. For that we need to edit the **/etc/sysctl.conf** file and add **net.ipv4.ip\_forward = 1**

---

* **ip link:** To list and modify the interfaces on the host
* **ip addr**: To see the ip address assigned to the interfaces
* **ip addr add <cidr>** **dev eth0** : To set ip address on the interfaces
* **ip route**: To view the routing table
* **ip route add network\_cidr via router\_ip**: To add entries in the routing table
* **cat /proc/sys/net/ipv4/ip\_forward** : To check if ip forwarding is enabled

Any changes made do not persist on reboot. For it, you need to edit **/etc/network/interfaces** or **/etc/sysctl.conf** file

---

Every server has it's ip address and if you want to ping to that server you need to enter its ip address. 

You can also give a name to the server, for example, **db** 

These changes are to be done on the **source server's etc hosts** file

Simply edit the file and enter the ip address followed by the name you want to refer to. example:  
<**ipaddess> db** \- now, if you run **ping db** from the source server, you should be able to reach the target server

---

However, as your organization grows you cannot edit the hosts file in each server, it will be cumbersome. Rather, you can move all of these dns entries into a single server and manage it centrally, that's is the DNS server

Then, we configure the servers' to look up the DNSs from the DNS server instead of it **etc hosts** file. How do we do that? 

Every host has a DNS resolution configuration file at /etc/resolve.conf. Add entry:   
**nameserver <ip\_address>**

Now, using the DNS server for all of the DNS configuration doesn't mean that you can't use the **/hosts** file any more. You can still use it

---

Lets say that you have entries in the **DNS server** and in the **hosts server** for a DNS but with **different ip address**, which one would be **picked**? The DNS servers or the host servers?

The host server, the server looks for the **local /hosts file** and if it couldn't find it there then it looks for it in the 

This is the **default behaviour** and it can be changed by editing the **/etc /nsswitch.conf** file: 

**hosts: file dns** (so first it looks for hosts file and then dns)

**hosts: dns file** (so first it looks for dns and then file)

---

Domain Names: www.facebook.com, www.google.com are all domain names and when we try to ping any of the domain name, the server first goes through the internal DNS. Since the internal DNS doesn't know where it is, it'll route the traffic to internet. The ip address of the www.google.com maybe resolved by multiple DNS servers

The **.com, .net, .edu, .org, .io** are the top level domains

* **.com** \- For e-commerce
* **.net** \- For network related sites
* **.edu** \- For educational purposes
* **.org** \- For non profit organization
* **.io** \- For

Taking the example of **www.google.com:** 

* **.com** \- is the top level domain name
* **google** \- is the domain name
* **www** \- is the subdomain

**Google's maps is available at maps.google.com and gmail is available at mail.goole.com**

**The gmail and the maps are the subdomains of Google**

---

The request goes in this way: 

**Internal DNS** \> **Root DNS** \> **.com DNS** \> **Google DNS** \> resolving the ip address of the domain

In order to speed up the connectivity, the **internal DNS will cache the ip address** for some time

---

Lets say your organization has it's own domain called, mycompany.com and it has many subdomains for different services, like the mail, pay, hr, etc.

Since it's internal, you should be able to ping the **mail** service by just running: **ping mail**

However, in your DNS server, the entry is marked as **mail.mycompany.com.** How do we use just the sub domain to ping? 

For that, we need to add another **entry** in the **resolv.conf** file to append after the service name  
**search mycompany.com**

---

Record Types in DNS:

* **A record**: Stores IPv4 address
* **AAAA record**: Stores IPv6 address
* **CNAME record**: Maps one name to another. example: facebook.com, fb.com (facebook can also be reached when searched fb.com)

There are different tools to resolve DNS: **nslookup** & **dig**

**Note: nslookup doesn't resolve hostnames in the local hosts file.** It has to have the entry in the DNS server. It only resolves entries that are in the DNS server. The same goes for **dig**

---

The k8s cluster consists of **master** & **worker nodes**. 

* Each node must have at least one interface connected to a network.
* Each interface must have an address configured
* Each hosts must have a unique hostname set and a MAC address

---

The following ports must be allowed for the core components to communicate with each other

* **Kubelet**: The kubelet on the master and the worker nodes listen on **10250**
* **Kube-api-server**: The master should accept connections on **6443** so that it accepts requests from **Kubelets** on the **worker nodes** & all other components
* **ETCD**: Listens on **2379**
* **ETCD multi-cluster**: Port **2380** needs to be open if we have multiple master nodes so that etcd servers can talk to each other
* **Kube-scheduler**: Listens on **10259**
* **Kube-controller-manager**: Listens on **10257**
* The worker nodes expose services externally so they need ports from **30000-32767** to be open

---

`**ifconfig**`: Display or configure **network interfaces**

`**ss -tuln**` : Show listening ports (faster & better than `**netstat**`)

`**ip link**` is a Linux command used to **view and manage network interfaces** (like Ethernet, loopback, virtual adapters)

`**ip link set eth0 up**` : **Enable** interface `**eth0**`

`**ip link add veth0 type veth peer name veth1**` **Creates** virtual **Ethernet** pair

`**ip route**` (or `**ip r**`) is a **Linux command used to view and manipulate the kernel's routing table** — basically, it tells Linux where to send network packets.

`**ip route add 10.10.0.0/16 via 192.168.1.254**` : **Add** static **route**

---

`**ip addr add/delete 192.168.1.100/24 dev eth0**` : Add/Delete a **new IP**

`**ip addr**` : See what **IP addresses** your system has

---

**netstat -npa | grep -i etcd | grep -i 2379 | wc -l**

* netstat -npa | grep -i **<service\_name> |** grep -i **<port\_number> |** wc -l **:** This command is to see the number of established connection for a service on a specified port
* **ip address show type** : Will show all the interfaces that are of bridge type

---

**CNI** : **ContainerNetworkInterface** takes care of assigning the ip address to a pod and makes sure that all the pods in the cluster can talk to each other regardless of the nodes.

This is done by creating a network interface spanning across the nodes.

When a pod is created, the CNI plugin: 

* Creates a Linux network namespace for the pod
* Assigns an IP address
* Configures a virtual Ethernet pair (veth)
* Adds routing rules to allow inter-pod communication

---

The CNI in the background runs a script that performs the below mentioned actions when a pod is created.

---

So basically, a CNI is like a operations person (operator) that looks at all the pods that are getting created. If it sees a pod is getting created, it'll create a network namespace for the pod and assign an IP address to it so that it can then use this ip address to attach to a virtual ethernet pair and adds routing rules to this ip address from all the other pods that are present

---

By default, all the CNI plugins that are available as stored in the **/opt/cni/bin** directory, such as: 

1. flannel
2. weaveworks
3. vmware nsx
4. cilium

And the configurations related to the network plugin are stored in the **/etc/cni/net.d** directory, such as: 

1. flannel.conflist
2. bridge.conflist

There could be multiple configuration file, each for each CNI plugin.

To check which CNI is currently being used, **ls /etc/cni/net.d** directory. and to check which binary will be executed, check the **type**

---

How does a network plugin work?

In a cluster, if you have minimal nodes and pods, the kube-proxy can handle the communication between the pods, however, if the size of the nodes and pods increases, it fails to keep up with the routing and the router might not support adding multiple number of routes and could lead to failure.

Weaveworks, deploys an agent as the daemonset and this agent creates a bridge network in each of the node. When a pod wants to communicate with another pod: 

1. Weaveworks plugin intercepts the pod and looks at the source and destination, it then encapsulates the packet which consists the details about the node it's trying to reach
2. When the pod reaches the other node, the weaveworks agent running on that node looks at the configuration and decapsulates and the traffic is routed to specified pod

---

IP address management for the pods and the virtual bridge network are assigned by CNI plugin

And no duplicate IPs are to be assigned. To manage this, all the ip address being assigned should be stored in a file with their status. 

CNI plugin comes with two built-in plugin (**DHCP & host-local**) to outsource this task. 

The plugin that manages ip address of the pods on the host are controlled by **host-local** plugin, but it's still our responsibility to **invoke** the plugin

  
The CNI configuration file has a section for **ipam** which controls which plugin to be used and also pass the **subnet range** & **route** for the pods 

---

Weaveworks creates an IP range of **/12**. example: **10.32.0.0/12**

Which gives the range from **10.32.0.1** \> **10.47.255.254**

Which is about a **million IPs**

The Weaveworks plugin then splits the **IP addresses** among the nodes to be managed. And the pods created in the node will have the IP address within that subnet range

---

**Service Networking**: When a service is created, the **service is accessible from all pods** and from **all nodes**. While a **pod** is created on a **node**, a **service** is **hosted across** the **cluster**

**ClusterIP**: The service is only accessible within the cluster

**NodePort**: The service uses a port on each node to be accessible from outside the cluster

---

**KubeDNS**: Whenever a pod and a service is created, the **KubeDNS** makes note of the **service name and it's IP address in a table,** so that whenever you try to curl to that particular service using:   
**curl web-service --> You get the ip address and routed to that particular pod**

The above will work since both pods are in the same namespace -> **default**

If the service were to be in a different namespace, then: **curl** web-service.**apps** (apps is the namespace)

---

The DNS names are further categorized into the **Types:** for service is svc, returning: **web-service.apps.svc**

And further comes **Root:** which is always set to **cluster.local**, returning: **web-service.apps.svc.cluster.local (**this is the fully qualified domain name for the service) and thats how services are resolved in Kubernetes

---

**For pods:** The hostname is not added as the pod name but as the IP address of the pod but replacing the "." with "-". eg: **10-244-1-5**

So, their DNS becomes: **10-244-1-5.apps.pod.cluster.local** \- apps being the namespace

---

How Kubernetes Implements DNS? : All pods have the **/etc/resolv.conf file.** This file contains the details related to the Kube-DNS or the CoreDNS server details, named as **nameserver**

**Nameserver:** Points to the ip address of the CoreDNS pod

**Search:** Contains the domains it should try to resolve the dns names (contains default.svc.cluster.local svc.cluster.local)

**ndots:** This forces the resolver to try adding search domains first → multiple lookups before going external (by default set to 5: **ndots:5**)

---

**CoreDNS** is deployed as a Deployment, having 2 replicas. To get the IP address of the CoreDNS service, a default service is always running called **kube-dns**

CoreDNS relies on the **/etc/coredns/Corefile.** 

This file contains plugins related settings that handle the traffic

The Corefile is passed is a configmap object to the CoreDNS

---

**Ingress**: Think of it as Layer-7 LoadBalancer configured into the Kubernetes cluster which handles and routes traffic to different services that are present into the cluster

  
How does Ingress work? How does Ingress Loadbalance? How does it implement SSL? 

To understand this, how would we do it, without using Ingress? By using a reverse proxy like **nginx, and configure it to use SSL certificates.**

Ingress works in kind of the same way, Using **Nginx Ingress Controller**, remember, it doesn't come by default

Some more Ingress Controller are, **HAProxy, Nginx, Traefik.** 

---