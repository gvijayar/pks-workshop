## Pivotal Container Service Workshop
This is a sample SpringBoot application that performs Geo Bounded queries against an Elastic Search instance and plots the data on a map interactively. This application can be run on a workstation or in a cloud environment such as Cloud Foundry. In this example, I will show how to deploy the application on a running Cloud Foundry instance. 

Please follow these steps to deploy this application.

1. Create a user defined Namespace
<ul><pre>kubectl create namespace {USER_NAMESPACE}</pre></ul>

2. Create Harbor Registry Secret
<ul><pre>kubectl create secret docker-registry regsecret --docker-server="https://{HARBOR_REGISTRY_URL}" --docker-username="USER_NAME" --docker-password="PASSWORD" --docker-email="user@acme.org" -n {USER_NAMESPACE}</pre></ul>

3. Initialize the Elastic Search service with Geo Location data. Update the scripts with the Elastic Search Service endpoints.
<ul><pre>src/main/resources/data/create_schema.sh</pre></ul>
<ul><pre>src/main/resources/data/insert_big_cities.sh</pre></ul>


