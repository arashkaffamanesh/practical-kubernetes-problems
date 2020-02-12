# Practical Kubernetes Problems

## Preparation

### The Golden Kubernetes Tooling and Helpers list

http://bit.ly/kubernetes-tooling-list

Note: Install at least kubectx and kns with krew

### Kubernauts Kubernetes Learning Resources List

https://goo.gl/Rywkpd

### Kubernauts Kubernetes Trainings Slides

https://goo.gl/Hzk2sd


### Useful aliases

<details><summary>Expand here to see the solution</summary>
<p>

```yaml
alias k="kubectl"
alias kx="kubectx"
alias kn="kubens"
alias kgel="k get events --sort-by=.metadata.creationTimestamp"
```

</p>
</details>

## KUBECTL CheatSheet and GOODIES

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

https://github.com/dennyzhang/cheatsheet-kubernetes-A4

<details><summary>Expand here to see the solution</summary>
<p>

k get events --sort-by=.metadata.creationTimestamp # List Events sorted by timestamp

k get services --sort-by=.metadata.name # List Services Sorted by Name

k get pods --sort-by=.metadata.name

k get endpoints

k explain pods,svc

k get pods -A # --all-namespaces 

k get nodes -o jsonpath='{.items[*].spec.podCIDR}'

k get pods -o wide

k get pod my-pod -o yaml --export > my-pod.yaml

k get pods --show-labels # Show labels for all pods (or other objects)

k get pods --sort-by='.status.containerStatuses[0].restartCount'

k cluster-info

k api-resources

k get apiservice

</p>
</details>

### k create namespace imperative via declarative

<details><summary>Expand here to see the solution</summary>
<p>

```yaml
k create ns <namespace name, e.g. your name or your project!>
k create ns --dry-run test -o yaml > test-ns.yaml
k create -f test-ns.yaml
k delete ns test
```

</p>
</details>

### k create / run pods or deploymens with dry-run

<details><summary>Expand here to see the solution</summary>
<p>

```yaml
k run --generator=run-pod/v1 <pod name> --image=<image name> --dry-run -o yaml > <podname.yaml>

or

k run --restart=Never <pod name> --image=<image name> --dry-run -o yaml > <podname.yaml>

k run --generator=run-pod/v1 "nginx-pod" --image=nginx -o yaml --dry-run > nginx-pod.yaml

k create <object> <name> <options> --dry-run -o yaml > <objectname.yaml>

k create deployment nginx-deployment --image=nginx --dry-run -o yaml > nginx-deployment.yaml

cat nginx-pod.yaml

cat nginx-deployment.yaml

k create -f nginx-pod.yaml

# create a service via exposing the pod

k expose pod nginx-pod --port=80

k get svc

k port-forward service/nginx-pod 8080:80

# open a new terminal session

curl http://127.0.0.1:8080/

k delete all --all # with caution!!!

k create -f nginx-deployment.yaml

k get all

k get all -A

k expose deployment nginx-deployment --port=80

k port-forward service/nginx-deployment 8080:80

k scale --replicas 3 deployment nginx-deployment

k edit deployment nginx-deployment

vi nginx-deployment.yaml # adapt the number of replicas, e.g. to 2

k apply -f nginx-deployment.yaml

```

</p>
</details>

### k get events and logs, describe objects

<details><summary>Expand here to see the solution</summary>
<p>

```yaml
kx
kn
k delete all --all # with caution!!!
k apply -f 0-nginx-all.yaml
k get all
# where is the ingress?
k get ingress # ingress objects are not namespaced
k get events
k get events -A
k get events -n <namespace name>
k logs nginx-<press tab>
k describe pod nginx-<press tab>
k describe deployment nginx
k describe replicasets nginx-<press tab>
```
</p>
</details>

### Merging contexts (e.g. merge 2 kubeconfigs from 2 cluster contexts)

<details><summary>Expand here to see the solution</summary>
<p>

Sometimes you'll need to merge multiple kubeconfigs into a single file, here you go:

```bash
KUBECONFIG=file1:file2:file3 kubectl config view --merge --flatten > my_new_kubeconfig
or
cp ~/.kube/config ~/.kube/config.bak
KUBECONFIG=/my/new/kubeconfig:~/.kube/config.bak kubectl config view --flatten > my_new_kubeconfig
# test it
export KUBECONFIG=my_new_kubeconfig
kx
cp my_new_kubeconfig ~/.kube/config
```
</p>
</details>

Don't miss: Mastering the KUBECONFIG file by Ahmet Alp Balkan:

https://ahmet.im/blog/mastering-kubeconfig/

### Kubernetes Secrets are not secret

Secrets are resources containing keys with base64 encoded values. Secrets are not encrypted by default, they are only encoded and can get decoded easily by everyone who has access to a namespace or to the whole cluster.

Secret values can be exposed to pods as environment variables or mounted as files.

In order to create a secret from a text file, you can run the following, This creates a generic secret named secretname and automatically encodes the value as base64:

<details><summary>Expand here to see the solution</summary>
<p>

