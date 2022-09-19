# Kubeflow in Amazon Elastic Kubernetes Service (EKS)

## Overview

This is your new Kedro project, which was generated using `Kedro 0.18.2`.

Take a look at the [Kedro documentation](https://kedro.readthedocs.io) to get started.

## How to install Kubeflow Pipelines in EKS

### Guide to install Kubeflow Pipelines on EKS using AWS CLI

### Prerequisites

1.1 Kubectl Installation

Kubectl allows you to run commands against Kubernetes clusters. We use it to deploy applications, inspect and manage clusters resources (pods, namespaces e.t.c) and view cluster and node logs. If you have [minikube](https://minikube.sigs.k8s.io/docs/) installed you can skip this step in order to avoid package conflicts as minikube comes bundled with kubectl. We install kubectl using brew by running the following command.

    brew install kubectl

For Windows users you can find the kubectl installation guide [here](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/) and Linux users [here](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/).

1.2 AWS CLI installation

The AWS command line interface (AWS CLI) is an open source tools that enables you to interact with AWS services using commands directly in your command line shell. You can use it to launch and configure instances, deploy Kubernetes clusters. It offers commands that implement functionality equivalent to that provided by the browser-based AWS Management Console from your terminal. To install AWS CLI we run:

     curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"

     sudo installer -pkg ./AWSCLIV2.pkg -target /

The first Curl commands downloads the AWSCLI version 2 package and saves it in the current directory as a file called AWSCLIV2.pkg. The second installer command installs AWS CLI, we must include sudo on the command to grant write permissions to the root folder (/).

For Linux and Windows users you can find the AWS CLI installation guide [here](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

1.3 Configuring AWS CLI

In order for AWS CLI to work as intended we have to add the credentials for our accounts. We will use an access key ID and a secret access key for the AWS CLI configuration. We login into our AWS account (on a browser), and click on the account’s name on the top right of the landing page, we get a drop down menu as show below:

![AWS-Console-1](https://miro.medium.com/max/700/1*Uq_IsBEFGj1Cv7_WfgMydQ.png)

We then click on the security credentials, and get a page similar to the one below. We click on the Access keys drop down section and get:

![AWS-Console-2](https://miro.medium.com/max/700/1*3pouDZmcB3E3dFujyeGwyg.png)

We click the blue “Create New Access Key Button” and on the pop up that appears click on download the access key generated. We then run the command below, and enter the generated access key ID and secret access key when prompted.

    aws configure

In order to confirm the credentials have been saved after running aws configure. We run the following command, and should receive as similar output as shown below.

    ls ~/.aws
    config    credentials

If you check the contents of the credentials file, you should see the values of the access key ID and secret access key you had created.  

1.4 eksctl installation

eksctl is a simple CLI tool that enables creation of clusters on Amazon Elastic Kubernetes Service (EKS), amazon’s new managed Kubernetes services for EC2. In order to install eksctl we run the commands below.

    brew tap weaveworks/tap
    brew install weaveworks/tap/eksctl

The brew tap command allows brew to get access to another repository of formulae (online package browser). To confirm the installation, we run the command below

    eksctl version
    0.83.0

For Linux and Windows users you can find the eksctl installation guide [here](https://github.com/weaveworks/eksctl).

1.5 aws-iam-authenticator installation

Aws-iam-authenticator is a tool that uses AWS IAM credentials to authenticate your Kubernetes cluster. This allows us to use one set of credentials (The ones create in the previous section) for provisioning and updating a created Kubernetes cluster. To install the aws-iam-authenticator, we run:

    brew install aws-iam-authenticator

To test if aws-iam-authenticator works we run and should get:

    aws-iam-authenticator version
    {"Version":"0.5.3","Commit":"a0516fb9ace571024263424f1770e6d861e65d09"}

For Linux and Windows users you can find the aws-iam-authenticator installation guide [here](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html).

### Creating an EKS Cluster using eksctl.

In order to have Kubeflow up and working, we need to have a kubernetes cluster running in Amazon EKS. We use eksctl to create our clusters, We can create a cluster using other tools EKS Console, or via AWS CloudFormation, Terraform, or the AWS Cloud Development Kit (CDK). Using eksctl is quick and simple and saves us the hustle of changing the context of our local kubectl.

NOTE: If you’ve used the other tools to create a EKS cluster, you have to change the context of kubectl. You can find the guide for doing so [here](https://aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-connection/)

We start by declaring some environment variables:

    export AWS_CLUSTER_NAME=kubeflow
    export AWS_REGION=eu-west-1
    export K8S_VERSION=1.20
    export EC2_INSTANCE_TYPE=m5.large 

We specify the name of the cluster, this can be any value you wish. The AWS_REGION variable should be set to the region you’re working in on AWS. We use the K8s (Kubernetes) version 1.20 to avoid compatibility issues with the Kubeflow version 1.2 manifest. For the EC2_INSTANCE_TYPE variable we went with the m5.large but you can use any EC2 instance type you want based on the needs of your end-to-end ml pipeline.

The next step is creating a cluster configuration file for use with eksctl. We’ll use the cat command to do this directly from the terminal, this makes it easier to access the variables we defined earlier. We run the following code snippet in our terminal.

    $ cat << EOF > cluster.yaml
    ---
    apiVersion: eksctl.io/v1alpha5
    kind: ClusterConfig

    metadata:
    name: ${AWS_CLUSTER_NAME}
    version: "${K8S_VERSION}"
    region: ${AWS_REGION}

    managedNodeGroups:
    - name: kubeflow-mng
    desiredCapacity: 3
    instanceType: ${EC2_INSTANCE_TYPE}
    EOF

The above should create a file called cluster.yaml in your current working directory.

Finally we create the cluster, We run the command below to do this. This command can take several minutes to complete.

    eksctl create cluster -f cluster.yaml

To confirm that the cluster is created, we run the command below

    kubectl get nodes --all-namespaces

You should have something similar to:

![Output](https://miro.medium.com/max/700/1*jbxj-j5OrOd3Q2Bd7iidsg.png)

kfctl installation

Note: kfctl is currently available for Linux and Mac-OS users only. If you use Windows, you can install kfctl on Windows Subsystem for Linux (WSL).

kfctl is the control plane for deploying and managing Kubeflow, We have to set it up first before installing Kubeflow. First we’ll define environment variables:

    $ PLATFORM=$(uname)
    $ export PLATFORM
    $ mkdir -p ~/Kubeflow/bin
    $ export KUBEFLOW_TAG=1.2.0
    $ KUBEFLOW_BASE="https://api.github.com/repos/kubeflow/kfctl/releases"
    $ KFCTL_URL=$(curl -s ${KUBEFLOW_BASE} | grep http | grep "${KUBEFLOW_TAG}" | grep -i "${PLATFORM}" | cut -d : -f 2,3 | tr -d '\" ' )
    The above block of commands creates the PLATFORM, KUBEFLOW_TAG, KUBEFLOW_BASE and KFCTL_URL variables which represent the current platform we’re running the commands on (Linux or Darwin), the version of kfctl we want, URL to the all the kfctl releases and URL to the version of kfctl we have specified respectively. We also create the directory ~/Kubeflow/bin to store the kfctl binaries.

The above block of commands creates the PLATFORM, KUBEFLOW_TAG, KUBEFLOW_BASE and KFCTL_URL variables which represent the current platform we’re running the commands on (Linux or Darwin), the version of kfctl we want, URL to the all the kfctl releases and URL to the version of kfctl we have specified respectively. We also create the directory ~/Kubeflow/bin to store the kfctl binaries.

    $ wget "${KFCTL_URL}"
    $ KFCTL_FILE=${KFCTL_URL##*/}
    $ tar -xvf "${KFCTL_FILE}"
    $ mv ./kfctl ~/Kubeflow/bin
    We use the wget tool to download the kfctl compressed tar file, get the downloaded tar file’s name by splitting the URL string defined in the previous command block and save it to the KFCTL_FILE variable then extract it using the tar command. We then move the contents of the tar file (the kfctl program) to the directory we created earlier.

We add the kfctl directory (~/Kubeflow/bin) to the PATH variable so as to use kfctl in the current terminal shell.

    $ export PATH=$PATH:~/Kubeflow/bin

Note: this will allow kfctl to be used in the current terminal shell only, once you close the terminal you will have to run the command again. To add it permanently, you have to add the command to ~/.zshrc or ~/.bashrc depending on the shell you’re using.

To verify kfctl is installed, we run the command below. You should receive same version as below:

    $ kfctl version
    kfctl_v1.2.0-0-gbc038f9

### Kubeflow Installation

Docker and Kubernetes use yaml files for configuration, therefore it shouldn’t came as a surprise that Kubeflow uses yaml files also or as they are called in this context manifests. The manifest files define all the kubeflow services (components) to be deployed in this cluster. Cloud Services usually have different manifests tailored to their platforms. We’ll get the aws manifest from the kubeflow/manifest repo and set an environment variable to it.

    $ export CONFIG_URI="https://raw.githubusercontent.com/kubeflow/manifests/v1.2-branch/kfdef/kfctl_aws.v1.2.0.yaml"

NOTE: You can find other cloud services’ manifest [here](https://github.com/kubeflow/manifests/tree/v1.2-branch/kfdef). For enabling multi-user authentication using [AWS Congito](https://aws.amazon.com/cognito/), you can use the kfctl_aws_cognito.v1.2.0.yaml found in the linked manifest repo above.

Finally we create a deployment directory for our cluster, change to it and download the manifest file.

    $ mkdir ${AWS_CLUSTER_NAME} && cd ${AWS_CLUSTER_NAME}
    wget -O kfctl_aws.yaml $CONFIG_URI

### Kubeflow Configuration

The downloaded manifest file has default values assigned to certain parameters, for example username is set to admin@kubeflow.org and the password is 12341234. To secure our Kubeflow deployment, we have to change this configuration. We edit the kfctl_aws.yaml in VScode (Feel free to use any editor you wish).

Note: The username should be in an email address format (someone@somewhere.something) as that’s what you’ll to login when accessing the Kubeflow UI.

Since v1.0.1, Kubeflow supports use of IAM roles for service accounts.This allows for fine-grained AWS policy configuration bound to a specific service account in your Kubernetes cluster, as opposed to attaching the required policies to the node instance role. In the default configuration, kfctl will create or reuse your cluster’s IAM OIDC Identity Provider, will create the required IAM roles, and configure the trust relationship binding the roles with your Kubernetes Service Accounts. We just have to update the section below in the manifest file with our AWS Region, in our case it should look like.

    region: eu-west-1
    enablePodIamPolicy: true

To initialize the Kubeflow cluster, we run:

    $ kfctl apply -V -f kfctl_aws.yaml

After all the resources are ready, i.e the above command has run to completion. We run the command below to get all the components running in the Kubeflow namespace.

    kubectl -n kubeflow get all
    
After running the command above you should have all Kubeflow resources listed and divided into their various kubernetes categories e.g pods, services, daemonsets, deployments e.t.c. This should resemble:

![Kube-Output](https://miro.medium.com/max/700/1*TIsbbJ_ibx5-sU3fDK6WzA.png)

### Accessing Kubeflow UI

Since Kubeflow is running on AWS, we have to get Kubeflow service’s endpoint host name, We get it by running the command below:

    $ kubectl get ingress -n istio-system

You should get something similar to 123-istiosystem-istio-2af2-4567.us-west-2.elb.amazonaws.com in the address section. Copy paste this into your browser and you’re good to go.

NOTE: if the above command doesn’t show anything in the address section. We do a simple port forwarding using the command kubectl port-forward svc/istio-ingressgateway -n istio-system 7777:80 and enter http://localhost:7777/ in your browser and you should be good to go.

Once the page loads, you’ll be prompted for your email address and password (They should match the values you entered earlier) and after that you should have a page similar to:

![Kube-UI](https://miro.medium.com/max/700/1*S1hRnh432XCsCxxBITtMjg.png)
## Documentation

| Tool | Link |
| ------ | ------ |
| Ingress NGINX controller | <https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx> |
| Cert-manager | <https://artifacthub.io/packages/helm/cert-manager/cert-manager> |
| Kedro Kubeflow Plugin | <https://kedro-kubeflow.readthedocs.io/en/latest/source/02_installation/01_installation.html> |