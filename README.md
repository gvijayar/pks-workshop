## Pivotal Container Service Workshop
This is a sample SpringBoot application that performs Geo Bounded queries against an Elastic Search instance and plots the data on a map interactively. This application can be run on a workstation or in a cloud environment such as Cloud Foundry. In this example, I will show how to deploy the application on a running Cloud Foundry instance.
<!-- TOC depthFrom:3 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [1. Install and Setup CLIs](#1-install-and-setup-clis)
	- [Install PKS CLI](#install-pks-cli)
	- [Install kubectl CLI](#install-kubectl-cli)
- [2. Cluster Access and Validation](#2-cluster-access-and-validation)
	- [Get Cluster Credentials](#get-cluster-credentials)
	- [Validating your Cluster](#validating-your-cluster)
	- [Accessing the Dashboard](#accessing-the-dashboard)
- [3. Lab Exercise: Set Environment Variables](#3-lab-exercise-set-environment-variables)
- [4. Lab Exercise: Deploy A SpringBoot application with an Elastic Search Backend](#4-lab-exercise-deploy-a-springboot-application-with-an-elastic-search-backend)

<!-- /TOC -->
### 1. Install and Setup CLIs
#### Install PKS CLI
In order to install the PKS CLI please follow these instructions: https://docs.pivotal.io/runtimes/pks/1-2/installing-pks-cli.html#windows. Note, you will need to register with network.pivotal.io in order to download the CLI.

Download from: https://network.pivotal.io/products/pivotal-container-service/

#### Install kubectl CLI
You can install the kubectl CLI from PivNet as well, https://network.pivotal.io/products/pivotal-container-service

What you download is the executable. After downloading, rename the file to `kubectl`, move it to where you like and make sure it's in your path.

For reference, here are some other ways to install, https://kubernetes.io/docs/tasks/tools/install-kubectl

### 2. Cluster Access and Validation
#### Get Cluster Credentials
You will need to retrieve the cluster credentials from PKS. First login using the the PKS credentials that were provided to you for this lab exercise.

<pre>
pks login -a api.pks.cfrocket.com -u USERNAME -p PASSWORD -k
</pre>

Now you can retrive your Kubernetes cluster credentials. Please use the cluster name that was provided to you for this lab exercise.

<pre>
pks get-credentials CLUSTER-NAME
</pre>

#### Validating your Cluster
Ensure that you can access the API Endpoints on the Master
<pre>kubectl cluster-info</pre>

You should see something similar to the following:
<pre>
Kubernetes master is running at https://workshop.clusters.pks.cfrocket.com:8443
Heapster is running at https://workshop.clusters.pks.cfrocket.com:8443/api/v1/namespaces/kube-system/services/heapster/proxy
KubeDNS is running at https://workshop.clusters.pks.cfrocket.com:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://workshop.clusters.pks.cfrocket.com:8443/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
monitoring-influxdb is running at https://workshop.clusters.pks.cfrocket.com:8443/api/v1/namespaces/kube-system/services/monitoring-influxdb/proxy
</pre>

#### Accessing the Dashboard

To access Dashboard from your local workstation you must create a secure channel to your Kubernetes cluster. Run the following command:

<pre>kubectl proxy</pre>

Now access Dashboard at:

http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/.

When prompted for choosing either the Kubeconfig or Token, choose Kubeconfig.  You will need to browse to HOME-DIR/.kube and select the file named `config`.

On Windows, you may want to use Firefox or Chrome as Explorer has some issues.

### 3. Lab Exercise: Set Environment Variables

Prerequisite: Initialize the environment with your credentials for the registry. Please use the account and user index that was provided to you for this lab exercise.

Unix/Mac
<pre>
export USER_INDEX="1"
export HARBOR_REGISTRY_URL="harbor.pks.cfrocket.com"
export HARBOR_USERNAME="User1"
export HARBOR_PASSWORD="Pivotal1"
export HARBOR_EMAIL="User1@acme.org"
</pre>

Windows PowerShell
<pre>
$env:USER_INDEX="1"
$env:HARBOR_REGISTRY_URL="harbor.pks.cfrocket.com"
$env:HARBOR_USERNAME="User1"
$env:HARBOR_PASSWORD="Pivotal1"
$env:HARBOR_EMAIL="User1@acme.org"
</pre>

### 4. Lab Exercise: Deploy A SpringBoot application with an Elastic Search Backend
1. **(Skip this step)** Provision a StorageClass for the Cluster. This is provisioned at the Kubernetes cluster level and therefore no need to namespace qualify it.

<ul>GCP:

  <pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_0_ProvisionStorageClass_GCP.yaml</pre>
</ul>


<ul>vSphere:

  <pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_0_ProvisionStorageClass_vSphere.yaml</pre>
</ul>

2. Create a user defined Namespace. Note: Update the command below to use the namespace that you are going to be delpoying into.
<ul>Unix/Mac
<pre>kubectl create namespace geosearch-$(echo $USER_INDEX)
kubectl config set-context $(kubectl config current-context) --namespace=geosearch-$(echo $USER_INDEX)
</pre></ul>

<ul>Windows PowerShell
<pre>kubectl create namespace geosearch-$(echo $env:USER_INDEX)
kubectl config set-context $(kubectl config current-context) --namespace=geosearch-$(echo $env:USER_INDEX)
</pre></ul>


3. Create Harbor Registry Secret. Use the Registry credentials that was provided to you for this step.
<ul>Unix/Mac
<pre>kubectl create secret docker-registry harborsecret --docker-server="$(echo $HARBOR_REGISTRY_URL)" --docker-username="$(echo $HARBOR_USERNAME)" --docker-password="$(echo $HARBOR_PASSWORD)" --docker-email="$(echo $HARBOR_EMAIL)"</pre>
</ul>

<ul>Windows PowerShell
<pre>kubectl create secret docker-registry harborsecret --docker-server="$(echo $env:HARBOR_REGISTRY_URL)" --docker-username="$(echo $env:HARBOR_USERNAME)" --docker-password="$(echo $env:HARBOR_PASSWORD)" --docker-email="$(echo $env:HARBOR_EMAIL)"</pre>
</ul>

4. Create a new Service Account and Image pull secret
<ul>Unix/Mac
<pre>
kubectl create serviceaccount userserviceaccount
kubectl patch serviceaccount userserviceaccount -p "{\"imagePullSecrets\": [{\"name\": \"harborsecret\"}]}"
</pre></ul>

<ul>Windows PowerShell
<pre>
kubectl create serviceaccount userserviceaccount
kubectl patch serviceaccount userserviceaccount -p '{\"imagePullSecrets\": [{\"name\": \"harborsecret\"}]}'
</pre></ul>

5. Create the Storage Volume
<ul><pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_1_ProvisionStorage.yaml</pre></ul>

6. Deploy Elastic Search
<ul><pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_2_DeployElasticSearch.yaml</pre></ul>

7. Expose the Elastic Search Service
<ul><pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_3_ExposeElasticSearch.yaml</pre></ul>

8. Load the Data via a Job
<ul><pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_4_LoadData.yaml</pre></ul>

9. Deploy the SpringBoot Geosearch Application
<ul><pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_5_DeploySpringBootApp.yaml</pre></ul>

10. Expose the SpringBoot Application. This can be done in a couple of ways. We will look at two ways of doing it in this example.

<ul>Exposing with the LoadBalancer
	<pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_6_ExposeSpringBootApp.yaml</pre>
</ul>

<ul>Exposing with the Ingress 
	<pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_6_ExposeSpringBootAppIngress.yaml</pre>
</ul>

11. Scale the Frontend
<ul><pre>kubectl scale deployment --replicas=3 geosearch</pre></ul>
