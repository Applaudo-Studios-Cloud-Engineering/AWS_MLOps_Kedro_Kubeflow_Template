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


## Documentation

| Tool | Link |
| ------ | ------ |
<!-- | gcloud CLI| <https://cloud.google.com/sdk/docs/initializing>|
| Ingress NGINX controller | <https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx> |
| Cert-manager | <https://artifacthub.io/packages/helm/cert-manager/cert-manager> | -->
| Kedro Kubeflow Plugin | <https://kedro-kubeflow.readthedocs.io/en/latest/source/02_installation/01_installation.html> |