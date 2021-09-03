# Basic commands referencing Kubernetes
### Useful Kubectl Commands
#### - Login to Azure
```
az login
```
#### - Connect to an Azure Subscription
```
az account set --subscription af0e1a40-9ad6-46d1-aa24-9ae4a7a1f786
```
#### - Start AKS cluster = jagathaksdemo
```
az aks start --name jagathaksdemo --resource-group jagathaksdemo
```
#### - To set alias for repeadly using kubectl commands, go to \home\<user>\.bashrc and provide the below entries. After providing these entries give **source .bashrc**
```
alias k=kubectl
alias kcn='kubectl config set-context --current --namespace'
alias kcg='kubectl config get-contexts'
alias kcu='kubectl config use-context'
```
#### - List the namespace
```
k get ns
```
#### - Switch to namespace = bigdatadb
```
kcn bigdatadb
```
#### - List the status of pods
```
k get pods
```
#### - Check the pod in detail, where pod = mysql-primary-0
```
k get describle pod mysql-primary-0
```
## - To apply a yaml file
```
k apply -f /home/jagathuser/nginxjagath.yaml
```
#### - To check status of pv and pvc
```
k get pv
k get pvc
**Note: The status of the persistent volume and volume claim should be BOUND**
```
#### - To check the status of services in pods
```
k get services
```
#### - To check the secrets
```
k get secrets
```
#### - To list down the statefulsets
```
k get sts
```
#### - To scale down the statefulset
```
k scale sts mysql-secondary --replicas=0
```
#### - To describe the secret
```
k get secret mysql -oyaml
```
#### - To see the decode the encoded password
```
echo cHJvbS1vcGVyYXRvcg== | base64 -d
```
#### - To get volumesnapshot class
```
k get volumesnapshotclass
```
