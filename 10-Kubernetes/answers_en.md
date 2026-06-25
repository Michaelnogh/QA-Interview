This article is under preparation.

Topic is divided into the following sections, I know these are overlapping sections, it's difficult to distinguish between them:

- Administration
- Compute
- Storage
- Network
- Security
- Monitoring
- Logging

#### Administration

A: Maintenance activity are inevitable part of administration, you may need to do the patching or apply some security fixes on K8. Mark the node unschedulable and then drain the PODs which are present on K8 node.

- kubectl cordon <hostname>
- kubectl drain --ignore-daemonsets <hostname>

It's important to include the `--ignore-daemonsets` for any daemonset running on this node. Just in case if any statefulset is running on this node, and if no more node is available to maintain the count of statesful set then statesfulset POD will be in pending status.

A: Pause container servers as the parent container for all the containers in your POD.

- It serves as the basis of Linux namespace sharing in the POD.
- PID 1 for each POD to reap the zombie processes.

https://www.ianlewis.org/en/almighty-pause-container

A: A service mesh ensures that communication among containerized and often ephemeral application infrastructure services is fast, reliable, and secure. The mesh provides critical capabilities including service discovery, load balancing, encryption, observability, traceability, authentication and authorization, and support for the circuit breaker pattern.

A: With requests and limits resource usage of a POD can be control.

request: the amount of resources being requested for a container. If a container exceeds its request for resources, it may be throttled back down to it's request.

limit: an upper cap on the resources a container is able to use. If it tries to exceed this limit it may be terminated if Kubernetes decides that another container needs the resources. If you're sensitive to pod restarts, it makes sense to have the sum of all container resource limits equal or less than the total resource capacity for your cluster.

https://www.noqcks.io/notes/2018/02/03/understanding-kubernetes-resources/

A: CPU is in milicores and memory in bytes. CPU can be easily throttled but not memory.

A: You may also set resource limit on a namespace. This is helpful in scenarios where people have habit of not defining the resource limits in POD definition.

A: Before doing the update of K8, it's important to read the release notes to understand the changes introduced in newer version and whether version update will also update the etcd.

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade-1-12/

A: An Operator is an application-specific controller that extends the Kubernetes API to create, configure and manage instances of complex stateful applications on behalf of a Kubernetes user. It builds upon the basic Kubernetes resource and controller concepts, but also includes domain or application-specific knowledge to automate common tasks better managed by computers. On the other hand, helm is a package manager like yum or apt-get.

A: A custom resource is an extension of the Kubernetes API that is not necessarily available in a default Kubernetes installation. It represents a customization of a particular Kubernetes installation. However, many core Kubernetes functions are now built using custom resources, making Kubernetes more modular.

A: Mainly K8 cluster consists of two type of nodes: master and executor

- master services:

  - kube-apiserver: Master API service which acts like a door to K8 cluster.
  - kube-scheduler: Schedule PODs according to available resources on executor nodes.
  - kube-controller-manager: controller is a control loop that watches the shared state of the cluster through the
    apiserver and makes changes attempting to move the current state towards the desired state

- executor node: (These also runs on master node)

  - kube-proxy: The Kubernetes network proxy runs on each node. This reflects services as defined in the Kubernetes API on
    each node and can do simple TCP, UDP, and SCTP stream forwarding or round robin TCP, UDP, and SCTP forwarding across a set of backends.
  - kubelet: kubelet takes a set of PodSpecs that are provided through various mechanisms (primarily through the
    apiserver) and ensures that the containers described in those PodSpecs are running and healthy

A: kubectl looks for the config file, multiple clusters access information can be specified in this config file. `kubectl config` commands can be used to manage the access to these clusters.

A: A PDB specifies the number of replicas that an application can tolerate having, relative to how many it is intended to have. For example, a Deployment which has a .spec.replicas: 5 is supposed to have 5 pods at any given time. If its PDB allows for there to be 4 at a time, then the Eviction API will allow voluntary disruption of one, but not two pods, at a time. This is applicable for voluntary disruptions.

