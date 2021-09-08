# Basic commands referencing Helm
### Useful Helm Commands
#### - To lists all of the releases under all namespace
```
helm list -A
```
#### - To list helm chart repositories, search a particular chart, and show the chart values, here chart is external-secrets
```
helm repo list
helm search repo external-secrets
helm show values  external-secrets/kubernetes-external-secrets
```
#### - To export the values in the chart to values.yaml, here external-secrets and bitn
```
helm show values external-secrets\mysecrets --version=8.3.0 > values.yaml
helm show values bitnami/mysql --version=8.7.1 > values.yaml
```
#### - To upgrade the release of chart to new version after changing the parameters in value.yaml
```
helm upgrade bitnami-mysql  bitnami/mysql -f value.yaml
```
