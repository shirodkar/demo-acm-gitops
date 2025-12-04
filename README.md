# ACM GitOps Demo

This is a GitOps repo which demonstrates the features of Red Hat Advanced Cluster Management for Kubernetes (RHACM).

## Prerequisites

- Hub Cluster - Openshift Container Platform 4.20
- 1 or more Managed Clusters - Openshift Container Platform 4.20

## Setup

1. Install Advanced Cluster Management for Kubernetes (ACM) Operator on the Hub cluster, and the MultiClusterHub instance.
2. Install Openshift GitOps (ArgoCD) Operator on the Hub cluster.
3. Make sure you are logged into the OCP cluster as a cluster admin.
4. Give ArgoCD access to the OCP cluster:

```oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller --rolebinding-name gitops-role-binding```

5. Set up the following users in each cluster - 'admin', 'developer'. 
Note: RBAC for these users will be automatically set up via GitOps in the next section.

## Install the Platform components using GitOps

1. ```git clone https://github.com/shirodkar/acm-gitops-demo.git```
2. ```cd acm-gitops-demo```
3. Make sure you are logged into the OCP cluster as a cluster admin.
4. Run the command: 
```oc apply -f gitops/platform/app-of-apps/applications.yaml```

**Note:** Wait for the 'hub' app in Cluster Argo CD to sync up.

5. In the Openshift Console, switch to ACM using the 'Fleet Management' prespective.
6. Add the 'local-cluster' to the 'hub-clusters' cluster set via 'Infrastructure=>Clusters=>Cluster sets'

**Note:** ACM Policies and GitOps manifests will be automatically applied to the Hub cluster. It could take about 5-10 minutes for the installation to complete. Wait until all apps in ArgoCD are Healthy and Synced.

## Import Managed Clusters into RHACM

1. In the Openshift Console, make sure you have switched to ACM using the 'Fleet Management' prespective.
2. Import the Managed cluster(s) into ACM via 'Infrastructure=>Clusters=>Import Cluster'
  - Make sure to add the cluster to one of the following ClusterSets: dev, qa, uat, prod
  - Make sure to provide the 'env' label with one of the following values dev, qa, uat, prod. Example: ```env=dev```
  - You can provide the server url and token of your cluster(s) to ACM.

**Note:** ACM Policies and GitOps manifests will be automatically applied to the imported Managed clusters. It could take about 5-10 minutes for the installation to complete on the managed clusters. Wait until all apps in ArgoCD are Healthy and Synced.

**TBD:** On the uat cluster, the Gatekeeper instance is not created automatically. It needs to be manually created for now.

## Demo

### Deploying an application Manually using the ACM Console

1. 'Applications=>Create application=>Argo CD ApplicationSet - Push Model'
2. Enter the following values in the wizard:
    - Name: book-import
    - Argo server: shared-gitops
    - Repository type: Git
    - Git URL: https://github.com/shirodkar/book-import.git
    - Git Revision: master-no-pre-post
    - Git Path: book-import
    - Remote Namespace: book-import
    - Placement: Existing Placement - all-managed-placement
3. Click 'Submit'.
4. Wait for successful deployment in Shared Argo CD at: https://shared-gitops-server-shared-gitops.<base url of hub cluster>
5. View the ACM Topology at 'Applications=>book-import=>Topology'

### Using Gatekeeper (OPA Policy) for controlling Deployment (in uat)

1. Verify that the 'book-import' application is not deployed in uat.
2. Check the ACM Governance Policy that requires at least 10 replicas. This Gatekeeper OPA policy is active in uat only.

### Deploying an application using GitOps and visualizing it in ACM

1. Log into the Hub cluster from command line as 'developer'.
2. Run the following command to deploy the 'rollouts-demo' application in all environments: 
```oc apply -f gitops/shared/app-of-apps/applications.yaml```
3. Wait for successful deployment in Shared Argo CD at: https://shared-gitops-server-shared-gitops.<base url of hub cluster>
4. View the ApplicationSets YAML manifests.

#### Push Model (in dev)

1. View the ACM Topology at 'Applications=>demo-rollouts-dev=>Topology'

#### Pull Model (in qa)

1. View the ACM Topology at 'Applications=>demo-rollouts-uat=>Topology'
2. View the apps in Shared Argo CD.
3. View the ACM annotations in the YAML manifest.

### Deploying an Application using Progressive Delivery (in prod)

1. View the ACM Topology at 'Applications=>demo-rollouts-prod=>Topology'
2. View the apps in Shared Argo CD.
3. View the paused status in Argo Rollouts UI.
    - Log into the prod cluster as a cluster admin
    - Run the command: ```oc argo rollouts dashboard```
    - Launch the provided url in a browser, and select the 'demo-rollouts' project, on the top right.
4. View the application UI. It should show partial rollout - 20%.
5. Promote the rollout, and watch the application UI progress to 100% rollout.