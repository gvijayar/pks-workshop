# pks-workshop

Step 1: Create Harbor Registry Secret

kubectl create secret docker-registry regsecret --docker-server="https://harbor.pks.cfrocket.com" --docker-username="USER_NAME" --docker-password="PASSWORD" --docker-email="user@acme.org" -n NAMESPACE