```yaml
echo -n "yourvalue" > ./secret.txt
k create secret generic secretname --from-file=./secret.txt
k describe secrets secretname
k get secret secretname -o yaml
echo 'eW91cnZhbHVl' | base64 --decode
# or
k create secret generic mysecret --dry-run -o yaml --from-file=./secret.txt > secret.yaml
k create -f secret.yaml
# or
k create secret generic mysecret --dry-run -o yaml --from-literal=secret.txt=yourvalue > secret.yaml
```
</p>
</details>

Since K8s secrest are not so secret, there are some ways to keep you secrets secret:

https://learnk8s.io/kubernetes-secrets-in-git

### Kubernetes ConfigMaps

A ConfigMap is an object consisting of key-value pairs which can be injected into your application.

With a ConfigMap you can separate configuration from your Pods. This way, you can prevent hardcoding configuration data.

ConfigMaps are useful for storing and sharing non-sensitive, unencrypted configuration information. Sensitive information should be stored in a Secret instead.

Exercise:

Create a ConfigMap named kubernauts that contains a key named dev with the value ops.

With the --from-literal argument passed to the k create configmap command you can create a ConfigMap containing a text value.

<details><summary>Expand here to see the solution</summary>
<p>

```yaml
k create cm kubernauts --from-literal=dev=ops --dry-run -o yaml > cm-kubernauts.yaml
cat cm-kubernauts.yaml
echo -n "ops" > dev
k create cm kubernauts --from-file=./dev
k get cm
k describe cm kubernauts
k delete cm kubernauts
k create -f cm-kubernauts.yaml
k describe cm kubernauts
```
</p>
</details>

Using this ConfigMap, we can inject data in our application:

<details><summary>Expand here to see the solution</summary>
<p>

```yaml
cat 0-nginx-configmap.yaml
k create -f 0-nginx-configmap.yaml
```
</p>
</details>



## Whoami, Whoareyou and Whereami Problems

### What We’ll Do

We’ll use a pre-made container — containous/whoami — capable of telling you where it is hosted and what it receives when you call it.

If you'd like to build the container image with docker, do:

<details><summary>Expand here to see the solution</summary>
<p>

```yaml
git clone https://github.com/containous/whoami.git
docker build -t whoami .
docker tag whoami kubernautslabs/whoami
docker push kubernautslabs/whoami
docker images | head
```

</p>
</details>

We’ll define two different deployments, a whoami and a whoareyou deployment that will use `containous/whoami` container image.

We’ll create a deployment to ask Kubernetes to deploy 2 replicas of whoami and 3 replicas of whoareyou.

We’ll define two services, one for each of our Pods.

We’ll define Ingress objects to define the routes to our services to the outside world.

We’ll use our Nginx Ingress Controller on our Rancher Cluster.

Explanations about the file content of whoami-deployment.yaml:

We define a “deployment” (kind: Deployment)

The name of the object is “whoami-deployment” (name: whoami-deployment)

We want two replica (replicas: 2)

It will deploy pods that have the label app:whoami (selector: matchLabels: app:whoami)

Then we define the pods with the (template: …) which will have the whoami label (metadata:labels:app:whoami)

The Pods will host a container using the image containous/whoami (image:containous/whoami)

<details><summary>Expand here to see the solution</summary>
<p>

```yaml
k apply -f 1-whoami-deployment.yaml
k get all
# we expose the deployment with a service of type ClusterIP
k create -f 1-whoami-service-ClusterIP.yaml
k get svc
k port-forward service/whoami-service 8080:80
# in a new terminal session call
curl 127.0.0.1:8080
k delete svc whoami-service
# create a service of type NodePort
k create -f 1-whoami-service-nodeport.yaml
k get svc
curl csky08:30056 # adapt the nodeport for your env. please !
curl csky09:30056
curl csky10:30056
k delete svc whoami-service-nodeport
k create -f 1-whoami-service-loadbalancer.yaml
k get svc
curl <EXTERNAL-IP> # the external-ip is given from the LB IP pool above
k create -f 2-whoareyou-all.yml
k get all
k get svc
k get ing
curl <HOSTS value from ingress>
# are you happy? ;-)
```

</p>
</details>

## DNS based Service discovery with whereami kubia pod

### What We’ll Do

We'll use a slightly extended node.js app (which is a simple web server) from the Kubernetes in Action book by Marko Lukša in 2 different namespaces ns1 and ns2 to demonstrate the DNS based services discovery. 

A service provides a Virtual IP (VIP) address, which means the Service IP is not bound to a physical network interface. A service acts like an internal loadbalancer in K8s! The magic of of routing trafic through the VIP is implemented by IPtable rules managed by kube-proxy!

A service can be called through its FQDN in the form of:

`$SERVICE_NAME.$NAMESPACE.svc.cluster.local`

<details><summary>Expand here to see the solution</summary>
<p>