A: Daemonset are used to start the PODs on every node in cluster. It's used generally to run the monitoring or logging agents which are supposed to run on every executor node in cluster.

A: When you are running the applications which require quorum basically the applications which are not truely stateless for those applications stateful sets are required.

A: init containers will set a stage for you before running the actual POD.

- Wait for some time before starting the app Container with a command like sleep 60.
- Clone a git repository into a volume.

A: In this agile world there is continuous demand of upgrading the applciations, we have multiple options for deploying the new version of app:

1) Recreate: Old style, existing application version is destroyed and new version is deployed. Significant amount of downtime.
2) Rolling update: Gradually bringing down the existing deployment and introducing the new versions. You decide how many instances can be upgraded at single point of time.
3) Shadow: Traffic going to existing version of application is replicated to new version to see it's working. Istio provide this pattern.
4) A/B Testing using Istio: Running multiple variants of application together and determines the best one based on user traffic. It's more for managment decisions.
5) Blue/Green : Blue is mainly about switching the traffic from one version of app to another version.
6) Canary deployment : In which certain percentage of traffic is shifted from one version to another. If things work well we will keep on increasing the traffic shift. It's different from the rolling update in which existing version count is reduced gradually.

#### Compute

A: There are many factors which can led to unstartable POD. Most common one is running out of resources, use the commands like `kubectl desribe <POD> -n <Namespace>` to see the reason why POD is not started. Also, keep an eye on `kubectl get events` to see all events coming from the cluster.

A: Various methods are available to achieve it.

- nodeName: specify the node name in POD spec, it will try to run the POD on specific node.

- nodeSelector : you may assign a specific lable to node which have special resources and use the same label in POD spec so that POD will run only on that node.

- nodeaffinities: requiredDuringSchedulingIgnoredDuringExecution, preferredDuringSchedulingIgnoredDuringExecution are hard, soft requirements for running the POD on specific nodes. This will be replacing nodeSelector in future. It depends on the node labels.

A: podAntiAffinity and podAffinity are the affinity concept to not keep and keep the PODs on same node. Key point to note is that it depends on the POD labels.

A: Taints allow a node to repel a set of pods. You can set taints on the node and only the POD which have tolerations matching the taints condition will be able to run on those nodes. This is useful in the case when you allocated node for one user and don't want to run the PODs from other users on that node.

#### Storage

A: Persistent volumes are used for persistent POD storage. They can be provision statically or dynamically.

Static : A cluster administrator creates a number of PVs. They carry the details of the real storage which is available for use by cluster users.

Dynamically : Administrator creates a PVC (Persistent volume claim) specifying the existing storage class and volume created dynamically based on PVC.

#### Network

A: Kubernetes implements this by creating a special container for each pod whose only purpose is to provide a network interface for the other containers. These is one `pause` container which is responsible for namespace sharing in the POD. Generally, people ignore the existance of this pause container but actually this container is the heart of network and other functionalities of POD. It provides a single virtual interface which is used by all containers running in a POD.

A: By default POD should be able to reach the external world but for vice-versa we need to do some work. Following options are available to connect with POD from outer world.

- Nodeport (it will expose one port on each node to communicate with it)
- Load balancers (L4 layer of TCP/IP protocol)
- Ingress (L7 layer of TCP/IP Protocol)

One another method is kube-proxy which can be used to expose a service with only cluster IP on local system port.

$ kubectl proxy --port=8080
$ http://localhost:8080/api/v1/proxy/namespaces/<NAMESPACE>/services/<SERVICE-NAME>:<PORT-NAME>/

https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0

A: nodport relies on the IP address of your node. Also, you can use the node ports only from the range 30000–32767, on another hand load balancer will have it's own IP address. All the major cloud providers supports creating the LB for you if you specify LB type while creating the service. On baremetal based clusters, metallb is promising.

A: For each service you have one LB. You can have single ingress for multiple services. This will allow you do both path based and subdomain based routing to backend services. You can do the SSL termination at ingress.

