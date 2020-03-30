---
date: 2020-02-21T13:36:18+01:00
title: "OpenShift with Service Discovery (AWS)"
weight: 15
---

## AWS

### openshift-install and pull-secret

Download the **openshift-install** and _oc_ tools, and copy your _pull secret_ from:

> https://cloud.redhat.com/openshift/install/aws/installer-provisioned

Find more detailed instructions here:

> https://docs.openshift.com/container-platform/4.3/installing/installing_aws/installing-aws-default.html


### Make sure the aws cli is properly installed and configured

Installation instructions

> https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html

```bash
$ aws configure
AWS Access Key ID [None]: ....
AWS Secret Access Key [None]: ....
Default region name [None]: ....
Default output format [None]: text
```

See also for more details:

> https://docs.openshift.com/container-platform/4.3/installing/installing_aws/installing-aws-account.html

### Create and deploy cluster A

In this step you will deploy cluster A, with the default IP CIDRs

| Pod CIDR     | Service CIDR |
|--------------|--------------|
|10.128.0.0/14 |172.30.0.0/16 |


```bash
openshift-install create install-config --dir cluster-a
```

```bash
openshift-install create cluster --dir cluster-a
```

The create cluster step will take some time, you can create Cluster B in parallel if you wish.

### Create and deploy cluster B

In this step you will deploy cluster B, modifying the default IP CIDRs

| Pod CIDR     | Service CIDR |
|--------------|--------------|
|10.132.0.0/14 |172.31.0.0/16 |


```bash
openshift-install create install-config --dir cluster-b
```


Change the POD IP network, please note it’s a /14 range by default so you need to use 
+4 increments for “128”, for example: 10.132.0.0, 10.136.0.0, 10.140.0.0, ...
 
```bash
sed -i 's/10.128.0.0/10.132.0.0/g' cluster-b/install-config.yaml
```

Change the service IP network, this is a /16 range by default, so just use +1 increments
for “30”: for example: 172.31.0.0, 172.32.0.0, 172.33.0.0, ...

```bash
sed -i 's/172.30.0.0/172.31.0.0/g' cluster-b/install-config.yaml
```


And finally deploy

```bash
openshift-install create cluster --dir cluster-b
```

### Make your clusters ready for submariner

Submariner gateway nodes need to be able to accept traffic over ports 4500/UDP and 500/UDP
when using IPSEC. In addition we use port 4800/UDP to encapsulate traffic from the worker nodes
to the gateway nodes and ensuring that Pod IP addresses are preserved.

Additionally, the default Openshift deployments don't allow assigning an elastic public IP
to existing worker nodes, something that it's necessary at least on one end of the IPSEC connections. 

To handle all those details we provide a script that will prepare your AWS OpenShift deployment
for submariner, and will create an additional gateway node with an external IP.

```bash

curl https://raw.githubusercontent.com/submariner-io/submariner/master/tools/openshift/ocp-ipi-aws/prep_for_subm.sh -L -O
chmod a+x ./prep_for_subm.sh

./prep_for_subm.sh cluster-a     # respond yes when terraform asks
./prep_for_subm.sh cluster-b      # respond yes when terraform asks

```

{{% notice info %}}

Please note that the prep_for_subm.sh script has a few pre-requirements, you will need to install: **oc, aws-cli, terraform, and unzip**. 

{{% /notice %}}

### Install subctl

{{< subctl-install >}}

### Install kubefedctl

Download the kubefedctl binary and make it available on your PATH.

```bash
VERSION=0.1.0-rc3
OS=linux
ARCH=amd64
curl -LO https://github.com/kubernetes-sigs/kubefed/releases/download/v${VERSION}/kubefedctl-${VERSION}-${OS}-${ARCH}.tgz
tar -zxvf kubefedctl-*.tgz
chmod u+x kubefedctl
mv kubefedctl ~/.local/bin/
```

### Install Submariner with Service Discovery

{{% notice info %}}

The Lighthouse project is meant only to be used as a development preview. Installing the operator on an Openshift cluster will disable Openshift CVO.

{{% /notice %}}

To install Submariner with multi-cluster service discovery follow the steps below.

#### Create a merged kubeconfig

```bash
export KUBECONFIG=$PWD/cluster-a/auth/kubeconfig:$PWD/cluster-b/auth/kubeconfig
sed -i 's/admin/east/' cluster-a/auth/kubeconfig
sed -i 's/admin/west/' cluster-b/auth/kubeconfig
kubectl config view --flatten > ./merged_kubeconfig
```
#### Use cluster-a as broker with service discovery enabled

```bash
subctl deploy-broker  --kubecontext west --kubeconfig ./merged_kubeconfig --service-discovery
```

#### Join cluster-a and cluster-b to the broker

```bash
subctl join --kubecontext west --kubeconfig ./merged_kubeconfig broker-info.subm --clusterid west --broker-cluster-context west
```

```bash
subctl join --kubecontext east --kubeconfig ./merged_kubeconfig broker-info.subm --clusterid east --broker-cluster-context west
```

####  Verify Deployment
To verify the deployment follow the steps below.

```bash
export KUBECONFIG=./merged_kubeconfig
kubectl --context east create deployment nginx --image=nginx
kubectl --context east expose deployment nginx --port=80
kubectl --context west run --generator=run-pod/v1 tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash
curl nginx
```
