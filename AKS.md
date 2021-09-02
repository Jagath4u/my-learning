# Basic commands referencing Kubernetes
#### Useful Kubectl Commands
- <font size="1"> Login to Azure </font>
```
az login
```
- Connect to an Azure Subscription
```
az account set --subscription af0e1a40-9ad6-46d1-aa24-9ae4a7a1f786
```
- Start AKS cluster = jagathaksdemo
```
az aks start --name jagathaksdemo --resource-group jagathaksdemo
```
- To set alias for repeadly using kubectl commands, go to \home\<user>\.bashrc and provide the below entries. After providing these entries give **source .bashrc**
```
alias k=kubectl
alias kcn='kubectl config set-context --current --namespace'
alias kcg='kubectl config get-contexts'
alias kcu='kubectl config use-context'
```
