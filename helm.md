# Basic commands referencing Helm
### Useful Helm Commands
#### - To lists all of the releases under all namespace
```
helm list -A
o/p:
jagathuser@RaviTestVM:~$ helm list -A
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/jagathuser/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/jagathuser/.kube/config
NAME                            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                                   APP VERSION
fluent-bit                      logging         1               2021-08-05 02:27:27.0418335 +0800 +0800 deployed        fluent-bit-0.16.2                       1.8.3
k10                             kasten-io       4               2021-07-22 03:01:55.554142875 +0000 UTC deployed        k10-4.0.6                               4.0.6
mysql-external-secrets          secrets         1               2021-09-07 18:31:06.025121857 +0000 UTC deployed        kubernetes-external-secrets-8.3.0       8.3.0
nginx-ingress                   ingress-basic   1               2021-07-17 11:58:38.710982744 +0000 UTC deployed        ingress-nginx-3.34.0                    0.47.0
prometheus-jagathaskdemo        prometheus      1               2021-08-04 01:41:42.547055922 +0000 UTC deployed        kube-prometheus-stack-17.1.0            0.49.0
velero                          velero          1               2021-09-16 15:01:45.762344064 +0000 UTC deployed        velero-2.23.8                           1.6.3

```
#### - To list helm chart repositories from the above releases, search a particular chart, and show the chart values, here chart is external-secrets
```
**helm repo list**
o/p:
jagathuser@RaviTestVM:~$ helm repo list
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/jagathuser/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/jagathuser/.kube/config
NAME                    URL
bitnami                 https://charts.bitnami.com/bitnami
stable                  https://charts.helm.sh/stable
external-secrets        https://external-secrets.github.io/kubernetes-external-secrets/
vmware-tanzu            https://vmware-tanzu.github.io/helm-charts

helm search repo bitnami/mysql

helm show values external-secrets/kubernetes-external-secrets
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