A: For POD to POD communication, it's always recommended to use the K8 service DNS instead of POD IP because PODs are ephemeral and their IPs can get change after the redeployment.

If the two PODs are running on a same host then physical interface will not come into the picture.

- Packet will leave POD1 virtual network interface and go to docker bridge (cbr0).
- Docker bridge will forward the packet to right POD2 which is running on same host.

If two PODs are running on a different host then physical interface of both host machines will come into the picture. Let's consider a scenario in which CNI is not used.

POD1 = 192.168.2.10/24 (node1, cbr0 192.168.2.1)
POD2 = 192.168.3.10/24 (node2, cbr1 192.168.3.1)

- POD1 will send the traffic destined for POD2 to it's GW (cbr0) because both are in different subnet.
- GW doesn't know about 192.168.3.0/24 network hence it will forward the traffic to physical interface of node1.
- node1 will forward the traffic to it's own physical rourter/gateway.
- That physical router/GW should have the route for 192.168.3.0/24 network to route the traffic to node2.
- Once traffic reaches node2, it pass that traffic to POD2 through cbr1

If the Calico CNI it's responsible for adding the routes for cbr (docker bridge IP address) in all nodes.

A: PODs are ephemeral their IP address can change hence to communicate with POD in reliable way service is used as a proxy or load balancer. A service is a type of kubernetes resource that causes a proxy to be configured to forward requests to a set of pods. The set of pods that will receive traffic is determined by the selector, which matches labels assigned to the pods when they were created. K8 provides an internal cluster DNS that resolves the service name.

Service is using different internal network than POD network. netfilter rules which are injected by kube-proxy are used to redirect the request actually destined for service IP to right POD.

A: kubelet running on worker node is responsible for detecting the unhealth endpoints, it passes that information to API server then eventually this information is passed to kube-proxy which wil adjust the netfilter rules accordingly.

I highly recommend reading the following series to get solid understanding about the K8 networking.

https://medium.com/google-cloud/understanding-kubernetes-networking-pods-7117dd28727

https://medium.com/google-cloud/understanding-kubernetes-networking-services-f0cb48e4cc82

https://medium.com/google-cloud/understanding-kubernetes-networking-ingress-1bc341c84078

#### Security

A: This is a huge topic, I am sharing some thoughts on it.

- By default, POD can communicate with any other POD, we can setup network policies to limit this communication between the PODs.
- RBAC (Role based access control) to narrow down the permissions.
- Use namespaces to establish security boundaries.
- Set the admission control policies to avoid running the priviledged containers.
- Turn on audit logging.

#### Monitoring

A: Prometheus is used for K8 monitoring. Prometheus ecosystem consists of multiple components.

- main Prometheus server which scrapes and stores time series data
- client libraries for instrumenting application code
- a push gateway for supporting short-lived jobs
- special-purpose exporters for services like HAProxy, StatsD, Graphite, etc.
- an alertmanager to handle alerts
- various support tools

A: You may run multiple instances of prometheus HA but grafana can use only one of them as a datasource. You may put load balancer in front of multiple prometheus instances, use sticky sessions and failover if one of the prometheus instance dies. This make things complicated. Thanos is another project which solve these challenges.

A: Desipte of being very good at K8 monitoring, prometheus still have some issues:

- Prometheus HA support.
- No downsampling is available for collected metrics over the period of time.
- No support for object storage for long term metric retention.

All of these challenges are again overcome by Thanos.

A: The mission of the Prometheus Operator is to make running Prometheus on top of Kubernetes as easy as possible, while preserving configurability as well as making the configuration Kubernetes native.

#### Logging

A: This architecture depends upon application and many other factors. Following are the common logging patters

- Node level logging agent
- Streaming sidecar container
- Sidecar container with logging agent
- Export logs directly from the application

In our setup, filebeat and journalbeat are running as daemonset. Logs collected by these are dumped to kafka topic which are eventually dumped to ELK stack.

Same can be achieved using EFK stack and fluentd-bit.
