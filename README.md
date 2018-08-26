## Pivotal Container Service Workshop
This is a sample SpringBoot application that performs Geo Bounded queries against an Elastic Search instance and plots the data on a map interactively. This application can be run on a workstation or in a cloud environment such as Cloud Foundry. In this example, I will show how to deploy the application on a running Cloud Foundry instance. 

Please follow these steps to deploy this application.

1. Create a user defined Namespace
<ul><pre>kubectl create namespace {USER_NAMESPACE}</pre></ul>

2. Create Harbor Registry Secret
<ul><pre>kubectl create secret docker-registry regsecret --docker-server="https://{HARBOR_REGISTRY_URL}" --docker-username="USER_NAME" --docker-password="PASSWORD" --docker-email="user@acme.org" -n {USER_NAMESPACE}</pre></ul>

3. Create a new Service Account and Image pull secret
<ul><pre>
kubectl create serviceaccount geo-search-sa -n {USER_NAMESPACE}
kubectl edit serviceaccount geo-searcg-sa -n {USER_NAMESPACE}
</pre></ul>
and add the following lines
<ul><pre>
imagePullSecrets:
- name: regSecret
</pre></ul>

4. Create the Storage Volume
<ul><pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_1_ProvisionStorage.yaml -n {USER_NAMESPACE}</pre></ul>

5. Deploy Elastic Search
<ul><pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_2_DeployElasticSearch.yaml -n {USER_NAMESPACE}</pre></ul>

6. Expose the Elastic Search Service
<ul><pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_3_ExposeElasticSearch.yaml -n {USER_NAMESPACE}</pre></ul>

7. Load the Data via a Job
<ul><pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_4_LoadData.yaml -n {USER_NAMESPACE}</pre></ul>

8. Deploy the SpringBoot Geosearch Application
<ul><pre>kubectl create -f https://raw.githubusercontent.com/gvijayar/pks-workshop/master/Step_5_DeploySpringBootApp.yaml -n {USER_NAMESPACE}</pre></ul>
