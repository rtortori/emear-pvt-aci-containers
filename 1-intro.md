# ACI Containers Demo
###### Cisco Data Center Partner VT - October 2019
<hr>

### Introduction

This is a detailed step-by-step description of the demo delivered at the Cisco DC EMEAR Partner Virtual Team "ACI Containers" session.<br>

For this demo, we are leveraging the following assets: 

##### Sample Kubernetes Application: My Hero

This is a sample application made by Hank Preston, available on [GitHub](https://github.com/hpreston/myhero_demo).

It exposes a nice UI where you can vote your favourite Superhero. Its backend layer will then show the voting results.

![myhero](https://raw.githubusercontent.com/hpreston/myhero_demo/master/diagrams/myhero-demo-i2.png)

##### Cisco ACI

Our fabric runs Cisco ACI software version 4.1(1l)

##### Kubernetes

We are using Kubernetes v1.12.7 with one master and four worker nodes, deployed by [Cisco Container Platform on Hyperflex](https://www.cisco.com/c/m/en_us/products/cloud-systems-management/container-platform/ccp-hyperFlex.html), though this integration works for any Kubernetes, Red Hat Openshift or Docker EE in the [compatibility matrix](https://www.cisco.com/c/dam/en/us/td/docs/Website/datacenter/aci/virtualization/matrix/virtmatrix.html).

Next: [Explore the environment](https://github.com/rtortori/emear-pvt-aci-containers/blob/master/2-environment.md)

