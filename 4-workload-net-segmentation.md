# ACI Containers Demo
###### Cisco Data Center Partner VT - October 2019
<hr>

### Workload Network Segmentation

Let's have a look at how those PODs are placed in term of EPGs.
For this specific tenant, under Application Profiles -> Kubernetes, we see all pods (End Points) in the same EPG:

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci14.png)

If we inspect the policies, we see that the kube-default EPG is provider and consumer for a number of EPGs. Most of those EPGs have been created during the ACI integration process:

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci15.png)

How can we move Kubernetes endpoints (PODs) to EPGs and who is entitled to do it?
This is a two-step process:

1. The network admin creates EPGs and contracts based on corporate policies
2. The Kubernetes app admin annotates deployments or namespaces to select what (group of) PODs belong to already existing EPGs

What kind of segmentation we can implement with this approach?

1. Cluster Isolation. This is default behaviour, all PODs of all namespaces in a given cluster go in the kube-default EPG, which by default permits everything to and from the selected l3out
2. Namespace Isolation. All PODs in a given namespace are in the same EPG
3. Deployment Isolation. All PODs managed by a given deployment are in the same EPG

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci-container-seg.png)

Let's have a look. We are already implementing 'Cluster Isolation' as we have everything that belongs to the my-hero app in the kube-default EPG. This has been shown a earlier in APIC but can also easily be seen in Kubernetes using the following command:

```bash
➜  ~ > kubectl get pods -o=custom-columns=POD:'{.metadata.name}',ANNOTATIONS:'{.metadata.annotations.*}'
POD                             ANNOTATIONS
myhero-app-69cf7d6b89-2kmjs     {"policy-space":"emear_pvt","name":"kubernetes|kube-default"},[]
myhero-app-69cf7d6b89-85hrj     [],{"policy-space":"emear_pvt","name":"kubernetes|kube-default"}
myhero-app-69cf7d6b89-dsgh4     {"policy-space":"emear_pvt","name":"kubernetes|kube-default"},[]
myhero-data-7747855b45-f67nc    {"policy-space":"emear_pvt","name":"kubernetes|kube-default"},[]
myhero-ernst-5956cdd775-cdrhg   {"policy-space":"emear_pvt","name":"kubernetes|kube-default"},[]
myhero-mosca-6b7f6b5589-qvqzg   [],{"policy-space":"emear_pvt","name":"kubernetes|kube-default"}
myhero-ui-699645fd79-8n7vt      [],{"policy-space":"emear_pvt","name":"kubernetes|kube-default"}
myhero-ui-699645fd79-qrjrb      {"policy-space":"emear_pvt","name":"kubernetes|kube-default"},[]
nginx-dbddb74b8-dmz5k           {"policy-space":"emear_pvt","name":"kubernetes|kube-default"},[]
```

Let's go in APIC and create a new EPG. For convenience, we will use kube-default as EPG master to inherit all contracts, however in real life deployment your new EPG will reflect your desired policy:

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci16.png)

We created the 'default-namespace' EPG, assigned 'kube-default' EPG as master EPG, mapped to the right Bridge Domain and attached to the right VMM domain.

Let's go in Kubernetes and annotate the 'default' namespace, so that all current and future PODs will fall into this EPG:

```bash
➜  ~ > kubectl annotate namespace default opflex.cisco.com/endpoint-group='{"tenant":"emear_pvt","app-profile":"kubernetes","name":"default-namespace"}' --overwrite
```

If we go in APIC, we can now see all PODs in the default namespace falling in the 'default-namespace' EPG:

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci17.png)

To recap, to move Kubernetes PODs to specific EPGs you need to:

1. Create EPG and contracts in APIC for your tenant in the kubernetes application profile, mapped to the right VMM domain and bridge domain 
2. Annotate deployments or namespace in kubernetes either using kubectl to annotate or leveraging the acikubectl utility which gets automatically installed when you configure ACI integration

What if I want this process to be completely automated?

Next: [ACI-CNI Operator](https://github.com/rtortori/emear-pvt-aci-containers/blob/master/5-aci-cni-operator.md)