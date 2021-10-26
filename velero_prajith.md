![100]

# Overview

Velero (formerly Heptio Ark) gives you tools to back up and restore your Kubernetes cluster resources and persistent volumes. You can run Velero with a public cloud platform or on-premises. Velero lets you:

* Take backups of your cluster and restore in case of loss.
* Migrate cluster resources to other clusters.
* Replicate your production cluster to development and testing clusters.

Velero consists of:

* A server that runs on your cluster
* A command-line client that runs locally. (velero-cli binary can be downloaded from [here][3])


# Table of Contents 
   - [SKF AKS BACKUP SETUP WITH VELERO](#SKF-AKS-BACKUP-SETUP-WITH-VELERO)
      - [Architecture Diagram](#Architecture-Diagram)  
      - [Libraries for velero](#Libraries-for-velero)  
      - [Installation](#Installation)
      - [Velero CLI useful commands](#Velero-CLI-useful-commands)
      - [EXAMPLE: Backup and Restore StatefulSet mysql DB](#EXAMPLE:-Backup-and-Restore-StatefulSet-mysql-DB ) 
        - [Overview](#Overview)  
        - [Prerequisites](#Prerequisites)  
        - [Verify databases in running mysql instance](#Updating-databases-in-running-mysql-instance)
        - [Creating velero backup schedule](#Creating-velero-backup-schedule)
        - [Delete mysql instance by deleting entire namespace](#Delete-mysql-instance-by-deleting-entire-namespace)
        - [Restore mysql instance](#Restore-mysql-instance)
        - [Conclusions](#Conclusions)

##### 

# SKF AKS BACKUP SETUP WITH VELERO

## Architecture Diagram

![velero.drawio.png](/.attachments/velero.drawio-1df60377-3cda-4ee2-acac-42cacdafd084.png)


## Libraries for velero
Installation of velero in all the AKS across all factories (both Public Cloud AKS and HCI-AKS) are managed via gitops approach. All the deployment codes are in  [k8s-cluster-configuration][1] repo and is in jsonnet/libsonnet format.
Under above mentioned repository, the library code for velero deployment is placed under libs directory [here][2], which consist of all the velero libsonnet files. 

We are using helm charts to deploy velero in AKS cluster. Below are the details of each files under libs/velero directory:
- **chart.libsonnet**: Main file which is calling dependencies chart from offical velero helm [repo][4] and its version.
- **customizations.libsonnet**: All the custom values which you want to modify from the default values file is placed here.
- **extras.libsonnet**: All the additional resources that you want to create along with this chart are placed in the file (eg: External-secret resource)
- **values.libsonnet**: All the default values that comes along with specific version of velero chart is placed here.

## Installation
When ever you need to deploy velero in a newly build AKS cluster (either in public /on-prem), you deploy it by creating velero app in respective ***argoCD*** for that particular site/enviroment.

Gitops codes to deploy velero to each factory AKS is also under repo [k8s-cluster-configuration][1].

You need to create sub-folder named "velero" under your target AKS cluster folder. i.e, ***"clusters/<region>/<factory>/<env>/<aks-name>/"*** in the repo. Idea here is, You will be adding all your site specific velero customization under this velero folder which is also pulling velero library specified in  [Libraries for velero](#Libraries-for-velero) topic.
Please checkout [here][5], for reference code which need to be put under velero folder for each AKS cluster. 

Also make sure to udpate below code  in ***apps.jsonnet*** for creating new velero app in argocd.  Check [here][6] for reference.

```
velero: clustermgmt.JsonnetHelmApplicationWithCrds(p, 'velero', 'velero'),
```

You can create a new branch and push all the codes for velero deployment for that particular target AKS. Once done, proceed to make a Pull Request which will trigger automatic pipeline to validate the changes you made. Once it is approved and merge to master, argocd will detect the changes and velero will be deployed to the target cluster automtically. Moreover, you can proceed to argocd UI to check velero appilcation deployment status.

You can create all your custom backup schedule inside file ***overrides.libsonnet*** which is under velero folder for that specific target AKS cluster. 

## Velero CLI useful commands
```
velero backup-location get
velero backup get
velero schedule get
```
```
velero backup create firstbackup      //backup entire cluster
velero backup create firstbackup --include-namespaces testing   //backup only testing ns
velero backup create firstbackup --include-namespaces testing --inlcuding-resources pods,deployments
velero backup create firstbackup --include-namespaces testing --exclude-resources pods
```
```
velero schedule create firstschedule --schedule="*/15 * * * *"  // very 15 min
velero schedule create firstschedule --schedule="0 * * * *" // very hour
velero schedule create firstschedule --schedule="@every 2d"
velero schedule create firstschedule --schedule="@every 1w"
```
```
# Create a restore from the latest successful backup triggered by schedule "schedule-1"
velero restore create --from-schedule schedule-1

# Create a restore from the latest successful OR partially-failed backup triggered by schedule "schedule-1"
velero restore create --from-schedule schedule-1 --allow-partially-failed

# create a restore for only persistentvolumeclaims and persistentvolumes within a backup
velero restore create --from-backup backup-1 --include-resources persistentvolumeclaims,persistentvolumes

# create a restore for only the elasticsearch cluster within a backup
velero restore create --from-backup backup-1 --selector app=elasticsearch-master

# Create a restore including the nginx and default namespaces
velero restore create --from-backup backup-1 --include-namespaces nginx,default

# Create a restore excluding the kube-system and default namespaces
velero restore create --from-backup backup-1 --exclude-namespaces kube-system,default
```
# EXAMPLE: Backup and Restore StatefulSet mysql DB

## Overview
This example demonstrates Velero backup and restore for a StatefulSet application with namespace. The mysql DB is used for demonstrating backup and restore with Velero.

When restoring a stateful application using Velero, the storage class that was used by the PersistentVolumeClaim (PVC) in the application must be present on the Kubernetes cluster. If the PVC was using the default storage class, then the default storage class must also be present prior to initiating the restore operation with Velero.

## Prerequisites

- Install and configure Velero, and Restic.
- Running mysqldb statefulset 

## Verify databases in running mysql instance

- Here, we assume mysqldb statefulset (named mysql-test) deployed is running under namespace mysql-test and all the resources are running fine as show below

```
$ k get all -n mysql-test
NAME               READY   STATUS    RESTARTS   AGE
pod/mysql-test-0   1/1     Running   0          31m

NAME                          TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)          AGE
service/mysql-test            LoadBalancer   10.100.7.20   10.168.32.248   3306:30158/TCP   31m
service/mysql-test-headless   ClusterIP      None          <none>          3306/TCP         31m

NAME                          READY   AGE
statefulset.apps/mysql-test   1/1     31m
```

- Let's inspect the mysqldb

```
$ k exec -it mysql-test-0 -n mysql-test -- bash
I have no name!@mysql-test-0:/$ mysql -u dxcadmin -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 458
Server version: 8.0.23 Source distribution

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| my_database        |
| mysql              |
| performance_schema |
| sys                |
| test1              |
| test2              |
| test3              |
| test4              |
| test5              |
+--------------------+
10 rows in set (0.09 sec)

```
> ***Here, with this we resembles a DB which is having databases/tables created and is functional.***

## Creating velero backup schedule
-  Here we are creating backup schedule named mysql-test-ns with velero client binary to backup entire namespace where mysql-test db is running

- As mentioned in the command below, this will create a scheduled backup which will create backups every 6hours and default TTL would be 720hr unless we specify to change it explicitly. 

```
$ velero schedule create mysql-test-ns --schedule="@every 6h" --include-namespaces mysql-test
Schedule "mysql-test-ns" created successfully.
```

- You can see the schdule you created as below
```
$ velero schedule get
NAME                STATUS    CREATED                          SCHEDULE    BACKUP TTL   LAST BACKUP   SELECTOR
mysql-test-ns       Enabled   2021-04-21 00:33:45 +0200 CEST   @every 6h   720h0m0s     53s ago       <none>
```

- The schedule in turn creates backup object as show below and is in NEW status.
```
$ velero backup get
NAME                               STATUS       ERRORS   WARNINGS   CREATED                          EXPIRES   STORAGE LOCATION   SELECTOR
mysql-test-ns-20210420223345       New          0        0          <nil>                            29d                          <none>
```
- Once the backup is completed, you will see the status changes to 'completed'. With this our first backup is done as per schdeule.

```
$ velero backup get
NAME                               STATUS      ERRORS   WARNINGS   CREATED                          EXPIRES   STORAGE LOCATION   SELECTOR
mysql-test-ns-20210420223345       Completed   0        0          2021-04-21 00:37:11 +0200 CEST   29d       default            <none>
```

- You can query the details of backup created as shown below.

```
$ velero backup describe mysql-test-ns-20210420223345 --details
Name:         mysql-test-ns-20210420223345
Namespace:    velero
Labels:       velero.io/schedule-name=mysql-test-ns
              velero.io/storage-location=default
Annotations:  velero.io/source-cluster-k8s-gitversion=v1.18.10
              velero.io/source-cluster-k8s-major-version=1
              velero.io/source-cluster-k8s-minor-version=18

Phase:  Completed

Errors:    0
Warnings:  0

Namespaces:
  Included:  mysql-test
  Excluded:  <none>

Resources:
  Included:        *
  Excluded:        <none>
  Cluster-scoped:  auto

Label selector:  <none>

Storage Location:  default

Velero-Native Snapshot PVs:  auto

TTL:  720h0m0s

Hooks:  <none>

Backup Format Version:  1.1.0

Started:    2021-04-21 00:37:11 +0200 CEST
Completed:  2021-04-21 00:37:25 +0200 CEST

Expiration:  2021-05-21 00:37:11 +0200 CEST

Total items to be backed up:  34
Items backed up:              34

Resource List:
  apiextensions.k8s.io/v1/CustomResourceDefinition:
    - externalsecrets.kubernetes-client.io
  apps/v1/ControllerRevision:
    - mysql-test/mysql-test-5c887fffd4
  apps/v1/StatefulSet:
    - mysql-test/mysql-test
  discovery.k8s.io/v1beta1/EndpointSlice:
    - mysql-test/mysql-test-gf4gz
    - mysql-test/mysql-test-headless-xrlvz
  kubernetes-client.io/v1/ExternalSecret:
    - mysql-test/mysql-secret
  v1/ConfigMap:
    - mysql-test/mysql-test
  v1/Endpoints:
    - mysql-test/mysql-test
    - mysql-test/mysql-test-headless
  v1/Event:
    - mysql-test/data-mysql-test-0.1677b00499c3e06e
    - mysql-test/data-mysql-test-0.1677b0049a43a818
    - mysql-test/data-mysql-test-0.1677b004e1473ab0
    - mysql-test/mysql-test-0.1677b0049a7f23cd
    - mysql-test/mysql-test-0.1677b00524e2dc45
    - mysql-test/mysql-test-0.1677b005caabdb24
    - mysql-test/mysql-test-0.1677b00768054f56
    - mysql-test/mysql-test-0.1677b0076ffba3da
    - mysql-test/mysql-test-0.1677b00776bd5952
    - mysql-test/mysql-test-0.1677b00ed4b501fd
    - mysql-test/mysql-test.1677b00482069c4c
    - mysql-test/mysql-test.1677b00485c839e9
    - mysql-test/mysql-test.1677b004996bb0b6
    - mysql-test/mysql-test.1677b00499cbe01e
  v1/Namespace:
    - mysql-test
  v1/PersistentVolume:
    - pvc-3d57e2fa-d643-4762-9ae0-92cecccaa4b6
  v1/PersistentVolumeClaim:
    - mysql-test/data-mysql-test-0
  v1/Pod:
    - mysql-test/mysql-test-0
  v1/Secret:
    - mysql-test/default-token-cpt5k
    - mysql-test/mysql-secret
    - mysql-test/mysql-test-token-sv8d6
  v1/Service:
    - mysql-test/mysql-test
    - mysql-test/mysql-test-headless
  v1/ServiceAccount:
    - mysql-test/default
    - mysql-test/mysql-test

Velero-Native Snapshots: <none included>

Restic Backups:
  Completed:
    mysql-test/mysql-test-0: data
```
> As you can see above, We integrated restic with Velero for taking backup of PVs. With restic, users will have an out-of-the-box solution for backing up and restoring almost any type of Kubernetes volume. This is a new capability for Velero, not a replacement for existing functionality


## Delete mysql instance by deleting entire namespace
```
$ k delete ns mysql-test
namespace "mysql-test" deleted
```
> As you can see below, now there is no resources in mysql-test namespace.
```
$ k get all -n mysql-test
No resources found in mysql-test namespace.
```

## Restore mysql instance

- Let's first create a restore with velero client binary specifying the backup that was taken before.

```
velero restore create --from-backup mysql-test-ns-20210420223345
Restore request "mysql-test-ns-20210420223345-20210421004921" submitted successfully.
Run `velero restore describe mysql-test-ns-20210420223345-20210421004921` or `velero restore logs mysql-test-ns-20210420223345-20210421004921` for more details.
```

- You can see the status of restore as show below.

```
velero restore  get
NAME                                           BACKUP                          STATUS       STARTED                          COMPLETED                        ERRORS   WARNINGS   CREATED                          SELECTOR
mysql-test-ns-20210420223345-20210421004921    mysql-test-ns-20210420223345    InProgress   2021-04-21 00:49:21 +0200 CEST   <nil>                            0        0          2021-04-21 00:49:21 +0200 CEST   <none>
```
  
- Once restoration is completed, you can see status will change to ***completed***
```
velero restore  get
NAME                                           BACKUP                          STATUS      STARTED                          COMPLETED                        ERRORS   WARNINGS   CREATED                          SELECTOR
mysql-test-ns-20210420223345-20210421004921    mysql-test-ns-20210420223345    Completed   2021-04-21 00:49:21 +0200 CEST   2021-04-21 00:49:47 +0200 CEST   0        0          2021-04-21 00:49:21 +0200 CEST   <none>
```

- Lets check back the namespace mysql-test

```
k get all -n mysql-test
NAME               READY   STATUS    RESTARTS   AGE
pod/mysql-test-0   1/1     Running   0          101s

NAME                          TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
service/mysql-test            LoadBalancer   10.111.174.214   10.168.32.248   3306:32256/TCP   101s
service/mysql-test-headless   ClusterIP      None             <none>          3306/TCP         101s

NAME                          READY   AGE
statefulset.apps/mysql-test   1/1     101s
```
> As you can see above, all the resources has been recovered and restored in mysql-test namespace.

- Now, lets verify the DB contents 

```
$ k exec -it mysql-test-0 -n mysql-test -- bash
I have no name!@mysql-test-0:/$ mysql -u dxcadmin -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 15
Server version: 8.0.23 Source distribution

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| my_database        |
| mysql              |
| performance_schema |
| sys                |
| test1              |
| test2              |
| test3              |
| test4              |
| test5              |
+--------------------+
10 rows in set (0.01 sec)

mysql> exit
Bye
I have no name!@mysql-test-0:/$
I have no name!@mysql-test-0:/$
I have no name!@mysql-test-0:/$ exit
exit
```
> As you can see above, all the datas has been restored as well.

## Conclusions
Key takeaways from the restore operation:

- Pod annotation is still required for Velero backup of PV
- Restic works fine with CSI

[1]: https://dev.azure.com/skf-digital-manufacturing/SKF-DP-WCM%20Infrastructure/_git/k8s-cluster-configuration
[2]: https://dev.azure.com/skf-digital-manufacturing/SKF-DP-WCM%20Infrastructure/_git/k8s-cluster-configuration?path=/libs/velero
[3]: https://github.com/vmware-tanzu/velero/releases
[4]:  https://vmware-tanzu.github.io/helm-charts
[5]: https://dev.azure.com/skf-digital-manufacturing/SKF-DP-WCM%20Infrastructure/_git/k8s-cluster-configuration?path=/clusters/weu/de18l/prd/de-18l-prd-01-aks-hci/velero
[6]: https://dev.azure.com/skf-digital-manufacturing/SKF-DP-WCM%20Infrastructure/_git/k8s-cluster-configuration?path=/clusters/weu/de18l/prd/de-18l-prd-01-aks-hci/apps.jsonnet
[100]: https://velero.io/docs/main/img/velero.png
