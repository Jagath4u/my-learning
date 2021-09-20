# Command on Velero
### - Instruction to install Velero to take backup of MySQL DB containers
### - Before installing velero, Prepare MySQL for a Velero stateful backup by adding Annotations
The first step is to add annotations to each of the Pods in the StatefulSet to indicate that the contents of the persistent volumes, mounted on **data**, needs to be backed up as well. Velero uses the restic program at this time for capturing state/data from Kubernetes. Either make an entry on /home/jagathuser/helm-values/bitmani-mysql-8.8.7-values.yaml as shown below or give commands like below.
```
podAnnotations:
    backup.velero.io/backup-volumes: data
k -n mysql-jagath describe pod/mysql-0 | grep Annotations
Annotations:        <none>
$ k -n mysql-jagath annotate pod/mysql-0 backup.velero.io/backup-volumes=data
pod/mysql-0 annotated
$ k -n mysql describe pod/mysql-0 | grep Annotations
Annotations:        backup.velero.io/backup-volumes: data
```
### - Create a file: credentials-velero in /home/jagathuser/velero
```
AZURE_STORAGE_ACCOUNT_ACCESS_KEY="cxAK+g4ucjtLUfDctJxs8G8iWdQahtdwQwZ7WQ+LfmEvRY4hJ6Fz3shAghFByg52mf37bwWJUtwJRsTUZS+akQ=="
AZURE_CLOUD_NAME=AzurePublicCloud
```
#### - Install Velero
```
helm install  velero vmware-tanzu/velero \
--namespace velero \
--create-namespace \
--set-file credentials.secretContents.cloud=./credentials-velero \
--set configuration.provider=azure \
--set configuration.backupStorageLocation.name=azure \
--set configuration.backupStorageLocation.bucket='jagath' \
--set configuration.backupStorageLocation.config.resourceGroup=jagathaksdemo \
--set configuration.backupStorageLocation.config.storageAccount=bigdatadbstorageaccount \
--set snapshotsEnabled=false \
--set deployRestic=true \
--set image.repository=velero/velero \
--set image.pullPolicy=IfNotPresent \
--set initContainers[0].name=velero-plugin-for-microsoft-azure \
--set initContainers[0].image=velero/velero-plugin-for-microsoft-azure:master \
--set initContainers[0].volumeMounts[0].mountPath=/target \
--set initContainers[0].volumeMounts[0].name=plugins \
--set configuration.backupStorageLocation.config.storageAccountKeyEnvVar='AZURE_STORAGE_ACCOUNT_ACCESS_KEY'
```
#### - To create backup location, need to pass the provider, bucket, region, resourcegroup, storage account.
```
velero backup-location create mysqldb --provider azure --bucket mysql --region resourceGroup=jagathaksdemo,storageAccount=bigdatadbstorageaccount
```
#### - to list the backup location
```
velero get backup-location
```
#### - To create a backup of namespace: mysql-jagath
```
velero backup create mysql-jagath-backup --include-namespaces mysql-jagath
```
#### - To list the backup created, need to switch to velero namespace.
```
velero get backup
```
#### - To retore the namespace: mysql-jagath from the backup taken in storage account in azure
```
velero restore create mysql-jagath-restore --from-backup mysql-jagath-backup
```
#### - To list the restore operations performed
```
velero get restore
```
#### - To delete the velero backup
```
velero delete backup mysql-jagath-backup
```
#### - To delete the velero restore
```
velero delete restore mysql-jagath-latest
```
#### - To uninstall velero
```
kubectl delete namespace/velero clusterrolebinding/velero
kubectl delete crds -l component=velero
kcn velero
helm delete velero
```

