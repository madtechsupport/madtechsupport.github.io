---
layout: post
title:  "Learning Kubernetes Part I"
date:   2023-02-10 13:00:00 +0000
---
One of the most sought after skill sets at the moment is Kubernetes experience. So I thought I'd spin up a little project to explore Kubernetes and see what all the fuss is about. 

## Cover the basics
I'm using this page to cherry pick learning resources: https://www.containiq.com/post/kubernetes-projects-for-beginners

Watched the video first [https://www.youtube.com/watch?v=2gDvViyEtJ0](https://www.youtube.com/watch?v=2gDvViyEtJ0) it's mainly a Docker rapid start video that touches on Kubernetes at the end. But still, some nice ideas for commands to run. And I learnt about Google Borg and Omega and found this document too: https://research.google/pubs/pub44843/ "containers originall chroot" - oh yeah that makes sense!

### Installing minikube
* The video installs `kubectl` first before installing `minikube`... why? `kubectl` is the command line client that talks to the Kubernetes API. Turns out that `kubectl` is available as part of `minikube` and installing it separately is probably more for convinience. It's easier to type `kubectl get pods -A` rather than `minikube kubectl -- get pods -A`.
* `minikube` is dead easy to build from source. Almost as easy as downloading the binary. There doesn't seem to be a ready made rpm from one of the "built-in" channel available on my Fedora desktop just yet. So after installing some dependencies [`glibc-static` and `dnf install golang`](https://minikube.sigs.k8s.io/docs/contrib/building/binaries/) it was easy to `make` and `install` minikube.
* On Linux (Fedora) I want to use the [KVM driver](https://kubernetes.io/blog/2019/03/28/running-kubernetes-locally-on-linux-with-minikube-now-with-kubernetes-1.14-support/) (that is I want `minikube` to create local VM's for the cluster). I faced some issues with permissions. `minikube` with the `kvm2` driver doesn't like running as root and my non-root user account was not set up with the correct `libvirt` group membership permissions, the same issue as described [here](https://github.com/kubernetes/minikube/issues/5801#issuecomment-583488038).

``` sh
[warren@madtechsupport kubia]$ groups warren
warren : warren libvirt
[warren@madtechsupport kubia]$ sudo minikube start --driver=kvm2
😄  minikube v1.28.0 on Fedora 37
✨  Using the kvm2 driver based on user configuration
🛑  The "kvm2" driver should not be used with root privileges. If you wish to continue as root, use --force.
💡  If you are running minikube within a VM, consider using --driver=none:
📘    https://minikube.sigs.k8s.io/docs/reference/drivers/none/

❌  Exiting due to DRV_AS_ROOT: The "kvm2" driver should not be used with root privileges.

[warren@madtechsupport kubia]$ minikube start --driver=kvm2
😄  minikube v1.28.0 on Fedora 37
✨  Using the kvm2 driver based on user configuration
💾  Downloading driver docker-machine-driver-kvm2:
    > docker-machine-driver-kvm2-...:  65 B / 65 B [---------] 100.00% ? p/s 0s
    > docker-machine-driver-kvm2-...:  12.20 MiB / 12.20 MiB  100.00% 8.65 MiB 
💿  Downloading VM boot image ...
    > minikube-v1.28.0-1668700269...:  65 B / 65 B [---------] 100.00% ? p/s 0s
    > minikube-v1.28.0-1668700269...:  274.70 MiB / 274.70 MiB  100.00% 53.43 M
👍  Starting control plane node minikube in cluster minikube
💾  Downloading Kubernetes v1.25.3 preload ...
    > preloaded-images-k8s-v18-v1...:  385.44 MiB / 385.44 MiB  100.00% 67.64 M
🔥  Creating kvm2 VM (CPUs=2, Memory=3900MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.25.3 on Docker 20.10.21 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
💡  kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
### virsh
I'll use `virsh` to see the VM I'd just created.

``` sh
[warren@madtechsupport kubia]$ virsh list
 Id   Name   State
--------------------

[warren@madtechsupport kubia]$ virsh list --all
 Id   Name           State
-------------------------------
 -    fedora37-wor   shut off
```

Turns out that VM's are using `qemu:///system` and therefore are running as the `qemu` user not the local user that started `minikube` this is the [expected behaviour](https://github.com/kubernetes/minikube/issues/9009) :
``` sh
[warren@madtechsupport kubia]$ sudo virsh list --all
[sudo] password for warren: 
 Id   Name       State
--------------------------
 1    minikube   running
```
### Using minikube

There's a dashboard:

``` sh
[warren@madtechsupport kubia]$ minikube dashboard
🔌  Enabling dashboard ...
    ▪ Using image docker.io/kubernetesui/dashboard:v2.7.0
    ▪ Using image docker.io/kubernetesui/metrics-scraper:v1.0.8
💡  Some dashboard features require the metrics-server addon. To enable all features please run:

	minikube addons enable metrics-server	


🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:42077/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

Trying out some other commands like:
``` sh
[warren@madtechsupport kubia]$ minikube docker-env
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.39.15:2376"
export DOCKER_CERT_PATH="/home/warren/.minikube/certs"
export MINIKUBE_ACTIVE_DOCKERD="minikube"

# To point your shell to minikube's docker-daemon, run:
# eval $(minikube -p minikube docker-env)
```

Add an extra node to the cluster:
``` sh
[warren@madtechsupport kubia]$ minikube node

❌  Exiting due to MK_USAGE: Usage: minikube node [add|start|stop|delete|list]

[warren@madtechsupport kubia]$ minikube node add
😄  Adding node m02 to cluster minikube
❗  Cluster was created without any CNI, adding a node to it might cause broken networking.
👍  Starting worker node minikube-m02 in cluster minikube
🔥  Creating kvm2 VM (CPUs=2, Memory=2200MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.25.3 on Docker 20.10.21 ...
🔎  Verifying Kubernetes components...
🏄  Successfully added m02 to minikube!
```
Try the Docker Env command again:
``` sh
[warren@madtechsupport kubia]$ minikube docker-env

❌  Exiting due to ENV_MULTINODE_CONFLICT: The docker-env command is incompatible with multi-node clusters. Use the 'registry' add-on: https://minikube.sigs.k8s.io/docs/handbook/registry/

```
Ah, I didn't get this error before adding the second node. I was wondering how `docker` would report all the containers runnig across different nodes and now I see that it can't. Only a single node cluster is supported with the `minikube docker-env` command.


What does `minikube tunnel` do?
``` sh
[warren@madtechsupport kubia]$ minikube tunnel
[sudo] password for warren: 
Status:	
	machine: minikube
	pid: 13437
	route: 10.96.0.0/12 -> 192.168.39.165
	minikube: Running
	services: []
    errors: 
		minikube: no errors
		router: no errors
		loadbalancer emulator: no errors

```
So how do I use this tunnel? Hmmm, I think I need a service and an application running to test properly. I'll come back to this after running something in the cluster.

### minikube kubectl
`minikube` has a `minikube kubectl` command built in... So no need to install `kubectl` if you don't want to, plus it seems that the `kubectl` binaries version matches the cluster version. That's probably important, I can imagine that odd stuff will happen if client (`kubectl`) is sending or doesn't know the correct API calls for the server (`minikube` cluster).

``` sh
[warren@madtechsupport kubia]$ minikube kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
minikube       Ready    control-plane   25h   v1.25.3
minikube-m02   Ready    <none>          28m   v1.25.3
```
and for good measure let's compare with the output from `virsh` (remember need `sudo` here):
``` sh
[warren@madtechsupport kubia]$ sudo virsh list --all
[sudo] password for warren: 
 Id   Name           State
------------------------------
 1    minikube       running
 2    minikube-m02   running
```
The node names and the VM names match (one VM for each node).

But, what if I did want to install `kubectl` how would it know which cluster to connect to? After doing some reading about this it turns out that if I did install some other version of `kubectl`, so long as it can ready the Kubernetes config file which, by default, is stored in `.kube/config`, then it should, in theory, just work. I can see stored in the config file, on my workstation in the home directory, the address for the cluster and the path to the certifcate and key for authentication to the API.

### etcd

Another question that came to mind is can I access and use `etcd` without going via the Kubernetes API?

I think the answer is [yes](https://stackoverflow.com/questions/47807892/how-to-access-kubernetes-keys-in-etcd) (although I haven't tested this). The cert and key are on the node where `minikube ssh` can be used to `ssh` into the default node. From there I can see `etcd` running as a process and see the docker containers for it as well.

### minikube run
Next I want to actually `run` something in the cluster. I'm pretty sure this is not the same as "deploying" to the cluster which I think is (or has become) [a different concept](https://stackoverflow.com/questions/57658593/what-is-the-recommended-alternative-to-kubectl-generator-option).

I got caught out by the "unknown flag" error despite the warnings in the `minikube start` output!:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl run kubia --image=madtechsupport/kubia --port=8080
Error: unknown flag: --image
See 'minikube kubectl --help' for usage.
```
It's also mentioned in the [documentation](https://minikube.sigs.k8s.io/docs/handbook/kubectl/) that I should be using `--` to end the command line options passed to `minikube` before starting on the `run` part of the command line. This way:
``` sh
minikube kubectl -- run --help
```
returns all the help text for the `run` option and not the help text for `kubectl` (don't miss the reminder to not forget adding `--` after `kubectl`):
``` sh
[warren@madtechsupport kubia]$ minikube kubectl run --help
Run the Kubernetes client, download it if necessary. Remember -- after kubectl!

This will run the Kubernetes client (kubectl) with the same version as the cluster

Normally it will download a binary matching the host operating system and architecture,
but optionally you can also run it directly on the control plane over the ssh connection.
This can be useful if you cannot run kubectl locally for some reason, like unsupported
host. Please be aware that when using --ssh all paths will apply to the remote machine.

Examples:
minikube kubectl -- --help
minikube kubectl -- get pods --namespace kube-system

Options:
    --ssh=false:
	Use SSH for running kubernetes client on the node

Usage:
  minikube kubectl [flags] [options]

Use "minikube options" for a list of global command-line options (applies to all commands).
```
Running a container in the cluster:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- run kubia --image=madtechsupport/kubia --port=8080
pod/kubia created
```
I checked in both the Kubernetes Dashboard and the output of `kubectl get pods` to see the running container (I should probably say "pod" not container since [pods are the smallest deployable units of computing that you can create and manage in Kubernetes](https://kubernetes.io/docs/concepts/workloads/pods/)). Note: there's the `--all-namespaces` (or shorter `-A`) option available to see all the namespaces (not just `default`):
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- get pods --all-namespaces
NAMESPACE              NAME                                         READY   STATUS    RESTARTS      AGE
default                kubia                                        1/1     Running   0             15m
kube-system            coredns-565d847f94-5jwpt                     1/1     Running   3 (16h ago)   4d21h
kube-system            etcd-minikube                                1/1     Running   3 (16h ago)   4d21h
kube-system            kindnet-frghn                                1/1     Running   2 (16h ago)   3d20h
kube-system            kindnet-wpw5g                                1/1     Running   2 (16h ago)   3d20h
kube-system            kube-apiserver-minikube                      1/1     Running   3 (16h ago)   4d21h
kube-system            kube-controller-manager-minikube             1/1     Running   3 (16h ago)   4d21h
kube-system            kube-proxy-4fzmx                             1/1     Running   2 (16h ago)   3d20h
kube-system            kube-proxy-vfgx6                             1/1     Running   3 (16h ago)   4d21h
kube-system            kube-scheduler-minikube                      1/1     Running   3 (16h ago)   4d21h
kube-system            metrics-server-56c6cfbdd9-78bfn              1/1     Running   0             50m
kube-system            storage-provisioner                          1/1     Running   5 (50m ago)   4d21h
kubernetes-dashboard   dashboard-metrics-scraper-5f5c79dd8f-j2vvg   1/1     Running   3 (16h ago)   4d20h
kubernetes-dashboard   kubernetes-dashboard-f87d45d87-9m44f         1/1     Running   3 (16h ago)   4d20h
```
I was looking for the same "all namespace" view in the Kubernetes Dashboard. It took a moment, but to the left of the search box is a drop-down selector for the namespace.

![Kubernetes Dashboard namespaces dropdown screenshot ](/assets/posts/2023-02-02-learning-kubernetes.png)

Reading up on namespaces the take away I got was that "default" namespaces will always be present, use namespaces for Production environments (so don't use "default" in the Prod env) and use namespaces for multi-tenancy. When there is only a single tenant then no need for namespaces.

### minikube networking
Now that there is a pod running inside of the cluster the next step will be to interact with it over the network.

The "kubia" (example application created as part of the video linked at the top of the page) container was run with the `--port=8080` option (following the example in the video). I'm wondering to myself what that means for the pod and the cluster? I'll investigate with `get pods`:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
kubia   1/1     Running   0          52m   172.17.0.2   minikube-m02   <none>           <none>
[warren@madtechsupport kubia]$ minikube kubectl -- describe pods/kubia
Name:             kubia
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube-m02/192.168.39.232
Start Time:       Wed, 18 Jan 2023 12:40:33 +0000
Labels:           run=kubia
Annotations:      <none>
Status:           Running
IP:               172.17.0.2
IPs:
  IP:  172.17.0.2
Containers:
  kubia:
    Container ID:   docker://89fee508c8309faab158e7101002b8ac014a0ada0d7c99694902153eec9add63
    Image:          madtechsupport/kubia
    Image ID:       docker-pullable://madtechsupport/kubia@sha256:d2b8d4cad234f94365658e08db36cba1cf91c340009278d38ad2f158c3db488f
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 18 Jan 2023 12:40:52 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zm5g2 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-zm5g2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  53m   default-scheduler  Successfully assigned default/kubia to minikube-m02
  Normal  Pulling    53m   kubelet            Pulling image "madtechsupport/kubia"
  Normal  Pulled     52m   kubelet            Successfully pulled image "madtechsupport/kubia" in 17.750059878s
  Normal  Created    52m   kubelet            Created container kubia
  Normal  Started    52m   kubelet            Started container kubia
```
Port 8080 gets a mention in the `get pods` output, specifically in relation to the container. Next I'll have a look at the listening ports on the node where the container is running:
``` sh
[warren@madtechsupport kubia]$ minikube ssh -n minikube-m02 -- ss -napt
State                   Recv-Q                   Send-Q                                               Local Address:Port                                                 Peer Address:Port                   Process                  
LISTEN                  0                        0                                                          0.0.0.0:53525                                                     0.0.0.0:*                                               
LISTEN                  0                        0                                                       127.0.0.53:53                                                        0.0.0.0:*                                               
LISTEN                  0                        0                                                          0.0.0.0:22                                                        0.0.0.0:*                                               
LISTEN                  0                        0                                                          0.0.0.0:32793                                                     0.0.0.0:*                                               
LISTEN                  0                        0                                                          0.0.0.0:2049                                                      0.0.0.0:*                                               
LISTEN                  0                        0                                                        127.0.0.1:10248                                                     0.0.0.0:*                                               
LISTEN                  0                        0                                                          0.0.0.0:35883                                                     0.0.0.0:*                                               
LISTEN                  0                        0                                                          0.0.0.0:5355                                                      0.0.0.0:*                                               
LISTEN                  0                        0                                                          0.0.0.0:43277                                                     0.0.0.0:*                                               
LISTEN                  0                        0                                                          0.0.0.0:111                                                       0.0.0.0:*                                               
LISTEN                  0                        0                                                          0.0.0.0:40979                                                     0.0.0.0:*                                               
ESTAB                   0                        0                                                        127.0.0.1:33168                                                   127.0.0.1:36075                                           
ESTAB                   0                        0                                                   192.168.39.232:46606                                              192.168.39.165:8443                                            
ESTAB                   0                        0                                                   192.168.39.232:46610                                              192.168.39.165:8443                                            
ESTAB                   0                        72                                                  192.168.39.232:22                                                   192.168.39.1:59918                                           
ESTAB                   0                        0                                                   192.168.124.20:60952                                                   10.96.0.1:443                                             
LISTEN                  0                        0                                                                *:48405                                                           *:*                                               
LISTEN                  0                        0                                                                *:41749                                                           *:*                                               
LISTEN                  0                        0                                                                *:22                                                              *:*                                               
LISTEN                  0                        0                                                                *:37087                                                           *:*                                               
LISTEN                  0                        0                                                                *:2049                                                            *:*                                               
LISTEN                  0                        0                                                                *:42053                                                           *:*                                               
LISTEN                  0                        0                                                                *:2376                                                            *:*                                               
LISTEN                  0                        0                                                                *:10249                                                           *:*                                               
LISTEN                  0                        0                                                                *:10250                                                           *:*                                               
LISTEN                  0                        0                                                                *:36075                                                           *:*                                               
LISTEN                  0                        0                                                                *:5355                                                            *:*                                               
LISTEN                  0                        0                                                                *:111                                                             *:*                                               
LISTEN                  0                        0                                                                *:10256                                                           *:*                                               
LISTEN                  0                        0                                                                *:44787                                                           *:*                                               
ESTAB                   0                        0                                               [::ffff:127.0.0.1]:36075                                          [::ffff:127.0.0.1]:33168                                           
ESTAB                   0                        0                                          [::ffff:192.168.39.232]:10250                                     [::ffff:192.168.39.165]:39664                                           
ESTAB                   0                        0                                          [::ffff:192.168.39.232]:10250                                     [::ffff:192.168.39.165]:53808                                           
```
I should have used `grep` but there is nothing listening on port 8080 in the node that I can see.
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- exec kubia -- ss -napt
State      Recv-Q Send-Q        Local Address:Port          Peer Address:Port 
LISTEN     0      0                        :::8080                    :::*      users:(("node",pid=1,fd=10))
```
Whereas within the container port 8080 is in use. There's no mapping to the VM or anywhere else that I can see. I assume there is a way to map to a port on a container and after reading up on this it seems that there are a number of ways to expose the container port externally. I found a short [video summary here](https://www.youtube.com/watch?v=UzCgEVqIhg0) with plenty more documentation available online, I booked marked:  
* [https://kubernetes.io/docs/concepts/services-networking/service/]()
* [https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/]()
* [https://www.techtarget.com/searchitoperations/feature/Differences-between-Kubernetes-Ingress-vs-load-balancer]()

Going to see if I can do some kind of "port forwarding".

Checking the services with `get services`:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5d1h
```
This confuses me as I'm seeing "kubernetes" as a service, but I didn't set that up. Turns out I'm not the only one who is [wondering why a "kubernetes" service is present](https://discuss.kubernetes.io/t/what-is-the-use-of-kubernetes-service-in-default-namespace/17545)

Next add `-A` to see all the name spaces:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- get services -A
NAMESPACE              NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                  AGE
default                kubernetes                  ClusterIP   10.96.0.1      <none>        443/TCP                  5d1h
kube-system            kube-dns                    ClusterIP   10.96.0.10     <none>        53/UDP,53/TCP,9153/TCP   5d1h
kube-system            metrics-server              ClusterIP   10.100.197.7   <none>        443/TCP                  23h
kubernetes-dashboard   dashboard-metrics-scraper   ClusterIP   10.99.212.8    <none>        8000/TCP                 5d
kubernetes-dashboard   kubernetes-dashboard        ClusterIP   10.105.3.213   <none>        80/TCP                   5d
```
Trying out `describe services` on the "kubernetes" service it indicates that the "kubernetes" service is the Kubernetes API server:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- describe services kubernetes
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.0.1
IPs:               10.96.0.1
Port:              https  443/TCP
TargetPort:        8443/TCP
Endpoints:         192.168.39.165:8443
Session Affinity:  None
Events:            <none>
```
As per the answer in the link above it is this way this for "legacy" reasons, not by design.

Getting back to the effort to try and expose port 8080 on the container externally, ideally I'll map my workstation localhost:8080 to the container port 8080. The documentation for [Port Forwarding to Access Applications in a Cluster](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) says:
> `kubectl port-forward` allows using resource name, such as a pod name, to select a matching pod to port forward to.

Checking the pod before running commands:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- describe pod kubia
Name:             kubia
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube-m02/192.168.39.232
Start Time:       Wed, 18 Jan 2023 12:40:33 +0000
Labels:           run=kubia
Annotations:      <none>
Status:           Running
IP:               172.17.0.2
IPs:
  IP:  172.17.0.2
Containers:
  kubia:
    Container ID:   docker://89fee508c8309faab158e7101002b8ac014a0ada0d7c99694902153eec9add63
    Image:          madtechsupport/kubia
    Image ID:       docker-pullable://madtechsupport/kubia@sha256:d2b8d4cad234f94365658e08db36cba1cf91c340009278d38ad2f158c3db488f
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 18 Jan 2023 12:40:52 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zm5g2 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-zm5g2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```
Now try and link up port 8080 on the container there with 8080 on the localhost...
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- port-forward kubia 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
```
That's doing something, note need a new terminal now to test (because `minikube` is running in the foreground on the terminal - I should start running these processes in the background).
``` sh
[warren@madtechsupport kubia]$ curl http://localhost:8080
You've hit the Docker container kubia
[warren@madtechsupport kubia]$ 
```
Works and what does the pod description look like now?
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- describe pods/kubia
Name:             kubia
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube-m02/192.168.39.232
Start Time:       Wed, 18 Jan 2023 12:40:33 +0000
Labels:           run=kubia
Annotations:      <none>
Status:           Running
IP:               172.17.0.2
IPs:
  IP:  172.17.0.2
Containers:
  kubia:
    Container ID:   docker://89fee508c8309faab158e7101002b8ac014a0ada0d7c99694902153eec9add63
    Image:          madtechsupport/kubia
    Image ID:       docker-pullable://madtechsupport/kubia@sha256:d2b8d4cad234f94365658e08db36cba1cf91c340009278d38ad2f158c3db488f
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 18 Jan 2023 12:40:52 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zm5g2 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-zm5g2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```
No change from what I can tell... 

Till now, all I've really done is `run` a container inside of a cluster and `port-forward` locally. I think the next step will be to "deploy" to the container.

Before moving on, I've also been wondering about this question [What is the purpose of kubectl proxy?](https://stackoverflow.com/questions/54332972/what-is-the-purpose-of-kubectl-proxy). I think it might have been another way to expose the service running on port 8080 inside the pod externally. Something to look at another time.

### minikube deployment
Time to do a deployment. There are lots of `kubectl create deployment` examples available. However, I'm interested in trying out the "Create from form" that's available in the Dashboard.

But first I need to understanding "stopping" a pod and re-confirm [the difference between `kubectl run` and `kubectl deploy`](https://stackoverflow.com/questions/52890718/kubectl-run-is-deprecated-looking-for-alternative).

For stopping a pod this seemed to work (hints from [here](https://stackoverflow.com/questions/54821044/how-to-stop-pause-a-pod-in-kubernetes)):
``` sh
[warren@madtechsupport kubia]$ minikube kubectl delete pod/kubia
pod "kubia" deleted
[warren@madtechsupport kubia]$ echo $?
0
[warren@madtechsupport kubia]$ minikube kubectl -- get pods -o wide
No resources found in default namespace.
```
Or, just restarting `minikube` does the job too. There was nothing deployed, just a "running" pod.

I used the create from form in the dashboard to set up the deployment:
![Kubernetes Dashboard screenshot](/assets/posts/2023-02-09-learning-kubernetes.png)

After feeding all the deployment values into the "Create from form" I can press the "Preview" button and see the `YAML` or `JSON`, I suppose that's a handy "lazy" way to get started with the `YAML` config? (perhaps, but it only provides the deployment definition but not other things that will need to be created like the service definition).
``` sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubia
  namespace: default
  labels:
    k8s-app: kubia
annotations:
  description: Test deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      k8s-app: kubia
  template:
    metadata:
      name: kubia
      labels:
        k8s-app: kubia
    annotations:
      description: Test deployment
    spec:
      containers:
        - name: kubia
          image: madtechsupport/kubia
          securityContext:
            privileged: false
```
Very cool watching the deployment take place via the dashboard! Next work out how to connect to the service. Starting with:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- get services
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          5d20h
kubia        LoadBalancer   10.110.71.234   <pending>     8080:31987/TCP   3m31s
```
The "Create from form" takes the values entered into the form and creates not only the deployment definition but also the service defintion and anything else required. So a service of type `LoadBalancer` is also created. I want to learn more about the deployment so I'll try `get deployments` to get more information about what was deployed:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
kubia   3/3     3            3           8m51s
[warren@madtechsupport kubia]$ minikube kubectl -- describe deployment kubia
Name:                   kubia
Namespace:              default
CreationTimestamp:      Thu, 19 Jan 2023 11:54:36 +0000
Labels:                 k8s-app=kubia
Annotations:            deployment.kubernetes.io/revision: 1
                        description: Test deployment
Selector:               k8s-app=kubia
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:       k8s-app=kubia
  Annotations:  description: Test deployment
  Containers:
   kubia:
    Image:        madtechsupport/kubia
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   kubia-9b7fdf9f8 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  9m5s  deployment-controller  Scaled up replica set kubia-9b7fdf9f8 to 3
```
No mention there that I can see about a "load balancer" (I think because the creation of the service is not directly connected to the deployment), I notice that the Pod template shows empty Port and Host Port and in the `get services` output there is the appearance of 31987 as a port number. Describing the pod (use the dashboard or `get pods` to get the pod id) also reveals nothing about the container port 8080 or a load balancer:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- describe pod kubia-9b7fdf9f8-47gzq
Name:             kubia-9b7fdf9f8-47gzq
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube-m02/192.168.39.232
Start Time:       Thu, 19 Jan 2023 11:54:36 +0000
Labels:           k8s-app=kubia
                  pod-template-hash=9b7fdf9f8
Annotations:      description: Test deployment
Status:           Running
IP:               172.17.0.3
IPs:
  IP:           172.17.0.3
Controlled By:  ReplicaSet/kubia-9b7fdf9f8
Containers:
  kubia:
    Container ID:   docker://42e33f7773aaa5bfc51527298c87f825be4ba8286bfc3666cc3f281666a7c3aa
    Image:          madtechsupport/kubia
    Image ID:       docker-pullable://madtechsupport/kubia@sha256:d2b8d4cad234f94365658e08db36cba1cf91c340009278d38ad2f158c3db488f
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 19 Jan 2023 11:54:39 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bnjtl (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-bnjtl:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  11m   default-scheduler  Successfully assigned default/kubia-9b7fdf9f8-47gzq to minikube-m02
  Normal  Pulling    11m   kubelet            Pulling image "madtechsupport/kubia"
  Normal  Pulled     11m   kubelet            Successfully pulled image "madtechsupport/kubia" in 1.973817374s
  Normal  Created    11m   kubelet            Created container kubia
  Normal  Started    11m   kubelet            Started container kubia
```
No appearance of a port number for this pod, so how am I going to get this application to work? Well, there was some learning invovled here, but it turns out that there is no need to get into describing what port the container will be listening on, not when Kubernetes will make is so that:
> Any port which is listening on the default "0.0.0.0" address inside a container will be accessible from the network

[Here](https://stackoverflow.com/questions/55741170/container-port-pods-vs-container-port-service) is a good starting place for reading about this. 

Running `get services` again:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- get services
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          6d1h
kubia        LoadBalancer   10.110.71.234   <pending>     8080:31987/TCP   4h20m
```
I can see that the `LoadBalance` is mapping port 8080 on the CLUSTER-IP to port 31987 inside the cluster.

Describe the service:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- describe service kubia
Name:                     kubia
Namespace:                default
Labels:                   k8s-app=kubia
Annotations:              description: Test deployment
Selector:                 k8s-app=kubia
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.110.71.234
IPs:                      10.110.71.234
Port:                     tcp-8080-8080-jh4nc  8080/TCP
TargetPort:               8080/TCP
NodePort:                 tcp-8080-8080-jh4nc  31987/TCP
Endpoints:                172.17.0.6:8080,172.17.0.7:8080,172.17.0.8:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```
Decribing the service is more revealing, I can see mention of both port 8080 and 31987. I had to [ask about this ](https://discuss.kubernetes.io/t/whats-kubectl-describe-service-telling-me-about-the-ports/22932) but the "ports" output from `describe services` can be interpreted as:

```
Port:                     service.spec.ports.name  service.spec.ports.port/service.spec.ports.protocol
TargetPort:               service.spec.ports.targetPort/service.spec.ports.protocol
NodePort:                 service.spec.ports.name  service.spec.ports.nodePort/service.spec.ports.protocol
```

So what we have in the above `describe service` output is:

```
service.spec.ports.name: tcp-8080-8080-jh4nc
service.spec.ports.port: 8080
service.spec.ports.protocol: TCP
service.spec.ports.targetPort: 8080
service.spec.ports.nodePort: 31987
```
An explanation of what the different ports are can be found in the [documentation](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#ports) and in presentations like this [one](https://speakerdeck.com/thockin/kubernetes-a-very-brief-explanation-of-ports).

Combining the above info with what was entered into the "Create from form":

![Kubernetes Dashboard create from form ports screenshot](/assets/posts/2023-02-06-learning-kubernetes.png)

The LoadBalancer service is listening on port 8080 because that was the instruction provided during the "Create from form" setup. This is confirmed using `describe service` where it listed as `Port: tcp-8080-8080-jh4nc  8080/TCP` and in the output of `get services` as well (`8080:31987/TCP`). The port that the containers are using is port 8080, this we know when inspecting the containers and reviewing how they were created (`www.listen(8080);`). We've also instructed the service to target this port (`TargetPort: 8080/TCP`) on the containers (I assume that some containers much have multiple active ports and so Kubernetes will need to know which port to target for which service in such cases). Between the container port and the load balance port Kubernetes takes care of the rest, there was no place to enter the node port in the "Create from form" and so Kubernetes automatically maps the container port (8080) to the node port (31987) in this example. That in my head, at least, completes the chain from container (pod) to node to load balancer.

That's all well and good for the ports but what about the IP addresses for accessing the load balancer. In the `get services` output the EXTERNAL-IP is described as `<pending>`.

Did some more reading about what the [EXTERNAL-IP with status `<pending>`](https://stackoverflow.com/questions/59758707/loadbalancer-ip-and-ingress-ip-status-is-pending-in-kubernetes) means and compared that with the documentation for [creating an external load balancer](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/):
>When creating a Service, you have the option of automatically creating a _**cloud**_ load balancer.

I've decided that `EXTERNAL-IP` with status `<pending>` in `get services` and the absense of [`LoadBalancer Ingress:`](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/#finding-your-ip-address) in `describe services` indicates that there is no load balancer and that is because this cluster is no in a supported cloud environment. So, I'm thinking that I'll need to update the deployment to use a different service type... or not...

After some more reading and playing around I settled on three approaches for making the Kubia (test application) service available to a browser on my local workstation.

The first was just to open the node's INTERNAL-IP (get that from using `get nodes -o wide`) in the browser using the `nodePort` (from `describe service kubia`) for example `curl http://192.168.39.165:31987`. This was quite interesting. For the node with only one pod hitting the node INTERNAL-IP and `nodePort` directly either produces a successful result e.g `You've hit the Docker container kubia-9b7fdf9f8-6c7t4` or `404 page not found` or `url: (7) Failed to connect to 192.168.39.232 port 31987 after 2 ms: Connection refused` in a random order. For the node with two pods hitting the node INTERNAL-IP and `nodePort` directly either produces a successful result that varies between either of the two containers or a third result `curl: (7) Failed to connect to 192.168.39.232 port 31987 after 3063 ms: No route to host`, again in a random order.

The second was to use the [`minikube tunnel`](https://minikube.sigs.k8s.io/docs/handbook/accessing/#using-minikube-service-with-tunnel), from what I can tell, it's a drop in replacement for a "cloud loadbalancer" when working locally with `minikube`.

Turn on `minikube tunnel`:
``` sh
[warren@madtechsupport kubia]$ minikube tunnel
Status:	
	machine: minikube
	pid: 8700
	route: 10.96.0.0/12 -> 192.168.39.165
	minikube: Running
	services: [kubia]
    errors: 
		minikube: no errors
		router: no errors
		loadbalancer emulator: no errors

```
And in a separate terminal check the status of the services:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- get services
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)          AGE
kubernetes      ClusterIP      10.96.0.1       <none>            443/TCP          6d6h
kubia           LoadBalancer   10.110.71.234   10.110.71.234     8080:31987/TCP   9h
```
After enabling `minikube tunnel` the EXTERNAL-IP for the service appears, so does `LoadBalancer Ingress:` in the `describe service kubia` output and the new "External Endpoint" is reachable from the local network
``` sh
[warren@madtechsupport kubia]$ curl http://10.110.71.234:8080
You've hit the Docker container kubia-9b7fdf9f8-6c7t4
[warren@madtechsupport kubia]$ curl http://10.110.71.234:8080
You've hit the Docker container kubia-9b7fdf9f8-6c7t4
[warren@madtechsupport kubia]$ curl http://10.110.71.234:8080
You've hit the Docker container kubia-9b7fdf9f8-69q2s
[warren@madtechsupport kubia]$ curl http://10.110.71.234:8080
You've hit the Docker container kubia-9b7fdf9f8-j7lt5
[warren@madtechsupport kubia]$ curl http://10.110.71.234:8080
You've hit the Docker container kubia-9b7fdf9f8-6c7t4
[warren@madtechsupport kubia]$ curl http://10.110.71.234:8080
You've hit the Docker container kubia-9b7fdf9f8-6c7t4
[warren@madtechsupport kubia]$ curl http://10.110.71.234:8080
You've hit the Docker container kubia-9b7fdf9f8-6c7t4
[warren@madtechsupport kubia]$ curl http://10.110.71.234:8080
You've hit the Docker container kubia-9b7fdf9f8-69q2s
[warren@madtechsupport kubia]$ curl http://10.110.71.234:8080
You've hit the Docker container kubia-9b7fdf9f8-69q2s
```
The load balancer now works with `minikube tunnel` effectively filling in for the "cloud load balancer" and no matter how many times the url reloaded we always get a valid response from one of the three pods.

The third approach I tried was to create a new kubia-service and make that accessable using the `externalIPs`. It didn't work at first because I made the mistake of assigning "any old IP" address from the external subnet to be the service `externalIPs`. Kubernetes is smart but maybe not _**that**_ smart! Reading the [external IP documentation](https://kubernetes.io/docs/concepts/services-networking/service/#external-ips) it states:
>If there are external IPs that route to one or more cluster nodes, Kubernetes Services can be exposed on those `externalIPs`. 

The Service definition I used was:
``` sh
apiVersion: v1
kind: Service
metadata:
  name: kubia-service
spec:
  selector:
    k8s-app: kubia
  ports:
    - name: http
      protocol: TCP
      port: 8080
      targetPort: 8080
  externalIPs:
    - 192.168.39.119
```
With the `externalIPs` in the `kubia-service` set to be the IP of the `minikube` node and that started working too:
``` sh
[warren@madtechsupport kubia]$ minikube kubectl -- get services
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)          AGE
kubernetes      ClusterIP      10.96.0.1       <none>           443/TCP          6d6h
kubia           LoadBalancer   10.110.71.234   10.110.71.234    8080:31987/TCP   9h
kubia-service   ClusterIP      10.109.244.92   192.168.39.165   8080/TCP         3h50m
[warren@madtechsupport kubia]$ curl http://192.168.39.165:8080
You've hit the Docker container kubia-9b7fdf9f8-69q2s
[warren@madtechsupport kubia]$ curl http://192.168.39.165:8080
You've hit the Docker container kubia-9b7fdf9f8-6c7t4
[warren@madtechsupport kubia]$ curl http://192.168.39.165:8080
You've hit the Docker container kubia-9b7fdf9f8-6c7t4
[warren@madtechsupport kubia]$ curl http://192.168.39.165:8080
You've hit the Docker container kubia-9b7fdf9f8-j7lt5
[warren@madtechsupport kubia]$ curl http://192.168.39.165:8080
You've hit the Docker container kubia-9b7fdf9f8-j7lt5
[warren@madtechsupport kubia]$ curl http://192.168.39.165:8080
You've hit the Docker container kubia-9b7fdf9f8-6c7t4
[warren@madtechsupport kubia]$ curl http://192.168.39.165:8080
You've hit the Docker container kubia-9b7fdf9f8-69q2s
[warren@madtechsupport kubia]$ curl http://192.168.39.165:8080
You've hit the Docker container kubia-9b7fdf9f8-j7lt5
```
Interestingly that also works in a load balancer fashion, making all three pods accessible.
