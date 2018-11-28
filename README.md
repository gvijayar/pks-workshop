## Pivotal Container Service Workshop
This is a sample SpringBoot application that performs Geo Bounded queries against an Elastic Search instance and plots the data on a map interactively. This application can be run on a workstation or in a cloud environment such as Cloud Foundry. In this example, I will show how to deploy the application on a running Cloud Foundry instance. 

### Accessing the Dashboard

To access Dashboard from your local workstation you must create a secure channel to your Kubernetes cluster. Run the following command:

<pre>kubectl proxy</pre>

Now access Dashboard at:

http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/.

### Please follow these steps to deploy this application.

Prerequisite: Initialize the environment with your credentials. Please use the account that was provided to you for this lab exercise.

<pre>
export USER_INDEX="1"
export HARBOR_REGISTRY_URL="harbor.pks.cfrocket.com"
export HARBOR_USERNAME="User1"
export HARBOR_PASSWORD="Pivotal1"
export HARBOR_EMAIL="User1@acme.org"
</pre>

1. Provision a StorageClass for the Cluster. This is provisioned at the Kubernetes cluster level and therefore no need to namespace qualify it.

<ul>GCP:
  
  <pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_0_ProvisionStorageClass_GCP.yaml</pre>
</ul>

  
<ul>vSphere:
  
  <pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_0_ProvisionStorageClass_vSphere.yaml</pre>
</ul>

2. Create a user defined Namespace. Note: Update the command below to use the namespace that you are going to be delpoying into.
<ul><pre>kubectl create namespace geosearch-$(echo $USER_INDEX)
kubectl config set-context $(kubectl config current-context) --namespace=geosearch-$(echo $USER_INDEX)
</pre></ul>


3. Create Harbor Registry Secret. Use the Registry credentials that was provided to you for this step.
<ul><pre>kubectl create secret docker-registry harborsecret --docker-server="$(echo $HARBOR_REGISTRY_URL)" --docker-username="$(echo $HARBOR_USERNAME)" --docker-password="$(echo $HARBOR_PASSWORD)" --docker-email="$(echo $HARBOR_EMAIL)"</pre></ul>

4. Create a new Service Account and Image pull secret
<ul><pre>
kubectl create serviceaccount userserviceaccount
kubectl patch serviceaccount userserviceaccount -p "{\"imagePullSecrets\": [{\"name\": \"harborsecret\"}]}"
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

10. Expose the SpringBoot Application
<ul><pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_6_ExposeSpringBootApp.yaml</pre></ul>

11. Scale the Frontend
<ul><pre>kubectl scale deployment --replicas=3 geosearch</pre></ul>
