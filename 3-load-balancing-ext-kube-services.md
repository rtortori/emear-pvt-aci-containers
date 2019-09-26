# ACI Containers Demo
###### Cisco Data Center Partner VT - October 2019
<hr>
### Load Balancing External Kubernetes Services

The sample application we have deployed presents a UI where you can pick your favourite superhero, how can you reach the UI? 

Each deployment has a corresponding service, which basically presents a virtual IP and load balance the traffic to the PODs.
Kubernetes leverages external cloud providers to expose applications to the outside world, in case of ACI-CNI this is done by ACI through Policy Based Redirect performed in hardware. 

Those kind of services in Kubernetes are named 'LoadBalancer' services:

```bash
➜  ~ > kubectl get services
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)          AGE
kubernetes     ClusterIP      10.96.0.1       <none>            443/TCP          2d2h
myhero-app     LoadBalancer   10.97.169.16    192.168.163.131   80:30669/TCP     24h
myhero-data    NodePort       10.97.194.45    <none>            80:32321/TCP     24h
myhero-mosca   NodePort       10.100.85.62    <none>            1883:31712/TCP   24h
myhero-ui      LoadBalancer   10.111.87.251   192.168.163.132   80:31941/TCP     24h
```

If we inspect the UI service, we see the virtual IP external clients need to connect to as well as the PODs used as destinations:

```bash
➜  ~ > kubectl describe svc myhero-ui
Name:                     myhero-ui
Namespace:                default
Labels:                   app=myhero
                          tier=ui
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"myhero","tier":"ui"},"name":"myhero-ui","namespace":"def...
Selector:                 app=myhero,tier=ui
Type:                     LoadBalancer
IP:                       10.111.87.251
LoadBalancer Ingress:     192.168.163.132
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31941/TCP
Endpoints:                1.5.0.107:80,1.5.0.83:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Our UI is accessible from IP 192.168.163.132 (LoadBalancer ingress) and balances pods with label app=myhero and tier=ui. Let's check:

```bash
➜  ~ > kubectl get pods -l app=myhero,tier=ui
NAME                         READY   STATUS    RESTARTS   AGE
myhero-ui-699645fd79-8n7vt   1/1     Running   0          24h
myhero-ui-699645fd79-qrjrb   1/1     Running   0          24h
```

Let's connect to My Hero (http://192.168.163.132):

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/myhero1.png)

Let's vote for Spidey and see what happens (we already voted in this specific example):

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/myhero2.png)

All application tiers work just fine, let's move to ACI and see how the Kubernetes objects we inspected a minute ago appear in APIC as they are seen by the networking team.

Deployments:

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci7.png)

Pods:

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci8.png)

Services. In this case we focus to the UI service:

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci9.png)

Let's check how this Load Balancer service is exposed by ACI. This configuration is under the 'common' tenant. We go under 'Networking' -> 'External Routed Networks' -> Your L3 out network -> Networks and we see our Load Balancer 192.168.163.132 service IP address as seen in Kubernetes.

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci10.png)

Under 'Policies' -> 'Protocol' -> 'L4-L7 Policy-Based Redirect', we see two IP addresses being balanced, in our case 2.3.0.4 and 2.3.0.5. Note that those are not actual IP address but just a ....???

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci11.png)

Let's scale this application tier and see how ACI is starting to Load Balance to additional nodes:

```bash
➜  ~ > kubectl scale deployment myhero-ui --replicas 10
deployment.extensions/myhero-ui scaled
```

APIC view:

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci12.png)

Scaling down back to 1 replica:

```bash
➜  ~ > k scale deployment myhero-ui --replicas 1
deployment.extensions/myhero-ui scaled
```

We see in ACI we are now only forwarding traffic to the worker nodes where that single POD is running:

![](https://raw.githubusercontent.com/rtortori/emear-pvt-aci-containers/master/images/aci13.png)

Next: [Workload Network Segmentation](https://github.com/rtortori/emear-pvt-aci-containers/blob/master/4-workload-net-segmentation.md)