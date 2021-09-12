# Steps to deploy Nginx with secret in AKS
### - Create an Azure Container registry in Azure portal with name: containerregistryjagathaskdemo
#### - Install Docker Engine on Ubuntu using Registory
#### - Set up the repository
```
 sudo apt-get update
 sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
#### - Add Docker’s official GPG key
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
#### - Use the following command to set up the stable repository.
```
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
#### - Install Docker Engine
```
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io
```
#### - This completes the docker install

### - To view the docker images
```
docker images
```
### Steps to connect to ACR
- **login server**: containerregistryjagathaskdemo.azurecr.io, 
- **username**: containerregistryjagathaskdemo, 
- **password**: rUG0+/3SwbLS09aZUrHW8tbbda3VveN4,  then check in Azure Portal under ACR
```
az acr login --name containerregistryjagathaskdemo.azurecr.io
```
### - now give docker push
```
docker push containerregistryjagathaskdemo.azurecr.io/nginx
```
### - Create nginx deployment with the below yaml, file name: **nginx_acr.yaml**
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: containerregistryjagathaskdemo.azurecr.io/nginx:latest
        ports:
        - containerPort: 80
      imagePullSecrets:
      - name: acrsecret
```
### - Create secret with name: acrsecret
```
kubectl create secret docker-registry acrsecret \
> --docker-server=containerregistryjagathaskdemo.azurecr.io \
> --docker-username=containerregistryjagathaskdemo \
> --docker-password="rUG0+/3SwbLS09aZUrHW8tbbda3VveN4"
```
### - Finally deploy the nginx using secret.
k apply -f nginx_acr.yaml
```
