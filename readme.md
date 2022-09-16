# Kubeflow in Amazon Elastic Kubernetes Service (EKS)

## Overview

This is your new Kedro project, which was generated using `Kedro 0.18.2`.

Take a look at the [Kedro documentation](https://kedro.readthedocs.io) to get started.

## How to install Kubeflow Pipelines in EKS

### Guide to install Kubeflow Pipelines on EKS using AWS CLI

1. Prerequisites

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


## Documentation

| Tool | Link |
| ------ | ------ |
<!-- | gcloud CLI| <https://cloud.google.com/sdk/docs/initializing>|
| Ingress NGINX controller | <https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx> |
| Cert-manager | <https://artifacthub.io/packages/helm/cert-manager/cert-manager> | -->
| Kedro Kubeflow Plugin | <https://kedro-kubeflow.readthedocs.io/en/latest/source/02_installation/01_installation.html> |