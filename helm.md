# Basic commands referencing Helm
### Useful Helm Commands

#### - To list helm chart repositories
```
helm repo list
o/p:
jagathuser@RaviTestVM:~$ helm repo list
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/jagathuser/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/jagathuser/.kube/config
NAME                    URL
bitnami                 https://charts.bitnami.com/bitnami
stable                  https://charts.helm.sh/stable
external-secrets        https://external-secrets.github.io/kubernetes-external-secrets/
vmware-tanzu            https://vmware-tanzu.github.io/helm-charts
```
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
#### - To seach repo: bitnami/mysql
```
helm search repo bitnami/mysql
o/p:
jagathuser@RaviTestVM:~$ helm search repo mysql
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/jagathuser/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/jagathuser/.kube/config
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
bitnami/mysql                           8.8.7           8.0.26          Chart to create a Highly available MySQL cluster
stable/mysql                            1.6.9           5.7.30          DEPRECATED - Fast, reliable, scalable, and easy...
stable/mysqldump                        2.6.2           2.4.1           DEPRECATED! - A Helm chart to help backup MySQL...
stable/prometheus-mysql-exporter        0.7.1           v0.11.0         DEPRECATED A Helm chart for prometheus mysql ex...
bitnami/phpmyadmin                      8.2.12          5.1.1           phpMyAdmin is an mysql administration frontend
stable/percona                          1.2.3           5.7.26          DEPRECATED - free, fully compatible, enhanced, ...
stable/percona-xtradb-cluster           1.0.8           5.7.19          DEPRECATED - free, fully compatible, enhanced, ...
stable/phpmyadmin                       4.3.5           5.0.1           DEPRECATED phpMyAdmin is an mysql administratio...
bitnami/mariadb                         9.5.1           10.5.12         Fast, reliable, scalable, and easy to use open-...
bitnami/mariadb-cluster                 1.0.2           10.2.14         DEPRECATED Chart to create a Highly available M...
bitnami/mariadb-galera                  5.13.4          10.5.12         MariaDB Galera is a multi-master database clust...
stable/gcloud-sqlproxy                  0.6.1           1.11            DEPRECATED Google Cloud SQL Proxy
stable/mariadb                          7.3.14          10.3.22         DEPRECATED Fast, reliable, scalable, and easy t...
```
```
helm search repo external-secrets
jagathuser@RaviTestVM:~/velero$ helm search repo external-secrets
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/jagathuser/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/jagathuser/.kube/config
NAME                                            CHART VERSION   APP VERSION     DESCRIPTION
external-secrets/kubernetes-external-secrets    8.3.0           8.3.0           Kubernetes External Secrets CustomResourceDefin...
```
#### - To show the values of the repo: bitnami/mysql 
```
helm show values bitnami/mysql
helm show values external-secrets/kubernetes-external-secrets
```
#### - To export the values in the repo to values.yaml, here external-secrets and bitn
```
helm show values bitnami/mysql --version=8.8.7 > bitmani-mysql-8.8.7-values.yaml
helm show values external-secrets\mysecrets --version=8.3.0 > values.yaml
```
#### - To search repo: bitnami/mysql with all versions
```
jagathuser@RaviTestVM:~$ helm search repo bitnami/mysql --versions
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/jagathuser/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/jagathuser/.kube/config
NAME            CHART VERSION   APP VERSION     DESCRIPTION
bitnami/mysql   8.8.7           8.0.26          Chart to create a Highly available MySQL cluster
bitnami/mysql   8.8.6           8.0.26          Chart to create a Highly available MySQL cluster
bitnami/mysql   8.8.5           8.0.26          Chart to create a Highly available MySQL cluster
bitnami/mysql   8.8.4           8.0.26          Chart to create a Highly available MySQL cluster
bitnami/mysql   8.8.3           8.0.26          Chart to create a Highly available MySQL cluster
bitnami/mysql   8.8.2           8.0.26          Chart to create a Highly available MySQL cluster
bitnami/mysql   8.8.1           8.0.26          Chart to create a Highly available MySQL cluster
bitnami/mysql   8.8.0           8.0.26          Chart to create a Highly available MySQL cluste
```
#### - To install mysql

#### - To upgrade the release of chart to new version after changing the parameters in value.yaml
```
helm install jagath-mysql bitnami/mysql -f bitmani-mysql-8.8.7-values.yaml
```
### - To upgrade with with any values set in the value.yaml
```
helm upgrade bitnami-mysql  bitnami/mysql -f value.yaml
```
### - To connect to mysql pod
```
k exec -it mysql-0 -- bash
```
### - To connect to mysql shell
```
mysql -uroot -p
```
