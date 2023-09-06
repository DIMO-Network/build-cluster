# GitHub Actions Cluster Installation Guide

This guide will walk you through the process of setting up a GitHub Actions cluster on Amazon Elastic Kubernetes Service (EKS) with customizable scaling and machine types. This cluster will enable you to run GitHub Actions workflows on your own infrastructure.

## Prerequisites
Before you begin, ensure that you have the following prerequisites in place:

1. [AWS Command Line Interface (CLI)](https://aws.amazon.com/cli/): Make sure you have the AWS CLI installed and configured with the necessary permissions.

2. [eksctl](https://eksctl.io/): Install eksctl, a command-line utility for working with EKS clusters.

3. [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/): Install kubectl, the Kubernetes command-line tool.

4. [Helm](https://helm.sh/docs/intro/install/): Install Helm, the package manager for Kubernetes.

5. [GitHub Personal Access Token](https://github.com/actions/actions-runner-controller/blob/master/docs/authenticating-to-the-github-api.md): You'll need a GitHub Personal Access Token with the necessary permissions to create runners.

## Step 1: Create an AWS KMS Key
```shell
aws kms create-key --description "ActionRunnerKey" --region us-east-1
sed -i 's/YOUR_KEY_ARN/XXXXXXXXXXXXXXXXX/g' cluster_config.yaml
```

## Step 2: Create the EKS Cluster
```shell
eksctl create cluster -f cluster_config.yaml
```
Use your cluster_config.yaml file to create the EKS cluster. Ensure that your configuration includes the desired node instance types and scaling options.

## Step 3: Create namespaces
```shell
kubectl create namespace build-runners
kubectl create namespace build-operators
```
Create two Kubernetes namespaces, one for your build runners and another for the build operators.
## Step 4: Install Certificates Controller
```shell
helm install certificates dimo-certificates -n build-operators
```

Install the Certificates Controller in the build-operators namespace. This controller is essential for securing communication within the cluster.

## Step 5: Install Certificates Controller
```shell
TOKEN=<YOUR_GITHUB_PERSONAL_ACCESS_TOKEN>
helm install build-operators dimo-build-operators -n build-operators --set actions-runner-controller.authSecret.github_token="$TOKEN"
helm install build-runners dimo-build-runners -n build-runners
```
Install the GitHub Actions Runner Controller and GitHub Actions Runner components. Replace <YOUR_GITHUB_PERSONAL_ACCESS_TOKEN> with your actual GitHub Personal Access Token.




## Step 6: Use Custom Runners in Workflows
In your GitHub Actions workflows, you can specify which self-hosted runners should execute specific jobs based on their labels. Here's an example workflow YAML:
```yaml
name: Custom Label Workflow

on: [push]

jobs:
  build:
    runs-on: self-hosted
```
You can also specify additional labels if you need a larger machine, for example:
```yaml
name: Custom Label Workflow

on: [push]

jobs:
  build:
    runs-on: [self-hosted,large]
```



## Step 7: Delete the Cluster (Optional)
```shell
eksctl delete cluster --region=us-east-1 --name=build-cluster
```
If you ever need to delete the EKS cluster, you can use this command. Be cautious, as this will permanently remove the cluster and its resources.

That's it! You have successfully set up a GitHub Actions cluster on Amazon EKS with customizable scaling and machine types.