```yaml
cd whereami
k create ns ns1
k create ns ns2
kn ns1
k create -f  
k create -f kubia-deployment.yaml
k create -f kubia-deployment.yaml -n ns2
k expose deployment kubia
k get svc
k expose deployment kubia -n ns2
k get svc -n ns2
k exec -it kubia-<press tab> -- curl kubia.ns2.svc.cluster.local:8080
k scale deployment kubia -n ns2 --replicas 3
# repeat the service call many times and see how loadbalancing works
k exec -it kubia-<press tab> -- curl kubia.ns2.svc.cluster.local:8080
k exec -n ns2 -it kubia-<press tab> -- curl kubia.ns1.svc.cluster.local:8080
k exec -it kubia-<press tab> -- ping kubia.ns2.svc.cluster.local
--> PING kubia.ns2.svc.cluster.local (10.43.109.89) 56(84) bytes of data.
# you don't get any pong, why?
```
</p>
</details>

## Multi-Container Pods

Create a Pod with two containers, both with image alpine and command "echo hello; sleep 3600". Connect to the second container and run 'ls'.

The easiest way to do it is create a pod with a single container and save its definition in a YAML file and extend it with an additional container:

<details><summary>Expand here to see the solution</summary>
<p>

```yaml
k run alpine-2-containers --image=alpine --restart=Never -o yaml --dry-run -- /bin/sh -c 'echo hello;sleep 3600' > alpine-pod.yaml
```

Copy/paste the container related values, so your final YAML should contain the following two containers (make sure those containers have a different name):

```YAML
containers:
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: alpine
    name: alpine1
    resources: {}
  - args:
    - /bin/sh
    - -c
    - echo hello;sleep 3600
    image: alpine
    name: alpine2
    resources: {}
```

```yaml
k create -f alpine-pod-2-containers.yaml # alpine-pod-2-containers.yaml is in this repo
# exec / ssh into to the alpine2 container
k exec -it alpine-2-containers -c alpine2 -- sh
ls
exit

# or just an one-liner
k exec -it alpine2 -c alpine2 -- ls

# cleanup
k delete pod alpine-2-containers
```

</p>
</details>


### Shared Volume

We'll extend the above alpine-2-containers with a shared volume of type emptyDir named `share` with a volumeMount for each container with a mountPath `/tmp/share1` and `/tmp/share2` as follow:

<details><summary>Expand here to see the solution</summary>
<p>

```yaml
cat alpine-pod-share-volumes.yaml
k apply -f alpine-pod-share-volumes.yaml
k exec -it alpine-2-containers-share-volume -c alpine1 -- sh
touch /tmp/share1/sharefile
echo "test-share1" > /tmp/share1/sharefile
cat /tmp/share1/sharefile
exit
k exec -it alpine-2-containers-share-volume -c alpine2 -- cat /tmp/share2/sharefile
```

</p>
</details>

## Security

Kubernetes Security is a huge topic and security hardening is a nice problem which everyone has to implement according to their security requirements and governance model of their organization. We're going only to scratch the surface of K8s security here and highly recommend to go through the following resources by Michael Hausenblas, Liz Rice and the community.

https://kubernetes-security.info/

https://github.com/k8s-sec/k8s-sec.github.io


### RBAC (Role Based Access Control)

RBAC in K8s is activated by default and helps to provide access to resources (objects) like namespaces, pods, services, etc. to those Subjects like users, group or service accounts who need access to some resources and deny access to other resources who do not need access to them. RBAC increases security in K8s projects and shall be defined through a governance model in each organization (but in the theorie, you know we are all admins ;-)).

RBAC is implemented through Role, ClusterRole, RoleBinding, and ClusterRoleBinding.

#### Role

A Role defines what you or a subject can do to a set of resources, like get, set, delete, etc.A Role contains a set of rules which define a set of permissions. Roles are used to assigning permissions to resources on the namespace level.

#### ClusterRole

Similar to Role, ClusterRole can grant permissions on the Cluster Level such as giving resource permissions across all namespaces in the cluster.

#### RoleBinding and ClusterRoleBinding

RoleBinding and ClusterRoleBinding are used to grant permissions and priviledges to Subjects or Entities on the namespace (project RoleBinding) level or on the cluster level (ClusterRoleBinding).

<details><summary>Expand here to see the solution</summary>
<p>

```yaml
k get clusterroles | wc -l
k get clusterroles
k describe clusterrole view
k describe clusterrole view | grep pods
# the view role allows your application access to many other resources such as deployments and services.
k create namespace secapp
k -n secapp create role podreader --verb=get --verb=list --resource=pods
k -n secapp describe role/podreader
# nice, the role podview can only view now, but we need to attach the role podview to our application, represented by the service account myappid. 
k -n secapp create rolebinding mypodviewer --role=podreader --serviceaccount=secapp:myappid
k -n secapp describe rolebindings mypodviewer
k -n secapp auth can-i --as=system:serviceaccount:secapp:myappid list pods
# yes :-)
k -n secapp auth can-i --as=system:serviceaccount:secapp:myappid list services
# no :-)
```
</p>
</details>

#### Permission Manager

--> ToDo

## 3-Tier App (MVC)

Please read the README and the related blog post in the  [subfolder](3-tier-app/README.md)  3-tier-app and try to understand and get the todo list app running.

