# ACI Containers Demo
###### Cisco Data Center Partner VT - October 2019
<hr>
### ACI CNI Operator

So, what if I want this process to be completely automated? There's an operator for that at https://github.com/rtortori/rmlab-aci-operator.

This operator will install a CRD (Custom Resource Definition) in your Kubernetes cluster, extending its API server with a new object called 'AciNamespaces'.
When you create a new 'AciNamespace' in Kubernetes, the following will happen:

1. Based on your resource definition, it will create an EPG in APIC that will inherit contracts from existing EPGs. The network engineers will need to create templates upfront that the kubernetes engineer can leverage
2. It will create a kubernetes namespace 
3. It will annotate the kubernetes namespace so that it will be mapped 1:1 with the new EPG

Let's see how it works, <b>please note that as stated in the repository DISCLAIMER and LICENSE, this is not an official Cisco application and is intended for demostration purposes only.</b>

Clone the repository from GitHub:

```bash
➜  ~ > git clone https://github.com/rtortori/rmlab-aci-operator.git
Cloning into 'rmlab-aci-operator'...
remote: Enumerating objects: 82, done.
remote: Counting objects: 100% (82/82), done.
remote: Compressing objects: 100% (57/57), done.
remote: Total 82 (delta 21), reused 74 (delta 13), pack-reused 0
Unpacking objects: 100% (82/82), done.
```

cd into the rmlab-aci-operator and launch the installation script:

```bash
➜  ~ > cd rmlab-aci-operator
➜  rmlab-aci-operator git:(master) > ./install-operator-kubernetes.sh
This implementation is supported on the following platforms:
Kubernetes v1.11.3+
serviceaccount/aci-operator-serviceaccount created
clusterrolebinding.rbac.authorization.k8s.io/aci-operator-crb created
customresourcedefinition.apiextensions.k8s.io/acinamespaces.rmlab.cisco.com created
deployment.apps/aci-operator created
```

We check that the operator pod as been started successfully:

```bash
➜  rmlab-aci-operator git:(master) > k get pod -l name=aci-operator -n kube-system
NAME                           READY   STATUS    RESTARTS   AGE
aci-operator-cb97d8b67-gmqjd   2/2     Running   0          11m
```

We can now move forward and create our desired 'AciNamespaces'. In this example we will create two of them, named 'myapp-frontend' and 'myapp-backend':

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rmlab.cisco.com/v1alpha1
kind: AciNamespace
metadata:
  name: myapp-frontend
spec:
  epgcontractmaster: "kube-default"
EOF
```

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rmlab.cisco.com/v1alpha1
kind: AciNamespace
metadata:
  name: myapp-backend
spec:
  epgcontractmaster: "kube-default"
EOF
```

We can get those 'AciNamespaces' with the command:

```bash
➜  rmlab-aci-operator git:(master) > kubectl get acinamespaces
NAME             AGE
myapp-backend    11s
myapp-frontend   21s
```

and check that the corresponding Kubernetes namespaces have been created:

```bash
➜  rmlab-aci-operator git:(master) > kubectl get namespaces
NAME             STATUS   AGE
ccp              Active   2d19h
default          Active   2d19h
kube-public      Active   2d19h
kube-system      Active   2d19h
myapp-backend    Active   22s
myapp-frontend   Active   32s
```

Let's run two sample PODs on those namespaces so we can check in ACI that they go in the right EPG automatically:

```bash
➜  rmlab-aci-operator git:(master) > kubectl run application-frontend --image nginx --namespace myapp-frontend
kubectl run --generator=deployment/apps.v1beta1 is DEPRECATED and will be removed in a future version. Use kubectl create instead.
deployment.apps/application-frontend created
```

```bash
➜  rmlab-aci-operator git:(master) > kubectl run application-backend --image nginx --namespace myapp-backend
kubectl run --generator=deployment/apps.v1beta1 is DEPRECATED and will be removed in a future version. Use kubectl create instead.
deployment.apps/application-backend created
```

The PODs are up and running in Kubernetes:

```bash
➜  rmlab-aci-operator git:(master) > k get pods -n myapp-frontend
NAME                                    READY   STATUS    RESTARTS   AGE
application-frontend-57445b6b6d-l2nvm   1/1     Running   0          7m21s
➜  rmlab-aci-operator git:(master) > k get pods -n myapp-backend
NAME                                   READY   STATUS    RESTARTS   AGE
application-backend-5d756ff594-jgzc7   1/1     Running   0          44s
```

Back in ACI, we can see that the new EPGs have been created and the application PODs belong to the right EPGs:

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci18.png)

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci19.png)